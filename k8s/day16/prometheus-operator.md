## prometheus-operator监控告警

#### 1、prometheus-operator回顾

```

prometheus-operator把所有的部署抽象成k8s的资源，而这些资源是通过crd资源部署起来的

crd ：自定义资源==k8s内置的资源和用户自定义的资源

​	1、自定义prometheus资源===创建prometheus集群的资源statefulset 2

​	2、alertmanager资源====statefulset 3



​	servicemonitor----相当于以前写的configmap里面的kubernetes_sd_configs:
												- role: endpoints|pod|node|ingress

​		spec：

​			namespaceselector：===选择名称空间选择器

​			selector：根据标签匹配选择service来选择pod
			jobLabel:标签名（key）

​			endpoints： 选择端口

​			- bearerTokenFile：指定端口
			  interval: 15s
			  tlsConfig:
			    caFile: /var/run/secrets/kubernets.io/serviceaccount/ca.crt
			    serverName: kubernetes
​         	 path：指定路径、证书、token、scheme、tls等

```

![Snipaste_2026-05-11_17-19-57](pict\Snipaste_2026-05-11_17-19-57.png)

>通过crd创建prometheus和alertmanager|prometheusRule(主要用户生成告警规则的)资源，被operator时刻watch到，根据逻辑创建sts和svc、pv、pvc等，在通过servicemonitor创建一些配置规则（类似与写prom/cm的配置规则），通过servicemonitor.spec下的namespaceSelector|selector|jobLabel|endpoints(通过endpoints下发现pod)

![Snipaste_2026-03-28_09-49-28](pict\Snipaste_2026-03-28_09-49-28.png)

```
监控指标：
ingress    
          ingress-controller 
                            nginx
crd  --- controller
alertmanager
servicemonitor  --->     
                        job_name: 
                          kubernetes_sd_configs
                          - role: endpoints 
                          relabel:

prometheus----> apiserver ---etcd  
                    watch
                operator|controller  --->
                                        statefulset---> apiserver 
        registry.cn-chengdu.aliyuncs.com/mr5/monitor:registry.k8s.io.kube-state-metrics.kube-state-metrics-v2.9.2                                                controller                 
                                                                  pod
                                                                  pvc
replicaset ----> apiserver 
                      watch 
                  operator|controller
                                      ---> 
                                          pod
app.kubernetes.io/managed-by: prometheus-operator
app.kubernetes.io/name: kubelet
k8s-app: kubelet

```

#### 2、自定义发现规则

额外配置：additionalScrapeConfigs（类似于创建secret gengeric命令），可以监控其他带有annotations的pod

>监控一些带有annotation的pod可以通过additionalScrapeConfigs进行添加。

```
一、自定义发现规则（通过servermontior来采集pod） 
servicemonitor监控pod：
1、vi kube-prometheus/manifests/prometheus-additional.yaml
- job_name: 'endpoints'
  kubernetes_sd_configs:
    - role: endpoints
  relabel_configs: # 指标采集之前或采集过程中去重新配置
    - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
      action: keep # 保留具有 prometheus.io/scrape=true 这个注解的Service
      regex: true
    - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
      action: replace
      target_label: __metrics_path__
      regex: (.+)
    - source_labels:
        [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
      action: replace
      target_label: __address__
      regex: ([^:]+)(?::\d+)?;(\d+) # RE2 正则规则，+是一次多多次，?是0次或1次，其中?:表示非匹配组(意思就是不获取匹配结果)
      replacement: $1:$2
    - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scheme]
      action: replace
      target_label: __scheme__
      regex: (https?)
    - action: labelmap
      regex: __meta_kubernetes_service_label_(.+)
      replacement: $1
    - source_labels: [__meta_kubernetes_namespace]
      action: replace
      target_label: kubernetes_namespace
    - source_labels: [__meta_kubernetes_service_name]
      action: replace
      target_label: kubernetes_service
    - source_labels: [__meta_kubernetes_pod_name]
      action: replace
      target_label: kubernetes_pod
    - source_labels: [__meta_kubernetes_node_name]
      action: replace
      target_label: kubernetes_node

2、创建secret
kubectl create secret generic additional-configs --from-file=prometheus-additional.yaml -n monitoring

3、vi kube-prometheus/manifests/prometheus-prometheus.yaml
spec:
 additionalScrapeConfigs:  （类似于写configmap，将其挂载使用）
    name: additional-configs
    key: prometheus-additional.yaml
4、授权并绑定：（这个是授予集群管理员权限，权限过大，不建议）    
kubectl  create clusterrolebinding y2312monitor --clusterrole=cluster-admin --serveraccount=monitoring:prometheus-k8s （ns_name:serviceaccess_name）
或者使用以下授权：
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    app.kubernetes.io/component: prometheus
    app.kubernetes.io/instance: k8s
    app.kubernetes.io/name: prometheus
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 2.35.0
  name: prometheus-k8s
rules:
  - apiGroups:
      - ''
    resources:
      - nodes
      - services
      - endpoints
      - pods
      - nodes/proxy
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ''
    resources:
      - configmaps
      - nodes/metrics
    verbs:
      - get
  - nonResourceURLs:
      - /metrics
    verbs:
      - get
```

![Snipaste_2026-03-28_10-01-24](pict\Snipaste_2026-03-28_10-01-24.png)

>当我们使用servicemonitor监控redis时，将servicemonitor规则写入配置文件中出现ErrorDepth failed to list *v1.Endpoints: endpoints is forbidden，表示当前运行的pod没有权限，需要添加权限，给clusterrole用户绑定权限clusterrolebinding

#### 3、展示数据

```
vi kube-prometheus/manifests/grafana-service.yaml
spec:
  type: NodePort (将svc暴露在集群外部)
图形已经自动生成好，想要监控其他选项，自己手动创建。
```

#### 4、prometheus-operator告警规则

```
二、prometheus 告警规则  指定alertmanager
--- 
promethuesRule ===生成告警规则的
cd kube-prometheus/manifests/(manifests下有rules和servicemonitor两个目录)
mkdir rules
cp *Rule* rules/
cd rules/
kubectl delete -f ./ 
$ cat kubePrometheus-prometheusRule.yaml
vi nodeExporter-prometneusRule.yaml（样例）
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    app.kubernetes.io/component: exporter
    app.kubernetes.io/name: kube-prometheus
    app.kubernetes.io/part-of: kube-prometheus
    prometheus: k8s
    role: alert-rules
  name: kube-prometheus-rules
  namespace: monitoring
spec:
  groups:
  - name: general.rules
    rules:
    - alert: TargetDown
      annotations:
        description: '{{ printf "%.4g" $value }}% of the {{ $labels.job }}/{{ $labels.service
          }} targets in {{ $labels.namespace }} namespace are down.'
        runbook_url: https://runbooks.prometheus-operator.dev/runbooks/general/targetdown
        summary: One or more targets are unreachable.
      expr: 100 * (count(up == 0) BY (job, namespace, service) / count(up) BY (job,
        namespace, service)) > 10
      for: 15s
      labels:
        severity: warning
```

##### 4.1alertmanager告警规则实战：

```
vi kube-prometheus/manifests/rules/memrule.yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    prometheus: k8s
    role: alert-rules
  name: mem-rules
  namespace: monitoring
spec:
  groups:
    - name: "load too high"
      rules:
        - alert: "内存过高"
          annotations:
            summary: "节点 {{ $labels.instance }} 内存过高"
            description: "{{ $labels.instance }} of job {{ $labels.job }} 内存过高超过1分钟."
          expr: 100 * (1 - ((avg_over_time(node_memory_MemFree_bytes[1h]) + avg_over_time(node_memory_Cached_bytes[1h]) + avg_over_time(node_memory_Buffers_bytes[1h])) / avg_over_time(node_memory_MemTotal_bytes[1h]))) > 20
          for: 1m
          labels:
            team: node
删除monitoring中的prometheus实例，触发告警
```

>将alertmanager暴露在集群外部，vi  kube-prometheus/manifests/alertmanager-service.yaml
>
>spec:
>
>  ​	type: NodePort

#### 5、alertmanager监控告警之将告警分发至邮箱或者钉钉等告警媒介中

>在prometheus-operator alertmanager中，通过不同的路由发送告警，步骤：

* 1、创建secret

* 2、通过写spec.alertmanagerConfigSelector主配置。

* 3、spec.alertmanagerConfiguration

> 查看生成的secret：kubectl get secret/alertmanager-main -n monitoring -o json | jq .data
>
> #解密：echo xxxx | base64 -d

![Snipaste_2026-05-11_17-53-30](pict\Snipaste_2026-05-11_17-53-30.png)

##### 5.1将钉钉引入集群：

```
1、cp monitor/alert-dep-svc.yaml dingding-dep-svc.yaml (kube-prometheus/manifests/rules/)
# 创建dingding的dep-svc
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dingding
  namespace: monitoring
  labels:
    app: dingding
spec:
  selector:
    matchLabels:
      app: dingding
  template:
    metadata:
      labels:
        app: dingding
    spec:
      containers:
      - name: dingding
        image: registry.cn-chengdu.aliyuncs.com/mr5/dingding:v1
        imagePullPolicy: IfNotPresent
        env:
        - name: DURL
          value: 钉钉机器人的webhook地址
---
apiVersion: v1
kind: Service
metadata:
  name: dingding
  namespace: monitoring
  labels:
    app: dingding
spec:
  selector:
    app: dingding
  prots:
  - name: web
    port: 8080
    targetPort: 8080

：8080/alertmanager/webhook

```

##### 5.2、配置alertmanager的告警路由和接受者：

> kubectl explain AlertmanagerConfig.spec |less

```
vi kube-prometheus/manifests/rules/dingding-alert.yaml
# alertmanager-config.yaml
apiVersion: monitoring.coreos.com/v1alpha1
kind: AlertmanagerConfig
metadata:
  name: dinghook
  namespace: monitoring
  labels:
    alertmanagerConfig: example(与alertmanager-alaertmanager.yaml文件的alertmanagerConfigSelector标签中要一致。)
spec:
  receivers:
    - name: dingding
      webhookConfigs:
        - url: http://dingding:8080/alertmanager/webhook
          sendResolved: true
  route:
    groupBy: ['instance']
    groupWait: 30s
    groupInterval: 30s
    repeatInterval: 1m
    receiver: dingding
    matchers:
    - name: team 
      value: node
2、修改全局配置：
vi kube-prometheus/manifests/alertmanager-alertmanager.yaml
spec:
  alertmanagerConfigSelector:
    matchLabels:
      alertmanagerConfig: example
  ...
alertmanagerConfig: example（全局）
```

#### 6、将prometheus数据进行持久化：

```
1、vi kube-prometheus/manifests/rules/prometheus-operator-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: promrtheus-operator-local
  labels:
    app: prometheus
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  storageClassName: local-storage
  local:
    path: /data/k8s/prometheus-operator
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node1.y2312.com
  persistentVolumeReclaimPolicy: Retain
2、vi kube-prometheus/manifests/prometheus-prometheus.yaml
...
数据持久化: 
spec:
  storage:
    volumeClaimTemplate:
      spec:
        storageClassName: local-storage
          resources:
            requests:
              storage: 10Gi
如将数据进行持久化，需要创建pv和写入volumeClaimTemplate,自然会自动创建pvc（在statefulset中）
```

![Snipaste_2026-03-28_11-09-36](pict\Snipaste_2026-03-28_11-09-36.png)