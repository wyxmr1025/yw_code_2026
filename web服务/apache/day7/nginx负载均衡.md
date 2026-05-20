20（136）

```
vi /etc/nginx/nginx.d
添加后端
upstream  php {
     server 192.168.18.13:9000 ;
     server 192.168.18.200:9000;
[root@localhost ~]# vi /etc/nginx/conf.d/y2312.conf 
server {
 listen 80;
 index index.php;
  location / {===找文件
  try_files $uri $uri/index.php ;
 }
 location ~ \.php$ {
 fastcgi_pass   php;
 fastcgi_index  index.php;
 fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
 include        fastcgi_params;
    }
}

安装mariadb-server
yum install mariadb-server
mysql> create database wordpress;
mysql> grant all privileges on wordpress.* to 'wpuser'@'%' identified by '123456';
mysql> flush privileges;


```



13（51）

```
安装php-fpm
开启服务
代理到后端：
vi /scriptts/index.php
<?php
 echo "this node1" 
?>
解压wordpress
将wordpress所有文件放入/scripts/
找文件：

测试是否可以连接mysql：
mysql -uwpuser -h192.168.66.136 -p123456
mysql> show databases;

修改权限：
chown -R apache:apache /scripts

解决数据不一致：采用挂载方式解决
yum install nfs-utils -y 
vi /etc/exports
/scripts 192.168.66.0/24(rw,no_all_squash)
systemctl start rpcbind
systemctl start nfs

此时会话不一致，当调度器调度在51这台服务器时，有数据（但session保存在浏览器中），当调度在52时没有session，就会出现会话不一致，解决：采用挂载，将session挂载，数据一致。

```



200（52）

```
安装php-fpm
开启服务
代理到后端：
<?php
 echo "this node2"
?>
解压wordpress
将wordpress所有文件放入/scripts/

修改权限：
chown -R apache:apache /scripts

rm -rf /scripts
yum install nfs-utils
mount -t nfs 192.168.66.51:/scripts /scripts
```

浏览器中：

```
192.168.66.136===>出现wordpress页面
用户:wpuser
密码：123456
数据库：192.168.66.136

安装blog：
此时会话不一致，当调度器调度在51这台服务器时，有数据（但session保存在浏览器中），当调度在52时没有session，就会出现会话不一致，解决：采用挂载，将session挂载，数据一致。
51服务器：session保存在/var/lib/php/session文件中，当调度器调度在51保存位置，查看52没有这个文件夹，为保证数据一致，采用nfs将数据进行挂载。
51：vi /etc/exports
...
/var/lib/php/session 192.168.66.0/24(rw,no_root_squash)
systemctl restart nfs

52： rm -rf /var/lib/php/session
mount -t nfs 192.168.66.51:/var/lib/php/session /var/lib/php/session


```

