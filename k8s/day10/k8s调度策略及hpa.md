## k8s调度策略、QoS服务质量、hpa

#### 1、k8s调度策略：

```
k8s调度策略 
	1：预选过程，过滤掉不满足条件的节点，这个过程称为 Predicates
	2：优选过程，对通过的节点按照优先级排序，称之为 Priorities（打分）
nodeSelector 
vi pod1.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: busybox-pod
  name: test-busybox
spec:
  containers:
  - command:
    - sleep
    - "3600"
    image: busybox
    imagePullPolicy: Always
    name: test-busybox
  nodeSelector:
    com: mr5===标签为mr5
node1没有mr5这个标签，所以打标签：
kubectl label node/www.y2312node1.com com=mr5
查看是否打上标签： kubectl get node --show-labels
将标签去掉： kubectl label node/www.y2312node1.com com-

处于pending状态的原因：标签|资源（cou mem）| 存储


  软策略就是如果现在没有满足调度要求的节点的话，Pod就会忽略这条规则，继续完成调度过程，说白了就是满足条件最好了，没有的话也无
  硬策略就比较强硬了，如果没有满足条件的节点的话，就不断重试直到满足条件为止，简单说就是你必须满足我的要求，不然就不干
```

#### 2、pod亲和性:

##### 2.1、nodeAffinity:node的亲和性

```
affinity：亲和性
	1、nodeAffinity
	2、podAffinity
	3、podAntiAffinity
1、nodeAffinity:亲和性
# node-affinity-demo.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-affinity
  labels:
    app: node-affinity
spec:
  replicas: 1
  selector:
    matchLabels:
      app: node-affinity
  template:
    metadata:
      labels:
        app: node-affinity
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
          name: nginxweb
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:  # 硬策略
            nodeSelectorTerms: 
            - matchExpressions: #正则表达式
              - key: kubernetes.io/hostname #k
                operator: In #操作符有in exists gt lt notin...
                values: # v
                - node3.mr5.com
          preferredDuringSchedulingIgnoredDuringExecution:  # 软策略
          - weight: 1   ===权重
            preference:
              matchExpressions:
              - key: com
                operator: In
                values:
                - mr5 
```

##### 2.2、pod的亲和性

```
2、pod亲和性 
查看pod的标签：kubectl get pod --show-labels===>app=node-affinity,
# pod-affinity-demo.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-affinity
  labels:
    app: pod-affinity
spec:
  replicas: 3
  selector:
    matchLabels:
      app: pod-affinity
  template:
    metadata:
      labels:
        app: pod-affinity
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: nginxweb
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:  # 硬策略
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - node-affinity # pod的标签
            topologyKey: kubernetes.io/hostname ===带标签

node1掉了：1、systemctl restart docker 2、systemctl restart kubelet

```

##### 2.3、pod的反亲和性：

```
3、pod 反亲和性 
# pod-antiaffinity-demo.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-antiaffinity
  labels:
    app: pod-antiaffinity
spec:
  replicas: 3
  selector:
    matchLabels:
      app: pod-antiaffinity
  template:
    metadata:
      labels:
        app: pod-antiaffinity
    spec:
      containers:
      - name: nginx
        image: nginx
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
          name: nginxweb
      affinity:
        podAntiAffinity: # 
          requiredDuringSchedulingIgnoredDuringExecution:  # 硬策略
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - node-affinity
            topologyKey: kubernetes.io/hostname
```

终端删除整行ctrl+u删除命令行开始至光标处、ctrl+k删除光标处至命令行结尾、ctrl+a光标移动到最前面、ctrl+e光标移动到最后面。 

#### 3、k8s污点容忍度：

```
污点与容忍度 
查看某个污点：
kubectl get node/www.y2312master.com | less
effect：效果
1、NoSchedule ：表示 k8s 将不会将 Pod 调度到具有该污点的 Node 上。

2、PreferNoSchedule ：表示 k8s 将尽量避免将 Pod 调度到具有该污点的 Node 上。

3、NoExecute ：表示 k8s 将不会将 Pod 调度到具有该污点的 Node 上，同时会将 Node 上已经存在的 Pod 驱逐出去

给某个节点打上污点：
kubectl taint nodes www.y2312node1.com hobby=smoke:NoSchedule

容忍污点：
先给node1上打上标签： kubectl label node/www.y2312node1.com com=mr5
----容忍master------
# taint-demo.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: taint
  labels:
    app: taint
spec:
  replicas: 3
  selector:
    matchLabels:
      com: mr5
  template:
    metadata:
      labels:
        com: mr5
    spec:
      nodeSelector: # 此时node1上有污点又有标签com=mr5，运行后node1处于pending状态
        com: mr5
      containers:
      - name: nginx
        image: nginx
        ports:
        - name: http
          containerPort: 80
     或者：
     spec:
      nodeSelector: # 此时node1上有污点又有标签com=mr5，运行后node1处于pending状态
        com: mr5
      tolerations：====容忍度，在node1上面可以运行
      - key：hobby
        operator：Equal或者Exists
        value: smoke
        effect： NoSchedule===删除的话只匹配kv（容忍的程度）
        .....后续跟上面一样

对于 tolerations 属性的写法，其中的 key、value、effect 与 Node 的 Taint 设置需保持一致， 还有以下几点说明：

如果 operator（操作符） 的值是 Exists，则 value 属性可省略，就是容忍一切污点
如果 operator 的值是 Equal，则表示其 key 与 value 之间的关系是 equal(等于)
如果不指定 operator 属性，则默认值为 Equal
另外，还有两个特殊值：

空的 key 如果再配合 Exists 就能匹配所有的 key 与 value，也就是是能容忍所有节点的所有 Taints
空的 effect 匹配所有的 effect

spec:
      tolerations:
      - operator: Exists====容忍一切标签包括master
```

#### 4、pod 的QoS： 服务质量

​	定义pod的优先级

```
pod的QoS服务质量：
1、Guaranteed：vvvvvip，资源的下限和上线都保持一致requests和limits一致
2、Burstable：requests 设定下线
3、BestEffort：尽力而为（没有限制）
vi request.yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: nginx:latest
    resources:
      requests:====指定下线
        memory: "64Mi"===64M的内存
        cpu: "250m"===250微核的，1核=1000微核

      或者：
      limits：===指定上线
      	memory: "64Mi"===64M的内存
        cpu: "250m"===250微核的，1核=1000微核

```

#### 5、hpa：水平扩展

> 安装metric-server

```
hpa： 通过deployment来控制pod的数量，当前指定5个pod来应对当前的负载，可以应付当前的，随着用户的增多，5个pod的压力在增大，这个5个pod不能适应当前的负载，此时需要扩容===需要自动扩展，当cpu达到70%是设定阈值，当超过这个阈值，就表示压力过大，然后就自动扩大负载，自动扩载有两种方式： 1纵向扩载：给pod资源再加大  2横向扩载：增加pod的数量，也就是增加处理任务的能力加大（负载均衡的原理），压力就减小了，当峰值的时候过了pod数量多了，就会把多余的pod删了。安装metric-server来监控pod的cpu和mem
在github官网搜索：metric-server，安装kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml，跳过证书认证和修改拉去镜像地址： - --kubelet-insecure-tls
        image: registry.cn-chengdu.aliyuncs.com/mr5/metric-server:v0.6.4
当前就可以查看节点node的cpu和内存
[root@www hpa]# kubectl top node
NAME                  CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
www.y2312master.com   353m         17%    1522Mi          55%       
www.y2312node1.com    178m         8%     1211Mi          33%       
www.y2312node2.com    77m          3%     1788Mi          45% 

```

##### 5.1、水平扩展之cpu

```
查看内存及内存使用率：kubectl top node
查看pod： [root@www schedu]# kubectl get pod pod-cm1
此时做水平扩展：
安装一个pod
[root@www y2312mainfeast]# mkdir hpa
[root@www y2312mainfeast]# cd hpa/
vi dep1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpa-demo
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
        resources:
          requests:===内存50M，cpu为50微核
            memory: 50Mi
            cpu: 50m
k9s中进入default====hpa-demo===找到ip：10.244.1.137  并发起压力测试

先测试网络：[root@www hpa]# curl http://10.244.1.137  
复制会话： ab -n 10000 -c 100 http://10.244.1.137/(yace)
查看内存等：[root@www hpa]# kubectl top pod hpa-demo-6b4d9d86-z56sr
设定阈值： kubectl autoscale deployment hpa-demo --cpu-percent=10 --min=1 --max=10
                                         cpu使用的百分比  最小为1  最大为10

查看使用率： kubectl get hpa

扩大cpu
scaleDown：缩容（窗口期300s）
scaleUp：扩容（60s）
[root@www hpa]# kubectl get hpa/hpa-demo -o yaml > cpuhpa.yaml 
vi cpuhpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-demo
  namespace: default
spec:
  maxReplicas: 10
  metrics:==衡量指标
  - resource:
      name: cpu
      target:
        averageUtilization: 10  ==目标利用率
        type: Utilization
    type: Resource
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-demo
kubectl explain hpa.spec.behavior | less ===查看扩容和缩容
```

##### 5.2、水平扩展之mem

```
扩大内存（压测内存的脚本）：
vi mem-cm.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: increase-mem-config
data:
  increase-mem.sh: |  ==创建一个文件
    #!/bin/bash
    mkdir /tmp/memory  ===创建临时目录
    mount -t tmpfs -o size=40M tmpfs /tmp/memory  ==挂载，将40M挂载到临时目录中
    dd if=/dev/zero of=/tmp/memory/block  将这个文件导入到临时文件中
    sleep 60====60s后就删除
    rm /tmp/memory/block
    umount /tmp/memory
    rmdir /tmp/memory

vi hpa-mem-dep.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpa-mem-demo
spec:
  selector:
    matchLabels:
      app: nginx-mem
  template:
    metadata:
      labels:
        app: nginx-mem
    spec:
      volumes:
        - name: increase-mem-script
          configMap:
            name: increase-mem-config
      containers:
        - name: nginx
          image: nginx:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
          volumeMounts:
            - name: increase-mem-script
              mountPath: /etc/script
          resources:
            requests:
              memory: 50Mi
              cpu: 50m
          securityContext:
            privileged: true

k9s中找到hpa-mem-demo名称
内存通过写yaml文件来创建
vi memhpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-mem-demo
  namespace: default
spec:
  maxReplicas: 5
  metrics:
  - resource:
      name: memory
      target:
        averageUtilization: 30
        type: Utilization
    type: Resource
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-mem-demo

获取资源：
[root@www hpa]# kubectl get hpa
NAME           REFERENCE                 TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
hpa-demo       Deployment/hpa-demo       0%/10%    1         10        1          30m
hpa-mem-demo   Deployment/hpa-mem-demo   11%/30%   1         5         1          20s

对内存进行压测：
k9s进入default中的hpa-mem-demo中shell：bash /etc/script/increase.mem 
```



