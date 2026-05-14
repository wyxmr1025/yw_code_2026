## k8s-service

### 一、service前提知识

#### 1、svc前提：

```
service：将pod调度起来，负载均衡调度规则（kube-proxy： ipvs|iptables）

	基于名称来访问，跨名称空间 ==svc_name.ns_name.svc.cluster.local

coredns：service创建时自动将ip和名称注册到coredns==先解决集群内部的dns解决不了在解决宿主机的dns
pod和pod访问可以基于名称：
	curl svc1(service的名称)
```

#### 2、service的type：

```
1、clusterip pod集群内部的通信

2、nodeport 将service暴露在集群外部

3、ExternalName===将外部服务器引入到集群内部

	自定义1、endpoints和2、externalname

4、loadBalancer： 公有云负载均衡器

endpoints: 

pod和pod可以基于名称来访问


```

### 二、service的类型

##### 1、service的类型之ClusterIP：

```
vi svc1.yaml
apiVersion: v1
kind: Service
metadata:
  name: svc1
  namespace: prod
  labels:===》svc自己的标签，不是被监视的标签
    app: svc1
spec:
  ports:
  - name: web
    port: 80====》自己的端口
    targetPort: 80=====》被监视的端口（nginx）
  selector:
    app: dep1====》监视的pod
  type: ClusterIP
[root@www ~]# kubectl apply -f svc1.yaml 
k9s中输入：svc===得到10.100.219.235ip
在节点上curl：
[root@www ~]# curl  10.100.219.235
<h1>v2</h1>
在k9s的ns==testshell中curl 10.100.219.235也可以访问===》pod通过service来访问约等于调度器
新加一个pod到prod中：此时基于名称来访问svc1
shell进入pod-prod.yaml=k9s
root@pod1:/# curl svc1
<h1>v2</h1>
root@pod1:/# cat /etc/resolv.conf 
nameserver 10.96.0.10

===k9s找到ns中的kube-system==》搜索svc===kube-dns里面ip一样
删除100、101、102中/etc/resolve.conf
search master|node1和2

root@pod1:/# curl svc1
<h1>v2</h1>

root@pod1:/# curl svc1.prod.svc.cluster.local==名称.名称空间.资源类型.后两个集群内部
<h1>v2</h1>
跨名称空间通信：
在k9s中找到dev进入pod1==root@pod1:/# curl svc1.prod.svc.cluster.local
<h1>v2</h1>

```

##### 2、service的类型之nodeport

​	作用：nodeport 将service暴露在集群外部

```
vi svc2.yaml
apiVersion: v1
kind: Service
metadata:
  name: svc2
  namespace: prod
  labels:
    app: svc2
spec:
  ports:
  - name: web
    port: 80
    targetPort: 80
  selector:
    app: dep1
  type: NodePort
===进入k9s===找到svc2===端口为web:80►31169
浏览器中输入：192.168.18.100：31169===出现v2===》在每个集群上做一个端口映射

将外部服务器引入到集群内部==externalname
```

##### 3、service的类型之externalname

​	作用：将外部服务器引入到集群内部==externalname

```
在101上安装httpd==在集群外部
yum install httpd
systemctl start httpd ===(开启80 端口)
[root@www ~]# echo "this is www.y2312node1.com" > /var/www/html/index.html
[root@www ~]# systemctl start httpd
[root@www ~]# curl 192.168.18.101
this is www.y2312node1.com
100:
vi svc3.yaml
apiVersion: v1
kind: Service
metadata:
  name: svc3
  namespace: prod
  labels:
    app: svc3
spec:
  externalName: externalsvc===别名真正的名字是svc3
  ports:
  - name: web
    port: 80
    targetPort: 80
  type: ExternalName
```

##### 4、service的类型之endpoints

svc并没有关注pod而是关注的endpoints，svc和endpoints关联就要求名称一样

```
1、endpoints来访问
vi endpoints.yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: exsvc===》真正的名字
  namespace: prod
subsets:
- addresses:
  - ip: 192.168.18.101
  ports: ===外部名称和端口
  - name: web
    port: 80
kubectl apply -f endpoints.yaml

再创建一个service，与endpoints同名
vi exsvc.yaml
apiVersion: v1
kind: Service
metadata:
  name: exsvc===同名
  namespace: prod
spec:
  ports:
  - name: web
    port: 80
    targetPort: 80
[root@www y2312mainfeast]# kubectl apply -f exsvc.yaml
k9s中ns==prod==pod1中shell打开：
root@pod1:/# curl exsvc
this is www.y2312node1.com==通过集群访问内部的service

```

##### 5、通过别名来访问

```
2、通过externalname===别名来访问service
vi xxx.yaml
apiVersion: v1
kind: Service
metadata:
  name: svc8-----解析到别名上
  namespace: prod
spec:
  externalName: www.xxx.com--别名
  type: ExternalName
  
 k8s coredns 添加一条解析记录：
k9s===kube-system==configmap===第一个进入编辑模式：
hosts {
             192.168.18.101 www.xxx.com
             fallthrough
        }
k9s===prod===pod1==shell：
root@pod1:/# curl www.xxx.com
this is www.y2312node1.com
或者：root@pod1:/# curl svc8 
this is www.y2312node1.com
```

>命令行==添加coredns名称解析==：kubectl -n kube-system edit configmap coredns