# k8s生命周期及控制器

## 一、生命周期

#### 1、生命周期postStart之livenessProbe

​	livenessProbe存活检测，检测不通过就会重启pod

```
1、livenessProbe（存活检测，检测不通过重启pod）
vi pod11.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod11
  namespace: dev
  labels:
    app: web-front11
spec:
  containers:
  - name: web
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    lifecycle:
      postStart:
        exec:
          command:
          - /bin/sh
          - -c
          - echo "poststart" > /usr/share/nginx/html/health.html
    livenessProbe:   ====与lifecycle同级        
      initialDelaySeconds: 3===》初始化准备3s
      periodSeconds: 1=====》每隔一秒检测一次
      httpGet:
        path: /health.html
        port: 80

在shell删除usr/share/nginx/html/health.html，不会报错，会有存活检测，删了可以再起来。
```

#### 2、生命周期postStart之readinessProbe

​	readinessProbe就绪检测，检测不通过pod就不会被svc调度

	initContainers===>postStart
	           					livenessProbe（存活检测，检测不通过重启pod）
	      						readinessProbe（就绪检测，检测不通过pod，pod不会被service调度）
														preStop（完成后的清理，停止前执行的脚本，主要用于优雅退出清理资源）
	
	1、readinessProbe:=====就绪检测，检测不通过pod不会被service调度
	vi pod12.yaml
	apiVersion: v1
	kind: Pod
	metadata:
	  name: pod12
	  namespace: dev
	  labels:
	    app: web-front12
	spec:
	  containers:
	  - name: web
	    image: nginx:latest
	    imagePullPolicy: IfNotPresent
	    lifecycle:
	      postStart:
	        exec:
	          command:
	          - /bin/sh
	          - -c
	          - echo "poststart" > /usr/share/nginx/html/health.html
	    readinessProbe:=====就绪检测，检测不通过pod不会被service调度
	      initialDelaySeconds: 3
	      periodSeconds: 1
	      httpGet:   ===访问页面
	        path: /health.html
	        port: 80
	k9s中nginx删除health.htmlpod现红

#### 3、preStop:===结束后清理

​	执行优雅退出、清理资源

```
101：优雅退出和清理资源
[root@www ~]# touch /webroot/clean===需要清理
vi pod13.yaml
apiVersion: v1
kind: Pod
metadata:
	 name: pod13
	 namespace: dev
	 labels:
	    app: web-front13
spec:
	 nodeName: www.y2312node1.com指定节点
	 volumes:====挂载
  	 - name: webdir
       hostPath:
           path: /webroot/
           type: Directory
      containers:
      - name: web
        image: nginx:latest
        imagePullPolicy: IfNotPresent
        volumeMounts:
        - name: webdir
          mountPath: /usr/share/nginx/html/
        lifecycle:
          preStop:===结束后清理clean
            exec:
                command: ["rm","-rf", "/usr/share/nginx/html/clean"]
  kubectl  apply -f pod13.yaml
  k9s 中删除pod13.yaml 
  查看101： ls /webroot/====》里面没有clean
```

## 二、控制器：

#### 1、replicaSet：

​	资源 控制pod数量，期望值与当前值保持一致

>==查看k8s内置的资源==：kubectl api-resources

##### 1.1、replicaset控制pod的数量

```
[root@www y2312mainfeast]# kubectl get replicaset -A===查询有几个资源
查看有哪些资源(内置的资源)： kubectl api-resources | less

vi rs1.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs1
  namespace: prod
  labels:
    app: web
spec: 
  replicas: 10===》运行数量为10个
  selector:===控制器
    matchLabels:
      app: rs1
  template:  ===模板
    metadata:
      labels:
        app: rs1
    spec:
      containers:
      - name: 
        image: nginx:latest
        imagePullPolicy: IfNotPresent
[root@www y2312mainfeast]# kubectl apply -f rs1.yaml
replicaset.apps/rs1 created
[root@www y2312mainfeast]# kubectl get pod -n prod==查询prod名称空间运行数量
NAME        READY   STATUS    RESTARTS   AGE
rs1-5m448   1/1     Running   0          47s
rs1-6l6s2   1/1     Running   0          47s
rs1-8lln6   1/1     Running   0          47s
rs1-dl654   1/1     Running   0          47s
rs1-jdz54   1/1     Running   0          47s
rs1-q974s   1/1     Running   0          47s
rs1-qf7bm   1/1     Running   0          47s
rs1-tp27x   1/1     Running   0          47s
rs1-xc65n   1/1     Running   0          47s
rs1-zwbcr   1/1     Running   0          47s
k9s中删除rs1中的其中一个删不了，当删除其中一个就会再次创建一个。


```

##### 1.2、replicaset控制pod的版本更新

```
如何对pod的版本进行更新： 
vi rs2.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs2
  namespace: prod
  labels:
    app: web
spec:
  replicas: 5===跑5个pod可以扩容
  selector:
    matchLabels:
      app: rs2
  template:
    metadata:
      labels:
        app: rs2
    spec:
      containers:
      - name: web
        image: registry.cn-chengdu.aliyuncs.com/mr5/y2305:v01| 02.。。。
        imagePullPolicy: IfNotPresent
k9s中找到prod中pod的地址：[root@www y2312mainfeast]# curl 10.244.1.24==
<h1>v1</h1>
删除重新拉去镜像版本v02时 curl ip地址===<h1>v2</h1>了
```

#### 2、deployment：

​	作用：资源控制pod数量及pod版本（无状态，一般是指不做数据持久化的）

>rs控制pod数量

##### 2.1、deployment控制pod数量

```
vi dep1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dep1
  namespace: prod
  labels:
    app: dep1
spec:
  replicas: 2
  selector:
    matchLabels:
      app: dep1
  template:
    metadata:
      labels:
        app: dep1
    spec:
      containers:
      - name: web
        image: registry.cn-chengdu.aliyuncs.com/mr5/y2305:v01|  02等
        imagePullPolicy: IfNotPresent
配置清单修改：
v01版本时：<h1>v1</h1>
v02版本时：<h1>v2</h1>。。。。。
[root@www y2312mainfeast]# kubectl get rs -n prod===查看dep1.yaml运行了几次（更新了几个版本）

```



![1710406415472](pict\1710406415472.png)

```
kubectl来操作k8s，相当于客户端工具，每次apply都是向apiserver apply，apiserver都是校验是否符合k8s规范（准入控制），只是把它存到etcd数据库，判断有没有权限来创建这个东西，scheduler只是调度pod，kubelet时刻和apiserver通信，kubelet去调用底层容器接口去创建pod，controller-manager时刻和apiserver通信检测pod

api-server：准入控制、权限控制

controller： 负责把某个具体的资源运行起来，就像空调遥控器，生命式api：当前值与期望值保持一致

strategy：策略===recreate（在创建之前先杀掉之前的）和rollingupdate（滚动更新）
```

##### 2.2、deployment命令查看：

```
[root@www y2312mainfeast]# kubectl edit deploy/dep1 -n prod===》页面直接编辑修yaml一样
[root@www y2312mainfeast]# kubectl rollout history deployment/dep1 -n prod==回滚查询变了几次版本
[root@www y2312mainfeast]# kubectl rollout undo deployment/dep1 -n prod==回滚到上一个版本
[root@www y2312mainfeast]# kubectl rollout undo  deployment/dep1 --to-revision=2  -n prod==指定回到那个版本

```

#### 3、daesonset：

​	pod数量和node数量保持一致，每个node上只运行一个pod（无状态）

```
vi daemon1.yaml
apiVersion: apps/v1
kind: DaemonSet===》
metadata:
  name: daemon1
  namespace: test
  labels:
    app: daemon1
spec:
  selector:
    matchLabels:
      app: daemon1
  template:
    metadata:
      labels:
        app: daemon1
    spec:
      containers:
      - name: web
        image: registry.cn-chengdu.aliyuncs.com/mr5/y2305:v01
        imagePullPolicy: IfNotPresent
[root@www y2312mainfeast]# kubectl apply -f daemon1.yaml
[root@www y2312mainfeast]# kubectl get pod -n test===》有两个，是因为master有污点，默认不能容忍污点的
NAME            READY   STATUS    RESTARTS   AGE
daemon1-4wk97   1/1     Running   0          48s
daemon1-8tr6b   1/1     Running   0          48s
pod2            1/1     Running   0          9h
```

>
>master上默认有污点，daemonset调度的时候就不会调度到master上。

#### 4、控制器：job

​	执行一次任务

```
2、执行定时任务：
vi job1.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: test-job
  namespace: dev
spec: 
  template:
    metadata:
      name: test-pod
    spec:
      backoffLimit: 4
      containers:
      - name: test-container
        image: busybox:latest
        command:
        - /bin/sh
        - -c
        - "echo helloworld"
        restartPolicy: Never  ===重启策略
      
```



#### 5、控制器：cronjob定时任务

```
3、每分钟执行一次busybox：latest镜像
vi cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: test-cronjob
  namespace: dev
spec:
  schedule: "* * * * *"  ===每分钟
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            app: test
        spec:
          restartPolicy: Never
          containers:
          - name: test-container
            image: busybox:latest
            command:
            - /bin/sh
            - -c
            - echo "hello world"
          
```

控制器： statefulset有状态的pod控制器（需要借助存储）===先略过

=======>operator==protheume

   									mysql