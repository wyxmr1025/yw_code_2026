## k8s基础

### 1、k8s基本知识

pod：容器最小调度单位 一组容器 共享网络空间

查看pod：

```
kubectl get node
```

查看服务： 

```
kubectl get service
```

名称空间，namespace：

```
逻辑的将一些资源进行分组
```

创建k8s：kubectl apply|create|get|describe|delete

```
1、查询名称空间：
[root@www ~]# kubectl get namespace
NAME              STATUS   AGE
default           Active   176m
kube-flannel      Active   163m
kube-node-lease   Active   176m
kube-public       Active   176m
kube-system       Active   176m

2、查询kube-system的名称空间：
[root@www ~]# kubectl get pod -n kube-system

2.1创建k8s空间 通过命令或者配置清单
创建dev和test两个名称空间
命令形式：
[root@www ~]# kubectl create ns dev===创建dev名称空间
namespace/dev created
[root@www ~]# kubectl create ns test=====创建test名称空间
查询名称空间：
kubectl get ns

2.2配置清单：指定属于什么资源版本
apiVersion：属于资源组的那个资源版本
kind：资源类型
metadata：属于那些名称资源
spec：具体哪些资源是怎么运行的
status：状态（选填）
--pod-network指定pod的网络
--service-network指定service的网络
例：
指定名称空间test为yaml格式
[root@www ~]# kubectl get ns test -o yaml
[root@www ~]# mkdir y2312mainfeast
[root@www ~]# cd y2312mainfeast/
vi ns.yaml
apiVersion: v1
kind: Namespace
metadata: 
  name: prod
创建：[root@www y2312mainfeast]# kubectl create -f ns.yaml 
[root@www y2312mainfeast]# kubectl apply -f ns.yaml
查看：[root@www y2312mainfeast]# kubectl get ns===出现prod
删除： kubectl delete -f ns.yaml
intefer:整型
string：字符串
map[string]string: kv键值对
[]container: 切片===类似于python的列表

[root@www y2312mainfeast]# kubectl get pod/coredns-66f779496c-jczqt -n kube-system -o yaml

查看pod中apiVersion使用
[root@www y2312mainfeast]# kubectl explain pod.apiVersion|kind|metadata。。。

vi pod1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  namespace: dev
  labels:
    app: web-front
spec:
  containers:
  - name: web1
    image: nginx:latest
    imagePullPolicy: IfNotPresent
[root@www y2312mainfeast]# kubectl apply -f pod1.yaml 
pod/pod1 created
[root@www y2312mainfeast]# kubectl get pod -n dev==查询pod中dev
[root@www y2312mainfeast]# kubectl describe pod/pod1 -n dev===查询具体进度
[root@www y2312mainfeast]# kubectl get pod -n dev -o wide===查询pod详细信息
pod1   1/1  Running   0   7m7s   10.244.1.2   www.y2312node2.com   <none>   

```

### 2、被外界访问的方式：

​	端口映射 、共用宿主机网络

```
k9s客户端：
[root@www ~]# mv k9s /usr/bin/
vi pod1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod2
  namespace: test
  labels:
    app: web-front
spec:
  containers:
  - name: web
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    ports:
    - name: webport
      containerPort: 80
      hostPort: 8090===》端口映射
被外界访问的方式：1、端口映射 2、共用宿主机网络
[root@www y2312mainfeast]# kubectl apply -f pod2.yaml 
[root@www y2312mainfeast]# kubectl get pod -n test -o wide===查询安装进度
[root@www y2312mainfeast]# kubectl get pod -n test -o wide==查看分配到那个地址上
pod2   1/1     Running   0          3m53s   10.244.2.2   www.y2312node1.com   
===浏览器中输入：192.168.18.101：8090   出现nginx页面
不建议： 有专门的资源将pod暴露

===使用宿主机的网络
vi pod3.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod3
  namespace: dev
  labels:
    app: web-front
spec:
  hostNetwork: true===》》》添加网络，使用宿主机网络
  containers:
  - name: web
    image: nginx:latest
    imagePullPolicy: IfNotPresent

```

### 3、指定运行在那个节点上 ：

#### 	3.1nodeName（直接指定）

```
1、直接指定
vi pod4.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod4
  namespace: dev
  labels:
    app: web-front4
spec:
  nodeName: www.y2312node1.com  ====指定运行在哪个节点
  containers:
  - name: web
    image: nginx:latest
    imagePullPolicy: IfNotPresent

kubectl delete -f pod4.yaml
kubectl apply -f pod4.yaml
kubectl get pod -n dev -o wide====》。》》》查看运行在哪个节点上
```



>master上默认有污点，pod不会调度上去，除非能容忍污点
>
>查看node上的标签： kubectl get node --show-labels（标签为kv键值对存在）

#### 3.2、 nodeSelector（标签选择器）

```
2、通过标签指定：[root@www y2312mainfeast]# kubectl get node --show-labels
vi pod5.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod5
  namespace: dev
  labels:
    app: web-front5
spec:
  nodeSelector:====标签选择器
    noderole: web
  containers:
  - name: web
    image: nginx:latest
    imagePullPolicy: IfNotPresent
[root@www y2312mainfeast]# kubectl label node/www.y2312node2.com noderole=web
	node/www.y2312node2.com labeled====》》》》》给node2.com上打标签
[root@www y2312mainfeast]# kubectl get node --show-labels==查看标签是否打上
[root@www y2312mainfeast]# kubectl get pod -n dev==》》》》查看状态
```

>给某个node打上标签：kubectl label node/node1.y2312.com noderole=web
>
>查看node标签：kubectl get node --show-labels
>
>调度策略：预选和优选（根据其cpu、内存等资源进行打分）



### 4、一个pod跑多个容器

​	作用：可以一个pod用来跑数据，一个pod用来采集日志log

```
vi pod6.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod6
  namespace: dev
  labels:
    app: web-front6
spec:
  containers:
  - name: web
    image: nginx:latest
    imagePullPolicy: IfNotPresent
  - name: test
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    args: ["sleep","3600"] # pod运行起来就要求运行在前台
[root@www y2312mainfeast]# kubectl apply -f pod6.yaml 

命令：[root@www y2312mainfeast]# kubectl exec -it pod6 -c test -n dev -- /bin/sh
===》》》》进入交互式页面

k9s进入交互式页面
选中pod6===s===进入shell中
busybox程序
/ # wget -O - -q http://127.0.0.1===页面，大O
nginx中
root@pod6:/# echo "this is y2312page" > /usr/share/nginx/html/index.html
注：一个pod可以跑多个容器，也就是说可以一个跑业务，一个收集日志
```

>k8s运行容器时可以交互式进入： kubectl exec -it pod_name -c container_name -n ns_name -- /bin/sh
>
>

### 5、挂载

​	作用： 要求数据持久化，可以采用挂载或者pv-pvc

#### 5.1挂载在宿主机上：

```
101：准备，挂载到宿主机上
[root@www ~]# mkdir /webroot/
[root@www ~]# echo "this is www.y2312node1.com" > /webroot/index.html

100master：
vi pod7.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod7
  namespace: dev
  labels:
    app: web-front7
spec:
  nodeName: www.y2312node1.com  ===指定节点
  volumes:   =====挂载
  - name: webdir
    hostPath:
      path: /webroot/
      type: Directory
  containers:
  - name: web
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    volumeMounts:  ===使用挂载
    - name: webdir
      mountPath: /usr/share/nginx/html/
[root@www y2312mainfeast]# kubectl apply -f pod7.yaml
k9s===pod7--s中
root@pod7:/# cat /usr/share/nginx/html/index.html
this is www.y2312node1.com
在101中修改 echo "this is www.y2312node1.com......" > /webroot/index.html
在k9s中也可以看到
root@pod7:/# cat /usr/share/nginx/html/index.html
this is www.y2312node1.com.......

```

#### 5.2、容器之间使用同一个挂载分区：emptyDir(容器之间的数据共享)

​	作用：容器之间的数据共享，但是当容器结束后，数据就不能持久化，主要用于容器间的数据共享。

```
两个nginx和busybox都使用同一个挂载分区：
vi pod8.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod8
  namespace: dev
  labels:
    app: web-front8
spec:
  volumes:
  - name: webdir
    emptyDir:   ===容器之间的数据共享，当pod完了之后并不能做持久化。
     sizeLimit: 100Mi   ===指定大小 
  containers:
  - name: web
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: webdir
      mountPath: /usr/share/nginx/html/
  - name: test
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    args: ["sleep","3600"]
    volumeMounts:
    - name: webdir
      mountPath: /webroot/
[root@www y2312mainfeast]# kubectl apply -f pod8.yaml 
登录k9s中nginx输入：
root@pod8:/# echo "xxx" > /usr/share/nginx/html/index.html
root@pod8:/# cat /usr/share/nginx/html/index.html 
xxx
在k9sbusybox中输入
/ # cat /webroot/index.html 
xxx
```

### 6、pod的生命周期

#### 6.1、pod生命周期示意图：

![image-20250304115318685](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250304115318685.png)

```
pod生命周期： 

	initContainers（初始化容器）===>postStart
	           					livenessProbe（存活检测）
	      						readinessProbe（就绪检测）
														preStop（完成后的清理）

分为初始化容器和主容器，在初始化容器启动完成后才启动主容器，在主容器启动成功后会启动post start hook的脚本，执行完脚本后会启动两个检测，检测完成后会启动prestop hook的脚本优雅退出。

```

#### 6.2、通过initContainer初始化页面

```
1、通过initContainer初始化页面
vi pod9.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod9
  namespace: dev
  labels:
    app: web-front9
spec:
  volumes:
  - name: webdir
    emptyDir:
     sizeLimit: 100Mi
  initContainers:
  - name: initwebpage
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    args: ["/bin/sh","-c","echo 'hello world' > /webroot/index.html"]
    volumeMounts:
    - name: webdir
      mountPath: /webroot/
  containers:
  - name: web
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: webdir
      mountPath: /usr/share/nginx/html/
kubectl apply -f pod9.yaml
k9s==nginx中
root@pod9:/# cat /usr/share/nginx/html/index.html 
hello world
```

#### 6.3、通过postStart初始化页面

```
2、通过postStart初始化页面
vi pod10.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod10
  namespace: dev
  labels:
    app: web-front10
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
          - echo "poststart" > /usr/share/nginx/html/index.html
kubectl apply -f pod10.yaml
k9s中nginx===s
root@pod10:/# cat /usr/share/nginx/html/index.html 
poststart

```

#### 6.4、通过初始化容器运行mysql

启动数据库，并导入数据库.sql

数据库环境变量：

MYSQL_ROOT_PASSWORD

MYSQL_DATABASE

MYSQL_USER

MYSQL_PASSWORD

指定字符集：--character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

实战：

```
102：
先安装mariadb
[root@www ~]# mysql library < test.sql 
grant all privileges on *.* to 'root'@'localhost' identified by '123456'
flush privileges;
pass=123456
mysql -e 'show databases' -p$pass

100:
vi mysql.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql
  namespace: dev
spec:
  nodeName: www.y2312node2.com
  containers:
  - name: mysql
    image: mysql:5.5.62
    imagePullPolicy: IfNotPresent
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: Aa@123456
    - name: MYSQL_DATABASE
      value: library
    - name: MYSQL_USER
      value: zhangsan
    - name: MYSQL_PASSWORD
      value: Aa@123456
[root@www y2312mainfeast]# kubectl apply -f mysql.yaml 
k9s中查看地址：10.244.1.9===

102中输入：
[root@www ~]# mysql -h 10.244.1.9 -uroot -p
[root@www ~]# db=library
[root@www ~]# pass=Aa@123456
[root@www ~]# mysql -h 10.244.1.9 -uroot -p$pass
[root@www ~]# mysql -h 10.244.1.9 -uroot -p$pass $db < test.sql 
[root@www ~]# mkdir /sqlfile
[root@www ~]# mv test.sql /sqlfile/

100（master）：
采用挂载：
vi msyql.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql
  namespace: dev
spec:
  volumes:
  - name: sqldir
    hostPath:
     path: /sqlfile
     type: Directory
  nodeName: www.y2312node2.com
  containers:
  - name: mysql
    image: mysql:5.5.62
    imagePullPolicy: IfNotPresent
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: Aa@123456
    - name: MYSQL_DATABASE
      value: library
    - name: MYSQL_USER
      value: zhangsan
    - name: MYSQL_PASSWORD
      value: Aa@123456
    volumeMounts:
    - name: sqldir
      mountPath: /sqlfile
    lifecycle:
      postStart:
        exec:
          command:
          - /bin/bash
          - -c
          - "mysql -uroot -p$MYSQL_ROOT_PASSWORD $MYSQL_DATABASE < /sqlfile/test.sql"
[root@www y2312mainfeast]# kubectl apply -f mysql.yaml

```

>不建议k8s中跑数据库，会造成数据泄露，这里只是作为项目跑起来。