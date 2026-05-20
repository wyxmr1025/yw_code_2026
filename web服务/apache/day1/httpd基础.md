### http服务---->web服务

```
文件服务  
		ftp服务
		nfs 

域名解析 
		dns --- bind
```

```
web服务  基于http 协议
http： 超文本传输协议
超文本：  文本  特殊标签 超文本标签  
http + mime
http: 
		tcp 80 
https： 
		tcp 443		

服务端 
		linux : 
			httpd|apache
			nginx
		windows 
			IIS

客户端
		浏览器
		curl
		
请求报文：==
请求行：  请求方法  url路径  http协议版本
请求头部：
	。。。。
请求主体  post方法
POST /admin.php/core-login HTTP/1.1   ====》请求行
Host: www.yucaizhongxue.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:121.0) Gecko/20100101 Firefox/121.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 50
Origin: http://www.yucaizhongxue.com
Connection: close
Referer: http://www.yucaizhongxue.com/admin.php
Cookie: haRo_P8SESSION=5d5a4737221f0d69
Upgrade-Insecure-Requests: 1

username=admin&password=admin&forward=%2Fadmin.php

```

请求方法： 

```
get:  url路径 请求url参数
post： 写在请求主体
put ： 修改
delete
patch 
head
url路径  
uri：   https://blog.csdn.net/mao_xiaoxi/article/details/89313734
		协议://主机名|IP地址[:port]url路径?username=zhangsan&password=123456

/index.html
```

​	响应报文  

```
响应行  http版本 状态码  解释
响应头
响应体	
```



```
[root@localhost ~]# systemctl start httpd
[root@localhost ~]# netstat -taunp | grep 80
tcp6       0      0 :::80                   :::*                    LISTEN      10395/httpd         
[root@localhost ~]# systemctl stop firewalld===关闭防火墙
[root@localhost ~]# setenforce 0
setenforce: SELinux is disabled
[root@localhost ~]# getenforce
Disabled
[root@localhost ~]# setenforce 0
setenforce: SELinux is disabled
[root@localhost ~]# echo "hello world" > /var/www/html/index.html==》在浏览器中输入：本机ip/index.html就会显示hello world
[root@localhost ~]# echo "hello world" > /var/www/html/test.html
[root@localhost ~]# 

```

状态码：

```
		2xx  成功
		3xx  重定向
			301  永久重定向
			302  临时重定向
			304  使用缓存
		4xx 客户端错误
			403： 权限|禁止访问
			404 : 资源未找到
		5xx 服务端错误
```


  	

	centos  yum install httpd 
	Ubuntu  apt-get install apache2

```
查看httpd的配置文件
[root@localhost ~]# rpm -ql httpd | less
/etc/httpd
/etc/httpd/conf
/etc/httpd/conf.d
/etc/httpd/conf.d/README
/etc/httpd/conf.d/autoindex.conf
/etc/httpd/conf.d/userdir.conf
/etc/httpd/conf.d/welcome.conf ===》欢迎页面
/etc/httpd/conf.modules.d
/etc/httpd/conf.modules.d/00-base.conf
/etc/httpd/conf.modules.d/00-dav.conf
/etc/httpd/conf.modules.d/00-lua.conf
/etc/httpd/conf.modules.d/00-mpm.conf
/etc/httpd/conf.modules.d/00-proxy.conf
/etc/httpd/conf.modules.d/00-systemd.conf
/etc/httpd/conf.modules.d/01-cgi.conf ===》httpd模块
/etc/httpd/conf/httpd.conf


```

删除配置文件（欢迎页面的文件）

```
[root@localhost ~]# rm -rf /etc/httpd/conf.d/* 删除欢迎的配置文件
[root@localhost ~]# systemctl start httpd 重启

```

主配置文件  

#### 	1、/etc/httpd/conf/httpd.conf ===》修改端口号8080

```
vi /etc/httpd/conf/httpd.conf
#Listen 12.34.56.78:80
Listen 8080
systemctl reload httpd===》并重启网络
```

#### 2、修改路径：

```
Include conf.modules.d/*.conf ===》相对于ServerRoot "/etc/httpd"
[root@localhost ~]# vi /etc/httpd/conf.modules.d/0
00-base.conf     00-dav.conf      00-lua.conf      00-mpm.conf      00-proxy.conf    00-systemd.conf  01-cgi.conf      

[root@localhost ~]# ps -aux | grep httpd ==》查看是谁在使用apache
root      11572  0.0  0.2 221960  4936 ?        Ss   16:38   0:00 /usr/sbin/httpd -DFOREGROUND
apache    11574  0.0  0.2 224044  3812 ?        S    16:38   0:00 /usr/sbin/httpd -DFOREGROUND
。。。

```

#### 3、DocumentRoot "/var/www/html"===》用来指定url根所对应本地文件系统路径（apache）

```
vi /etc/httpd/conf/httpd.conf
[root@localhost ~]# echo "this is y2312" > /var/www/html/index.html==》在浏览器输入ip/index.html会显示---this is y2312
路径也可以修改
[root@localhost ~]# mkdir /webroot
[root@localhost ~]# echo "hhh" > /webroot/index.html
[root@localhost ~]# vi /etc/httpd/conf/httpd.conf 
DocumentRoot "/webroot/"
[root@localhost ~]# systemctl restart httpd

在浏览器刷新会出现
Forbidden
You don't have permission to access /index.html on this server.

[root@localhost ~]# vi /etc/httpd/conf/httpd.conf 
# Further relax access to the default document root:
<Directory "/webroot/">===》修改
在浏览器上刷新会出现hhh  ===》hhh是index.html的内容

```

#### 4、访问限制

```
1、[root@localhost ~]# vi /etc/httpd/conf/httpd.conf 
   # Controls who can get stuff from this server.
    #
    2、Require all denied   ====>所有人都不能访问
</Directory>

#
# DirectoryIndex: sets the file that Apache will serve if a directory
# is requested.
#
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
3、[root@localhost ~]# systemctl restart httpd
4、在浏览器上刷新
Forbidden
You don't have permission to access /index.html on this server.

```

#### 5、指定只有某些ip可以访问

```
1、[root@localhost ~]# vi /etc/httpd/conf/httpd.conf 
  AllowOverride None

    #
    # Controls who can get stuff from this server.
    #
   2、 Require ip 192.168.18===》制定网段
检查语法错误：
3、[root@localhost ~]# httpd -t
Syntax OK
4、重启
[root@localhost ~]# systemctl restart httpd
5、浏览器会出现hhh
```

#### 6、下载

```
1、[root@localhost ~]# mkdir /webroot/download
2、[root@localhost ~]# cp /etc/fstab /webroot/download/
3、浏览器中输入=====> http://192.168.66.120:8080/download/
浏览器中会出现
Index of /download
Parent Directory
fstab
4、[root@localhost ~]# vi /etc/httpd/conf/httpd.conf 
   #
5、 Options FollowSymLinks=====》将Indexes去掉（不允许下载）
6、[root@localhost ~]# systemctl restart httpd====》重启
7、浏览器中输入ip/download/会出现
Forbidden
You don't have permission to access /download/ on this server.


```

#### 7、允许放|不放符号链接

```
Options FollowSymLinks 允许符号链接
1、[root@localhost ~]# ln -s /etc/passwd /webroot/download/y2312==》创建链接y2312
2、在浏览器中输入ip/download/y2312 ==>可以访问
3、[root@localhost ~]# vi /etc/httpd/conf/httpd.conf
 # http://httpd.apache.org/docs/2.4/mod/core.html#options
    # for more information.
    #
   3、 Options None  ===》将FollowSymLinks修改成none==》什么都不允许
4、[root@localhost ~]# systemctl restart httpd
5、浏览器中就会出现Forbidden
```

#### 8、允许覆盖

```
1、[root@localhost ~]# vi /etc/httpd/conf/httpd.conf
# AllowOverride controls what directives may be placed in2、==》 .htaccess files. ==》覆盖文件存放地址
    # It can be "All", "None", or any combination of the keywords:
    #   Options FileInfo AuthConfig Limit
    #
    AllowOverride All===》允许覆盖
3、[root@localhost ~]# vi /webroot/download/.htaccess===》在配置文件中允许覆盖
Options All===》允许覆盖
4、[root@localhost ~]# systemctl restart httpd
5、在浏览器中刷新会出现
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync。。。

```

#### 9、伪静态：将静态---动态显示

>实际访问的是/index.html（静态资源）--实际上将html转发给后端/index.php进行动态展示。

```
1、[root@localhost ~]# vi /webroot/index.php
<?php
        phpinfo();
?>
2、在浏览器中不能解析php
3、[root@localhost ~]# cp /webroot/index.php /webroot/download/
4、[root@localhost ~]# vi /webroot/download/.htaccess===》将php解析成为HTML，以便浏览器解析
RewriteEngine On
RewriteBase /
RewriteRule ^(.+)\.html $1.php [L,NC]
5、在浏览器中输入：192.168.18.10/download/index.html
<?php
	phpinfo();
?>
```

#### 10、认证密码

```
1、[root@localhost ~]# vi /etc/httpd/conf/httpd.conf 
<Directory "/webroot/download/">
AuthName "xxxx"
AuthUserfile "/etc/httpd/.httpuser"
AuthType Basic
Require valid-user
</Directory>
2、[root@localhost ~]# httpd -t=====》 验证语法是否正确
Syntax OK
3、在浏览器中刷新，弹出密码用户的对话框====》只在当前窗口输入用户和密码
4、[root@localhost ~]# htpasswd -c -m /etc/httpd/.httpuser jack 用户：===》jack
New password: ===》123456
Re-type new password: 
Adding password for user jack
5、[root@localhost ~]# cat /etc/httpd/.httpuser ===》查看密码
jack:$apr1$eBuTTXPj$JDyiw25U7uwJAv/2ZLI13.

6、不写在/etc/httpd/conf/httpd.conf，写在密码的配置文件中
[root@localhost ~]# vi /webroot/download/.htaccess
AuthName "xxxx"
AuthUserfile "/etc/httpd/.httpuser"
AuthType Basic
Require valid-user


```

#### 11、指定默认页面：浏览器中默认输入index.html或者index.php 日志等

```
<IfModule dir_module>
    DirectoryIndex index.html index.php===》加入index.php

错误日志
# LogLevel: Control the number of messages logged to the error_log.
# Possible values include: debug, info, notice, warn, error, crit,
# alert, emerg.
#
错误日志级别：LogLevel warn==》可以修改debug, info, notice, warn, error, crit,alert, emerg.

访问日志
<IfModule log_config_module>
    #
    # The following directives define some format nicknames for use with
    # a CustomLog directive (see below).
    #
    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
%h:ip地址 登录名 用户名 时间 “请求行” 状态码 响应的大小 “插入一个头部” “”
    LogFormat "%h %l %u %t \"%r\" %>s %b" common
例：192.168.18.1 - jack [15/Jan/2024:19:11:00 +0800] "GET /download/ HTTP/1.1" 200 336 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36"


```

#### 12、别名

```
 # Example:
    Alias /xxxx /webroot/alias   ===>修改，类似于映射
    #
    # If you include a trailing / on /webpath then the server will
[root@localhost ~]# mkdir /webroot/alias
[root@localhost ~]# echo "xxxx" > /webroot/alias/index.html
[root@localhost ~]# systemctl restart httpd
浏览器中输入:192.168.66.100/xxxx 出现xxxx这个页面
```

```
静态资源  html css js 图片 视频 静态资源

动态资源  php jsp asp

DocumentRoot  指定url的根所对应本地文件系统路径


<Directory  > 


</Directory>





cncf： 云原生

Listen  

User apache
Group apache

DocumentRoot

<Directory >
Options 
AllowOverride 
Require
</Direcrtory>

DirectoryIndex   

ErrorLog 

CustomLog

Alias
```

