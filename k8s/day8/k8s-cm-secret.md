# k8s configmap和secret

## 一、configmap

#### 1、configmap：定义环境变量，配置文件

```
100：
[root@www y2312mainfeast]# mkdir cm
[root@www y2312mainfeast]# cd cm/
vi cm-1.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-demo
  namespace: default
data:
  data.1: hello
  data.2: world
  config: |  保留每行的换行符
    property.1=value-1
    property.2=value-2
    property.3=value-3

通过传递环境变量：
vi pod-cm.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-cm1
  namespace: default
spec:
  containers:
  - name: test-cm
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    args: ["sleep","3600"]
1、 env:
    - valueFrom:   ===切片
        configMapKeyRef:
          name: cm-demo
          key: data.1===赋值hello
      name: data1
     
2、  env: 
     - name: CONFIG  ====也可以赋值config
        valueFrom:
          configMapKeyRef:
            name: cm-demo
            key: config《------


kubectl apply -f pod-cm.yaml
k9s中default ==pod-cm中shell进入输入：env====》data.1得值 hello
也可以赋值config

envFrom
configMapRef：configmap中的k值作为环境变量传下去
vi pod-cm2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-cm2
  namespace: default
spec:
  containers:
  - name: test-cm
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    args: ["sleep","3600"]
    envFrom:  =====envFrom
    - configMapRef:
        name: cm-demo===名称通过kubectl get cm得知
        key: data.1
```

>应用：部署mysql时可以将环境变量保存在configmap中
>用于挂载就相当于配置文件

#### 2、使用configmap部署msyql

```
env：
MYSQL_ROOT_PASSWORD
MYSQL_DATABASE
MYSQL_USER
MYSQL_PASSWORD

vi cm-2.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-mysql
  namespace: default
data:
  MYSQL_ROOT_PASSWORD: Aa@123456
  MYSQL_DATABASE: wordpress
  MYSQL_USER: wpuser
  MYSQL_PASSWORD: Aa@123456

vi pod-cm3.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-cm3
  namespace: default
spec:
  containers:
  - name: test-cm
    image: mysql:5.5.62
    imagePullPolicy: IfNotPresent
    envFrom:
    - configMapRef:
        name: cm-mysql
k9s中default===pod-cm3进入shell输入：mysql -p===Aa@123456


```

#### 3、将nginx中的镜像保存在配置文件中（configmap），通过挂载来使用

```
vi pod-cm4.yaml
apiVersion: v1
kind: Pod
metadata:
  name: cm4-pod
spec:
  volumes:
    - name: config-volume
      configMap:
        name: cm-demo
  containers:
    - name: testcm3
      image: busybox：latest
      imagePullPolicy： IfNotPresent
      command: [ "/bin/sh", "-c", "sleep 3600" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
k9s中default==cm4-pod===shell进入===cat /etc/config内容


通过items来挂载单个key
vi pod-cm5yaml
apiVersion: v1
kind: Pod
metadata:
  name: cm5-pod
spec:
  volumes:
    - name: config-volume
      configMap:
        name: cm-demo
        items:
        - key: config====挂载的目录（cm-1当中的目录）
          path: redis/conf（将次目录实际上挂载到/etc/config），这里不能写绝对路径。会报错例如：/redis/conf
  containers:
    - name: testcm4
      image: busybox:latest
      imagePullPolicy: IfNotPresent
      command: [ "/bin/sh", "-c", "sleep 3600" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
k9s-====cm5-pod进入shell：
/ # cat /etc/config/redis/conf 
property.1=value-1
property.2=value-2
property.3=value-3

```

configmap可以通过环境变量或者挂载来实现配置文件

#### 4、还可以通过命令来创建configmap

```
configmap NAME  
[--dry-run=server|client|none] [options]

[--from-file=[key=]source]===configmap来自于那个文件

1、vi nginx.conf
server {
 listen 80 ;
 server_name www.y2312.com ;
}
[root@www cm]# kubectl create cm nginx-cm --from-file=./nginx.conf
                                 指定名称                当前的nginx.conf
查看：
[root@www cm]# kubectl get cm
NAME               DATA   AGE
cm-demo            3      95m
cm-mysql           4      48m
kube-root-ca.crt   1      7d9h
nginx-cm           1      13s====新增
[root@www cm]# kubectl get cm nginx-cm -o yaml以yaml形式查看内容

将nginx.conf以挂载形式进去
vi pod-cm6.yaml
apiVersion: v1
kind: Pod
metadata:
  name: cm6-pod
spec:
  volumes:
    - name: config-volume
      configMap:
        name: nginx-cm
  containers:
    - name: testcm4
      image: busybox:latest
      imagePullPolicy: IfNotPresent
      command: [ "/bin/sh", "-c", "sleep 3600" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
k9s===pod-cm6==shell：
/ # cat /etc/config/nginx.conf 
server {
 listen 80 ;
 server_name www.y2312.com ;
}

2、[--from-literal=key1=value1]====自定义k和v
[root@www cm]# kubectl create cm erwa --from-literal=NAME=dage
                                 自定义名称(k)            自定义值（v）
```





## 二、secret

#### 1、作用：将configmap里面的内容进行编码，但是没有进行加密

```
secret：将configmap里面的内容进行编码，并没有加密

Opaque：base64 编码格式的 Secret，用来存储密码、密钥等；但数据也可以通过 base64 –decode 解码得到原始数据，所有加密性很弱。
```

#### 2、如何使用secret

​	1环境变量| 2挂载

```
[root@www cm]# echo  YWRtaW4= | base64 --decode
admin[root@www cm]# 
或者：
[root@www cm]# echo admin | base64
YWRtaW4K

[root@www y2312mainfeast]# mkdir secret
[root@www y2312mainfeast]# cd secret/
vi secret1.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque   ====类型
data:
  username: YWRtaW4=
  password: YWRtaW4zMjE=

如何使用secret：1环境变量| 2挂载
环境变量：
vi pod-secret.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret
  namespace: default
spec:
  containers:
  - name: test-cm
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    args: ["sleep", "3600"]
    envFrom:
    - secretRef:
      name: mysecret
k9s==default==pod-secret==shell进入===输入env得到明文显示

stringDate：直接写明文，看到的是编码过后的文档
vi secret2.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret1
type: Opaque
stringData:======》明文写进去，
  username: admin
  password: admin123


查看是否是加密过后的文档
[root@www secret]# kubectl get secret/mysecret1 -o yaml
例：mysql
vi cm-2.yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-mysql
  namespace: default
stringData:===明文写，结果是编码过后
  MYSQL_ROOT_PASSWORD: Aa@123456
  MYSQL_DATABASE: wordpress
  MYSQL_USER: wpuser
  MYSQL_PASSWORD: Aa@123456

vi pod-secret2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-mysql
  namespace: default
spec:
  containers:
  - name: test-cm
    image: mysql:5.5.62
    imagePullPolicy: IfNotPresent
    envFrom:
    - secretRef:
        name: secret-mysql
k9s中default===pod-mysql==shell进入===env==出现明文===可以使用mysql

挂载相当于配置文件使用secret
```

#### 3、通过命令创建secret

```
通过命令来写secret：
一、generic
kubectl create secret --help
1、docker-registry   创建一个给 Docker registry 使用的 Secret
2、generic：一般性的  Create a secret from a local file, directory, or literal value
3、tls               创建一个 TLS secret
--from-file：来自于那个文件
--from-literal：指定k和v
[root@www secret]# kubectl create secret generic dagesecret --from-file=./nginx.conf --from-literal=age=23
secret/dagesecret created
查看：
[root@www secret]# kubectl get secret dagesecret
NAME         TYPE     DATA   AGE
dagesecret   Opaque   2      94s

二、tls
secret保存证书和私钥：
[root@www secret]# kubectl create secret tls --help
--cert=path/to/cert/file：证书名称 
--key=path/to/key/file：私钥的位置

生成证书：openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=www.dage.com"
命令来创建，将证书和私钥保存在secret当中了：
[root@www secret]# kubectl create secret tls dagecert --cert=./tls.crt --key=./tls.key 
查看：[root@www secret]# kubectl get secret dagecert -o yaml

三、secret保存私有云的用户和密码：
保存dockerhub的用户名和密码：====》拉取私有云仓库此时需要密码和用户
仓库为私有，docker pull拉取不了镜像，必须要先登录
[root@www secret]# docker login registry.cn-chengdu.aliyuncs.com/mr5
Username: wulei6022@hotmail.com
password：Aa@123456
此时再拉去镜像才能成功：
[root@www secret]# docker pull registry.cn-chengdu.aliyuncs.com/mr5/y2312:v1
 
 --docker-username=user 
 --docker-password=password
 --docker-server：钥匙是给那台服务器的

aliyuncret：自己取名
[root@www secret]# kubectl create secret docker-registry aliyuncret --docker-username=wulei6022@hotmail.com --docker-password=Aa@123456 --docker-server=registry.cn-chengdu.aliyuncs.com==创建成功

vi pod-pull-test.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-pull
  namespace: default
spec:
  imagePullSecrets:=====》将aliyuncret里面保存的用户名和密码交给节点node1或者2
  - name: aliyuncret
  containers:
  - name: test-pull
    image: registry.cn-chengdu.aliyuncs.com/mr5/y2312:v1

[root@www secret]# kubectl create secret generic myregistry --from-file=.dockerconfigjson=/root/.docker/config.json -type=kubernetes.io/dockerconfigjson
```

