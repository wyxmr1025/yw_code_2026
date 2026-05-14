## prometheus：采集数据

#### 1、监控pod

github上面找kube-metrics

> 如果要监控pod就可以在创建pod的时带上annotations，监控他的时候就查找带有annotations的pod，然后就能监控起来

```
100：monitor中：
vi prom-redis.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: monitor
spec:
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis==redis容器本身，提供服务
          image: redis:4
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
          ports:
            - containerPort: 6379
        - name: redis-exporter===容器redis-exporter，提供监控指标
          image: oliver006/redis_exporter:latest
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
          ports:
            - containerPort: 9121
---
kind: Service
apiVersion: v1
metadata:
  name: redis
  namespace: monitor
  annotations:
    prometheus.io/scrape: 'true'==提供监控（这个想怎么写就咋写，要符合k8s约定，这个是自己规定的）
    prometheus.io/port: '9121'===监控端口9121（同上，到时候这个作为监控的指标，监控起来）
spec:
  selector:
    app: redis
  ports:
    - name: redis
      port: 6379
      targetPort: 6379===服务端口
    - name: prom
      port: 9121
      targetPort: 9121=====监控端口

然后把所有带annotations的监控一起来（pod）：
vi 
- job_name: 'kubernetes-endpoints'
  kubernetes_sd_configs:
  - role: endpoints
  relabel_configs:
  - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
    action: keep===保留anntations是true的
    regex: true
  - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scheme]
    action: replace
    target_label: __scheme__
    regex: (https?)==提供监控的协议
  - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]==监控的路径
    action: replace
    target_label: __metrics_path__
    regex: (.+)===提供监控的路径
  - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
    action: replace===监控的端口
    target_label: __address__
    regex: ([^:]+)(?::\d+)?;(\d+) # RE2 正则规则，+是一次多多次，?是0次或1次，其中?:表示非匹配组(意思就是不获取匹配结果)（就是跳过第二个括号）
    replacement: $1:$2
  - action: labelmap
    regex: __meta_kubernetes_service_label_(.+)
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: kubernetes_namespace
  - source_labels: [__meta_kubernetes_service_name]
    action: replace
    target_label: kubernetes_name
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    target_label: kubernetes_pod_name
 
 
 annotations：就是告诉prometheus是否提供监控，是否提供路径，监控的端口等，通过匹配源标签
```

#### 2、promql： 查询语句

​	prometheus：采集的监控数据以指标（metric）的形式存储在内置的TSDB数据库中，这些数据都是时间序列，具有方向

```
指标格式：
go_gc_heap_allocs_by_size_bytes_total_bucket{le="8.999999999999998"} 65165
指标                                          标签                     值
====》向量
瞬时向量：一个时间序列，每个时间序列包含单个样本
区间向量：一组时间序列，包含一段范围的样本数据

指标类型：
counter（计数器）：累加
gauge（仪表盘）：收集的值就是什么值（瞬时，此时此刻），一般检测内存大小
histogram（直方图）：区间
summary（摘要）

prometheus提供的函数： sum、rate（使用率）、delta（过去之前使用的）、avg_over_time（过去一小时平均值）、predict_linear（预测值）
by（instance） 安装节点来分组

1、node上5mincpu的使用率：(1-sum(rate(node_cpu_seconds_total{mode="idle"}[5m])) by (instance,cpu) / sum(rate(node_cpu_seconds_total[5m])) by (instance,cpu))*100

2、过去五分钟cpu的空闲率：delta(node_cpu_seconds_total{mode="idle"}[5m])/300

3、过去一小时内存使用的平均值：
(1-(avg_over_time(node_memory_MemFree_bytes[1h])+avg_over_time(node_memory_Cached_bytes[1h])+avg_over_time(node_memory_Buffers_bytes[1h])) / avg_over_time(node_memory_MemTotal_bytes[1h]))*100

4、预测2小时候磁盘的使用率（空闲率）：predict_linear(node_filesystem_avail_bytes{device="/dev/mapper/centos-root", fstype="xfs", instance="node1.y2312.com", job="node_exporter", mountpoint="/"}[2h],7200)
```

#### 3、k8s集群引入grafana：

```
101：mkdir /data/k8s/grafana
100: pv-pvc deploymen service
vi  grafana.yaml
 # grafana.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: grafana-local
  labels:
    app: grafana
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 2Gi
  storageClassName: local-storage
  local: 
    path: /data/k8s/grafana
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - www.y2312node1.com
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-pvc
  namespace: monitor
  labels:
    app: grafana
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: monitor
spec:
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      volumes:
        - name: storage
          persistentVolumeClaim:
            claimName: grafana-pvc
      securityContext:   ====代表以root身份来运行，并不会很安全
        runAsUser: 0
      containers:
        - name: grafana
          image: grafana/grafana:8.4.6
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 3000
              name: grafana
          env:
            - name: GF_SECURITY_ADMIN_USER
              value: admin
            - name: GF_SECURITY_ADMIN_PASSWORD
              value: admin321
          readinessProbe:
            failureThreshold: 10
            httpGet:
              path: /api/health
              port: 3000
              scheme: HTTP
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 30
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /api/health
              port: 3000
              scheme: HTTP
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          resources:
            limits:
              cpu: 150m
              memory: 512Mi
            requests:
              cpu: 150m
              memory: 512Mi
          volumeMounts:
            - mountPath: /var/lib/grafana
              name: storage
---
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: monitor
spec:
  type: NodePort
  ports:
    - port: 3000
  selector:
    app: grafana
添加数据源： 
grafana ： 输入本机 http://prometheus:9090---save
新建panel： 内存使用率、、、
k9s=====monitor=====service===grafana的端口：32214
浏览器中输入：192.168.18.101：32214
====》admin，admin321
新建数据源： data sources===url:===http://prometheus:9090=== save
新建dashboard：

100 * (1-((avg_over_time(node_memory_MemFree_bytes[1h]) + avg_over_time(node_memory_Cached_bytes[1h]) + avg_over_time(node_memory_Buffers_bytes[1h])) / avg_over_time(node_memory_MemTotal_bytes[1h])))

预测2小时后磁盘可用情况：
predict_linear(node_filesystem_avail_bytes{device="/dev/mapper/centos-root", fstype="xfs", instance="www.y2312master.com", job="node_exporter", mountpoint="/"}[2h],7200)

预测2h后磁盘使用率：
(1-(predict_linear(node_filesystem_avail_bytes{device=~"/dev/.*"}[2h],7200) / predict_linear(node_filesystem_size_bytes{device=~"/dev/.*"}[2h],7200))) *100
```

