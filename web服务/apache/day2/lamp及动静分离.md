	https 及 动静分离实现 
	dns服务 
	防火墙 
	lvs 负载均衡

#### 1、自定义错误页面

```
1、[root@localhost ~]# vi /etc/httpd/conf/httpd.conf 
#ErrorDocument 500 "The server made a boo boo."
ErrorDocument 404 /404.html==《修改
3、浏览器中会出现
Not Found
IncludeOptional conf.d/*.conf
vi /webroot/404.html
<h1> this is 404 site </h1>

```

#### 2、虚拟主机：一台服务器可以提供多个站点web服务，一个站点就是一个web服务，可以根据不同ip地址，不同端口，不同hostname

中心主机：一台服务器只能提供一个站点web服务 需要虚拟主机是将documentroot注释掉

```
# symbolic links and aliases may be used to point to other locations.
#
1、# DocumentRoot "/webroot/"======《注释掉并将虚拟主机写在conf.d/*.conf

2、[root@localhost ~]# vi /etc/httpd/conf.d/virtual.conf===》新建虚拟主机并以.conf结尾

<VirtualHost 192.168.18.11:80>
DocumentRoot "/weba/"      #设置虚拟主机这个站点的根目录
<Directory "/weba/">       #设置虚拟主机的中心权限
 Require all granted       #允许所有能访问
</Directory>
</VirtualHost>
3、[root@localhost ~]# mkdir /weba/
4、[root@localhost ~]# echo "this is weba" > /weba/index.html
5、在浏览器中输入192.168.66.120出现
this is weba

第二个虚拟主机
1、添加多个ip
[root@localhost ~]# ip addr add 192.168.18.201/24 dev ens33
在dos命令中平一哈是否通过
2、添加第二个虚拟主机
<VirtualHost 192.168.18.201:80>
DocumentRoot "/webb/"
<Directory "/webb/">
 Require all granted
</Directory>
</VirtualHost>
3、[root@localhost ~]# mkdir /webb/
[root@localhost ~]# echo "this is webb" > /webb/index.html
[root@localhost ~]# systemctl restart httpd
4、在浏览器中输入192.168.18.201：80 ----会出现this is webb

基于端口的虚拟机
1、[root@localhost ~]# vi /etc/httpd/conf.d/virtual.conf
<VirtualHost 192.168.18.11:8080>
DocumentRoot "/webc/"
<Directory "/webc/">
 Require all granted
</Directory>
</VirtualHost>
2、[root@localhost ~]# mkdir /webc/
3、[root@localhost ~]# echo "this is webc" > /webc/index.html
4、在浏览器中输入192.168.18.11：8080会出现this is webc

基于域名
1、[root@localhost ~]# vi /etc/httpd/conf.d/virtual.conf
<VirtualHost 192.168.18.11:80>
2、ServerName www.y2312.com  =====《域名
DocumentRoot "/weba/"
<Directory "/weba/">
 Require all granted
</Directory>
</VirtualHost>
3、<VirtualHost 192.168.18.11:80>
ServerName www.y2310.com====《域名
DocumentRoot "/webd/"
<Directory "/webd/">
 Require all granted
</Directory>
</VirtualHost>
4、[root@localhost ~]# mkdir /webd/
5、[root@localhost ~]# echo "this is webd" > /webd/index.html 
6、[root@localhost ~]# systemctl restart httpd
7、在windos下面--system32---drivers---etc---hosts写上
192.168.18.11 www.y2312.com
192.168.18.11 www.y2310.com
浏览器中测试：www.y2312.com  ===>出现weba
www.y2310.com ====> 出现webd


需要对话框
1、[root@localhost ~]# vi /etc/httpd/conf.d/virtual.conf
<VirtualHost 192.168.18.11:8080>
DocumentRoot "/webc/"
<Directory "/webc/">
2 AuthType Basic======》修改
 AuthName "please input your name"
 AuthUserFile /etc/httpd/.httpuser
 Require valid-user====》
</Directory>
</VirtualHost>
3、[root@localhost ~]# systemctl restart httpd
4、在浏览器中会弹出密码框等：浏览器中输入192.168.18.11:8080==>出现用户名和密码框

基于日志
1、[root@localhost ~]# vi /etc/httpd/conf.d/virtual.conf
<VirtualHost 192.168.18.11:80>
ServerName www.y2310.com
DocumentRoot "/webd/"
<Directory "/webd/">
 Require all granted
</Directory>
2、Customlog "/var/log/httpd/webd.log" combined
</VirtualHost>
3、[root@localhost ~]# tail -f /var/log/httpd/webd.log
浏览器中输入：www.y2310.com ====>查看日志信息tail -f /var/log/httpd/webd.log
```



#### 解析php页面（动态页面）

```
1、[root@localhost ~]# vi /webd/index.php
<?php
 $name="zhangsan" ;
?>
<html>
<head>
</head>
<body>
<h1> my name is <?php echo $name ; ?> </h1>
</bady>
</html>
3、在浏览器中输入www.y2310.com/index.php不能解析php
4、
```



#### lamp 动静分离 主要配置  (lamp在同一个服务器中)

```
yum install php-fpm 
httpd+php在同一台服务器上面 此时php作为httpd的一个模块集成到httpd上了
lamp
1、[root@localhost ~]# yum install php
从其httpd服务php就解析出来了
浏览器中输入:www.y2310.com/index.php===出现my name is zhangshan

2、[root@localhost ~]# yum install mariadb-server
3、[root@localhost ~]# netstat -taunp | grep 3306
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      10671/mysqld     
4、[root@localhost ~]# mysql ===》进入mysql服务端
Welcome to the MariaDB monitor.  Commands end with ; or \g.
5、MariaDB [(none)]> show databases ; 查看数据库
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
6、MariaDB [(none)]> use y2312 ;
Database changed
7、MariaDB [y2312]> show tables ;
Empty set (0.00 sec)

8、MariaDB [y2312]> create table student (id int primary key , name varchar(10)) ;
Query OK, 0 rows affected (0.02 sec)

9、MariaDB [y2312]> desc student ;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | NO   | PRI | NULL    |       |
| name  | varchar(10) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
2 rows in set (0.01 sec)=====》\q 退出mysql

```



#### 基于php与数据库服务连接(基于lamp应用部署到平台上面)

```
1、[root@localhost ~]# mysql -uroot -hlocalhost -p 用户：root 密码：空
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
编写一个php连接数据库的测试代码：
2、[root@localhost ~]# vi /webd/test.php
<?php
servername = "localhost"; // MySQL服务器地址
$username = "root";       // MySQL用户名
$password = "";           // MySQL密码（根据需要设置）
$dbname = "y2312";         // 数据库名称

// 创建与MySQL服务器的连接
$conn = new mysqli($servername, $username, $password, $dbname);

// 检查连接是否成功
if ($conn->connect_error) {
    die("连接失败: " . $conn->connect_error);
} else {
    echo "连接成功！";
}
?>
php要和mysql连接就要遵循mysql的协议，所以php安装mysql客户端
3、[root@localhost ~]# yum install php-mysqlnd
4、[root@localhost ~]# systemctl restart httpd
5、浏览器中输入www.y2310.com/test.php===出现“连接成功”

```

#### 安装wordpress

```
1、[root@localhost ~]# unzip wordpress-6.4.2.zip 
2、[root@localhost ~]# mv wordpress /webd/ 将这个wordpress移动到/webd/下面
3、在浏览器中输入www.y2310.com/wordpress/===》有wordpress的配置文件===》显示没有权限

4、[root@localhost ~]# ls -ld /webd/wordpress/ 查看所属者所属组
drwxr-xr-x 5 apache apache 4096 12月  7 00:25 /webd/wordpress/
在浏览器上写上http://www.y2310.com/wordpress/弹出wordpress页面===》没有权限
5、[root@localhost ~]# chown -R apache:apache /webd/wordpress/
==》点击重新安装：admin Aa@123456 admin@y2312.com....
所以现在既能解析动态页面又能解析静态资源
```



#### 动静分离（开启另外一个服务器php服务器120）

##### 1、**php服务器：**

```
在另外开一个php服务器和apache服务器分开    120===扮演php服务器
1、[root@localhost ~]# yum install php-fpm
2、[root@localhost ~]# systemctl start php-fpm
3、[root@localhost ~]# netstat -taunp | grep 9000php默认监听在9000端口
4、[root@localhost ~]# vi /etc/php-fpm.d/www.conf 
listen = 0.0.0.0:9000===》改成所有服务器都能访问

; Set listen(2) backlog. A value of '-1' means unlimited.
; Default Value: -1
;listen.backlog = -1
 
; List of ipv4 addresses of FastCGI clients which are allowed to connect.
; Equivalent to the FCGI_WEB_SERVER_ADDRS environment variable in the original
; PHP FCGI (5.2.2+). Makes sense only with a tcp listening socket. Each address
; must be separated by a comma. If this value is left blank, connections will be
; accepted from any ip address.
; Default Value: any
; listen.allowed_clients = 127.0.0.1===》注销（默认只允许本机访问，先注销）
5、[root@localhost ~]# systemctl restart php-fpm
资源在php服务器上，当用户要访问php资源时，就要先访问apache服务器，apache服务器就转发给php服务器进行处理，并把处理完的结果交给apache服务器，apache服务器再返还给客户端（用户）；apache服务器如何判断用户请求的是php资源还是其他静态资源呢？是根据请求的后缀名来判断的；这个就叫做反向代理---反向代理的就是客户端
```

##### 2、**httpd主机**（110 code），删除之前的配置

>删除之前部署的wordpress、php等服务

```
1、[root@localhost ~]# yum remove php php-mysqlnd -y
2、[root@www ~]# rm -rf /etc/httpd/conf.d/virtual.conf
[root@www ~]# rm -rf /etc/httpd/conf.modules.d/0
[root@www ~]# rm -rf /etc/httpd/conf.modules.d/1
3、[root@localhost ~]# rm -rf /webd/wordpress/
4、[root@www ~]# systemctl restart httpd
 

```

##### 3、**php服务器**

```
1、[root@localhost ~]# mkdir /webdphp
[root@localhost ~]# cd /webdphp/
[root@localhost webdphp]# vi info.php
<?php
 phpinfo();
?>
```

##### 4、**httpd服务器配置代理：**

```
2、（====》18.110主机中（httpd服务器）===》正向、反向代理）
 [root@www ~]# vi /etc/httpd/conf/httpd.conf ---增加两种类型
    AddType application/x-httpd-php .php
    AddType application/x-httpd-source .phps

    #
    # AddHandler allows you to map ce
    
apache和php服务器遵循fast_cgi协议（通用网关协议）

3、 [root@www ~]# vi /etc/httpd/conf.d/virtual.conf
DocumentRoot "/webd/"===》添加
ProxyRequests off   ---正向代理关掉             
ProxyPassMatch ^/(.*\.php)$ fcgi://192.168.66.120:9000/webphp/$1 ---使用正则表达式

4、[root@www ~]# systemctl restart httpd
5、在浏览器中输入http://www.y2310.com/info.php出现php的页面
6、[root@www ~]# scp /webd/index.php 192.168.18.13:/webphp/
7、[root@www ~]# scp /webd/test.php 192.168.18.13:/webphp/
此时动静分离

8、grant all privileges on *.* to 'root'@'%' identified by 'Aa@123456';
flush privilges ;----110服务器（apache）
[root@www ~]# scp wordpress-3.3.1-zh_CN.tar.gz 192.168.18.13:/root
[root@www ~]# mkdir /webd/wordpress
```

##### 5、**php服务器中**:将wordpress应用部署到php服务器上

```
[root@localhost webdphp]# yum install php-mysqlnd
[root@localhost webdphp]# systemctl restart php-fpm
[root@localhost webdphp]# vi /webphp/test.php

<?php
$servername = "192.168.18.11"; // MySQL服务器地址(mysql与httpd在同一台服务器)
$username = "root";       // MySQL用户名
$password = "Aa@123456";           // MySQL密码（根据需要设置）
$dbname = "y2312";         // 数据库名称

// 创建与MySQL服务器的连接
$conn = new mysqli($servername, $username, $password, $dbname);

// 检查连接是否成功
if ($conn->connect_error) {
    die("连接失败: " . $conn->connect_error);
}

    echo "连接成功！";
$conn->close();
?>
在浏览器中输入www.y2310.com/test.php==》显示连接成功

[root@localhost ~]# tar -xf wordpress-3.3.1-zh_CN.tar.gz -C /webphp/
静态资源存放在apache服务器上，可以apache采用挂载形式来存放，首先在php服务器上：
vi /etc/exports
/webphp/wordpress 192.168.66.0/24(ro)
systemctl start rpcbind
systemctl start nfs
apache服务器上
mount -t nfs 192.168.66.120:/webphp/wordpress /webd/wordpress

浏览器上面输入会显示无法写入====此时为wordpress权限没有修改
php服务器上：chown -R apache:apache /webphp/wordpress
```



```
mysql-----与mysql server 基于某种协议

AddType application/x-httpd-php .php
AddType application/x-httpd-source .phps

proxyrequests off表示关闭正向代理；
				 /(info.php)
ProxyPassMatch ^/(.*\.php)$ fcgi://192.168.237.52:9000/webphp/$1
```

