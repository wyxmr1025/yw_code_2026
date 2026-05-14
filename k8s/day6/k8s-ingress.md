## k8s部署ingress

​	ingress基于七层访问

```
ingress资源------>对应ingress-controller（控制器）--->调度规则（配置文件），配置成调度规则就会重启，ingresee-controller就会自动重载
						nginx-------------------------------》
						haproxy------------------------------》
						traefik------------------------------------》
server {
	listen 80；
	server_name	 www.y2312.com ==替换成deamon1.mr5.com
	location  / {
		proxy_pass  http://svc_name;前端的svc==>替换成my-nginx==当调度y2312.com时就转到前端svc
	}
}
```

#### 1、安装ingress-nginx：

​	文档：https://kubernetes.github.io/ingress-nginx/

```
修改官网的ingress-nginx的 hostNetwork: true nodeName: node1.y2312.com,镜像拉去不下来可以修改镜像images: 阿里云

```

##### 1.1、实例：

```
vi ingress-1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      app: my-nginx
  template:
    metadata:
      labels:
        app: my-nginx
    spec:
      containers:
        - name: my-nginx
          image: nginx:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    app: my-nginx
spec:
  ports:
    - port: 80
      protocol: TCP
      name: http
  selector:
    app: my-nginx
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-nginx
  namespace: default
spec:
  ingressClassName: nginx # 使用 nginx 的 IngressClass（关联的 ingress-nginx 控制器）
  rules:
  - host: deamon1.mr5.com # 将域名映射到 my-nginx 服务
    http:
      paths:
        - path: /
            pathType: Prefix # 只匹配前缀
            backend:
              service: # 将所有请求发送到 my-nginx 服务的 80 端口
                name: my-nginx
                port:
                  number: 80

vi ingress-nginx.yaml
spec:
      hostNetwork: true
      nodeName: www.y2312node1.com====节点

```

##### 1.2、部署controller，将ingress部署在101上面

```
vi ingress-nginx
修改C盘：hosts====》192.168.18.101 deamon1.mr5.com
浏览器中输入：deamon1.mr5.com

```

#### 2、ingress部署nginx实战

##### 2.1、basic认证：弹出用户名和密码框

```
100：
yum install httpd
创建用户名和密码： 
htpasswd -c auth mr5===auth文件的名称，mr5用户的名称
======》密码：123456
将文件保存为secret
[root@www ingress]# kubectl create secret generic basic-auth --from-file=auth

vi ingress-basic.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ingress-dep-basic
spec:
  replicas: 1
  selector:
    matchLabels:
      app: basic
  template:====模版
     metadata:
       labels:
         app: basic
     spec:
       containers:
       - name: web
         image: nginx:latest
         imagePullPolicy: IfNotPresent
---
apiVersion: v1
kind: Service
metadata:
  name: ingress-svc-basic
spec:
  selector:
    app: basic
  ports:
  - name: http
	port: 80
    targetPort: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-with-auth
  annotations:
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required - foo'
spec:
  ingressClassName: nginx
  rules:
  - host: basic.mr5.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: ingress-svc-basic     =====与上方创建的svc名字要一样（要被service调度）
            port:
              number: 80
[root@www ingress]# kubectl apply -f ingress-basic.yaml 

可能会运行不起来： 查看controller的日志：kubectl logs -f ingress-nginx-controller-59c78c84bf-mmsns -n ingress-nginx
k9s===default==ingress==
win7修改hosts===》192.168.18.101 basic.mr5.com

```

##### 2.2、重写

​	当访问/gateway时才会出现nginx页面。相当于将/gateway重写到根目录里后面去了。

```
vi ingress-1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      app: my-nginx
  template:
    metadata:
      labels:
        app: my-nginx
    spec:
      containers:
        - name: my-nginx
          image: nginx:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    app: my-nginx
spec:
  ports:
    - port: 80
      protocol: TCP
      name: http
  selector:
    app: my-nginx
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2  ===当访问$2就是在访问nginx的gateway，才能访问nginx的后台 
  name: my-nginx
  namespace: default
spec:
  ingressClassName: nginx # 使用 nginx 的 IngressClass（关联的 ingress-nginx 控制器）
  rules:
    - host: deamon1.mr5.com # 将域名映射到 my-nginx 服务
      http:
        paths:
          - path: /gateway(/|$)(.*)
            pathType: ImplementationSpecific # 适配
            backend:
              service: # 将所有请求发送到 my-nginx 服务的 80 端口
                name: my-nginx
                port:
                  number: 80
浏览器中访问：deamon1.mr5.com/gateway==在/后面加gateway才会出现nginx页面


```

##### 2.3、https：证书

###### 1、生成证书和私钥:

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=www.y2312.com"
```

###### 2、将证书和私钥保存成secret交给ingress

```
kubectl create secret tls foo-tls --cert=tls.crt --key=tls.key -n prod
查询secret： kubectl get secret
```

###### 3、书写ingress规则：

```
将证书交给ingresscontroller
[root@www ingress]# vi ingress-https.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-with-auth
  namespace: prod
  annotations:
    # 认证类型
    nginx.ingress.kubernetes.io/auth-type: basic
    # 包含 user/password 定义的 secret 对象名
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    # 要显示的带有适当上下文的消息，说明需要身份验证的原因
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required - foo'
spec:
  ingressClassName: nginx
  tls: # 配置 tls 证书
    - hosts:
        - www.y2312.com===表示证书给谁用
      secretName: foo-tls
  rules:
    - host: www.y2312.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: wordpress-svc  ===注意这里要与service名称相同
                port:
                  number: 80
解析域名：
win7中==hosts
192.168.18.101 www.y2312.com
==浏览器中输入：https:/www.y2312.com===出现wordpress页面==私密链接等

```

#### 3、灰度发布：canary deployment（灰度发布或者金丝雀发布）

​	保证几个版本都存在，可以随时切换。

​	blue：老版本v1版本

​	green：新版本v2版本

```
vi blue.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue
  namespace: prod
  labels:
    app: blue
spec:
  replicas: 10
  selector:
    matchLabels:
      app: blue
  template:
    metadata:
      labels:
        app: blue
    spec:
      containers:
      - name: web
        image: registry.cn-chengdu.aliyuncs.com/mr5/y2305:v01
        imagePullPolicy: IfNotPresent
---
apiVersion: v1
kind: Service
metadata:
  name: blue
  namespace: prod
  labels:
    app: blue
spec:
  selector:
    app: blue
  ports:
  - port: 80
    targetPort: 80
    name: http
[root@www ingress]# kubectl apply -f blue.yaml

新版本：%s@blue@green@ig
vi green.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: green
  namespace: prod
  labels:
    app: green
spec:
  replicas: 10
  selector:
    matchLabels:
      app: green
  template:
    metadata:
      labels:
        app: green
    spec:
      containers:
      - name: web
        image: registry.cn-chengdu.aliyuncs.com/mr5/y2305:v02
        imagePullPolicy: IfNotPresent
---   
apiVersion: v1 
kind: Service
metadata:
  name: green
  namespace: prod
  labels:
    app: green
spec:
  selector:
    app: green
  ports:
  - port: 80
    targetPort: 80
    name: http

通过ingress来暴露：
vi ingress-blue.yaml  版本为v1
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: blue
  namespace: prod
spec:
  ingressClassName: nginx
  rules:
    - host: echo.mr5.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: blue
                port:
                  number: 80
[root@www ingress]# kubectl apply -f ingress-blue.yaml 
====》hosts中192.168.18.101 echo.mr5.com===出现v1页面

灰度发布：版本为v2
vi ingress-green.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: green
  namespace: prod
  基于流量：
  annotations:
    nginx.ingress.kubernetes.io/canary: 'true'==要开启灰度发布机制。首先要启用canary
    nginx.ingress.kubernetes.io/canary-weight: '30'==权重30%，分配30%
  
  或者基于头部：
  annotations:
    nginx.ingress.kubernetes.io/canary: 'true'
    nginx.ingress.kubernetes.io/canary-by-header-value: green==基于头部发布，user-value自己定义
    nginx.ingress.kubernetes.io/canary-by-header: canary
    nginx.ingress.kubernetes.io/canary-weight: '30'
    
或者基于cookie发布：
  annotations:
    nginx.ingress.kubernetes.io/canary: 'true' # 要开启灰度发布机制，收钱启用canary
    nginx.ingress.kubernetes.io/canary-by-cookie: 'users_from_beijing' #基于cookie
    nginx.ingress.kubernetes.io/canary-weight: '30' # 会被忽略，因为配置了canary
spec:
  ingressClassName: nginx
  rules:
    - host: echo.mr5.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: green
                port:
                  number: 80
vi /etc/hosts
192.168.18.101 echo.mr5.com
1、基于流量：当流量达到30%时就进行切割===就叫做灰度发布或者蓝绿发布
annotations:
    nginx.ingress.kubernetes.io/canary: 'true'==要开启灰度发布机制。首先要启用canary
    nginx.ingress.kubernetes.io/canary-weight: '30'==权重30%，分配30%
[root@www ingress]# for i in $(seq 1 100); do curl -s echo.mr5.com ; done | grep "v2" | wc -l
29



```

###### 1、灰度发布之基于权重（流量）发布：

```
基于权重（流量）：
  annotations:
    nginx.ingress.kubernetes.io/canary: 'true'==要开启灰度发布机制。首先要启用canary
    nginx.ingress.kubernetes.io/canary-weight: '30'==权重30%，分配30%
测试：
服务器： for i in $(seq 1 100); do curl -s echo.mr5.com ; done | grep "v2" | wc -l
```

###### 2、灰度发布之基于头部发布：

```
基于头部发布
annotations:
    nginx.ingress.kubernetes.io/canary: 'true'
    nginx.ingress.kubernetes.io/canary-by-header-value: green==基于user-value自己定义的来切割
    nginx.ingress.kubernetes.io/canary-by-header: canary==当user-value等于canary这个值时就到新版本
    nginx.ingress.kubernetes.io/canary-weight: '30'

基于头部：
annotations:
    nginx.ingress.kubernetes.io/canary: 'true'
    nginx.ingress.kubernetes.io/canary-by-header: canary    
当request header 设置为never 或者always 时，请求将不会或一直被发送到canary 版本
for i in $(seq 1 10); do curl -s -H "cancry: never" echo.mr5.com ; done

一、没有带头部的值就按自己的发布，就按流量30%发布：
[root@www ingress]# for i in $(seq 1 10); do curl -s -H "canary: xxx" echo.mr5.com; done==会交叉出现v1和v2

二、如果带头布user-value=green时====一直出现green版本
[root@www ingress]# for i in $(seq 1 10); do curl -s -H "canary: green" echo.mr5.com; done

```

###### 3、灰度发布之基于cookie发布：

```
3、基于cookie发布：也分never和always
 annotations:
    nginx.ingress.kubernetes.io/canary: 'true' # 要开启灰度发布机制，收钱启用canary
    nginx.ingress.kubernetes.io/canary-by-cookie: 'users_from_beijing' #基于cookie
    nginx.ingress.kubernetes.io/canary-weight: '30' # 会被忽略，因为配置了canary
3.1[root@www ingress]# for i in $(seq 1 10); do curl -s -b "users_from_beijing=always|never" echo.mr5.com ; done===always发布新版本v2，never发布v1

3.2没有带cookie时按流量30%切割
[root@www ingress]# for i in $(seq 1 10); do curl -s  echo.mr5.com ; done
```

