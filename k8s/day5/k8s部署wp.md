## k8s部署wordpress

前提条件：

```
wordpress   lamp容器mysql-------deployment

							集群外
```

#### 1、node1服务器：

>node1上使用dockerfile编写lamp平台，创建自建的image，写入到yaml文件中。

```
101:
vi Dockerfile
FROM centos:7   替代：quay.io/centos/centos:stream9
RUN yum install httpd php php-mysqlnd -y
ADD wordpress-3.3.1-zh_CN.tar.gz /var/www/html/
RUN chown apache:apache /var/www/html/wordpress
CMD httpd -DFOREGROUND

[root@www y2312wordpress]# docker login registry.cn-chengdu.aliyuncs.com
Username: wulei6022@hotmail.com  ===xyw5201
passwd：Aa@123456

打包镜像：
[root@www y2312wordpress]# docker tag . -t registry.cn-chengdu.aliyuncs.com/mr5/y2312wordpress:v1
[root@www y2312wordpress]# docker build . -t registry.cn-chengdu.aliyuncs.com/mr5/y2312wordpress:v1
[root@www y2312wordpress]# docker run --rm -p 8092:80 registry.cn-chengdu.aliyuncs.com/mr5/y2312wordpress:v1===》浏览器中输入：192.168.18.101：8092/wordpress
[root@www y2312wordpress]# docker push registry.cn-chengdu.aliyuncs.com/mr5/y2312wordpress:v1===推送

```

#### 2、node2服务器：

```
102：
systemctl start mariadb
mysql -uroot -p123456
create database wordpress;
grant all privileges on *.* to 'root'@'www.y2312node2.com' identified by '123456';
flush privileges;
```

#### 3、master服务器：

##### 3.1引入wordpress服务到k8s集群中：

```
100：
vi deploy-svc.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  namespace: prod
  labels:
    app: wordpress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - name: web
        image:  registry.cn-chengdu.aliyuncs.com/mr5/y2312wordpress:v1

---
apiVersion: v1
kind: Service
metadata:
  name: wordpress-svc
  namespace: prod
  labels:
    app: wordpress
spec:
  selector:
    app: wordpress
  ports:
  - name: web
    port: 80
    targetPort: 80
  type: NodePort
进入k9s==prod=svc===》找到端口号==30602

```

##### 3.2、引入外部服务mysql到集群内部：

```
将102的mysql引入集群===》
vi mysql-endpoints.yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: mysql
  namespace: prod
subsets:
- addresses: 
  - ip: 192.168.18.102
  ports:
  - name: mysql
    port: 3306

创建对应的svc
vi msyql-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: prod
spec:
  ports:
  - name: msyql
    port: 3306
    targetPort: 3306


查看端口--浏览器中输入 密码：123456 用户名：root 数据库主机： mysql
```

