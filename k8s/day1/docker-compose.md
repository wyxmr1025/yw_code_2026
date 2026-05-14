docker-compose	容器编排 单机编排

kuberenets ======》容器编排 集群

lamp：一个容器 跑多个进程

​			多个容器 共享一个网络空间

​			2个容器

docker-compose

```
vi Dockerfile
FROM centos:7
RUN yum install httpd php php-mysqlnd -y
CMD httpd -DFOREGROUND
[root@www y2312lap]# docker build . -t y2312lap:v1.0.1


[root@www ~]# mkdir y2312wordpress-compose
[root@www ~]# cd y2312wordpress-compose/
vi docker-compose.yml   ===docker官网 doc文档里面compose
version: "3"
services:
  web:
    images: y232lap:v1.0.1
    ports:
    - 8090:80
    volumes:
    - /webroot/:/var/www/html/
    depends_on:
    - db
  db:
    images: mysql:5.5.62
    command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    environment:
     MYSQL_ROOT_PASSWORD: Aa@123456
     MYSQL_DATABASE: wordpress
     MYSQL_USER: wpuser
     MYSQL_PASSWORD: Aa@123456

[root@www ~]# cp -r y2312wordpress/wordpress /webroot/
[root@www y2312wordpress-compose]# ln -s docker-compose-plugin-2.24.7-1.el7.x86_64 /usr/bin/
docker-compose up
chmod 777 y2312wordpress/wordpress
浏览器中输入： 192.168.18.13:8090/wordpress===输入：wpuser 密码：Aa@123456数据库:db(填写docker-compose文件中的名字)出现wordpress页面  

```

