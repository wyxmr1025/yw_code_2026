## web服务器有httpd、nginx

这里介绍另外一个web服务器： nginx

```
nginx： 解决高并发的（事件驱动异步非阻塞IO模型）
    用途：扮演web服务器
	     反向代理（7层调度器）
	工作模型：master进程：监听端口 加载配置文件
			worker进程：处理用户请求
	配置模型：
		全局配置（核心）：设定工作特性，例：要启动多少个进程啊、如何接受用户的请求的
		http配置
		邮箱配置	
```



安装nginx

```
nginx官网查找nginx====红毛系统====安装仓库
1、仓库：	
1、[root@www ~]# vi /etc/yum.repos.d/nginx.repo
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

2、更新仓库：[root@www ~]# yum repolist
3、安装nginx
[root@www ~]# yum install nginx
查看生成的文件
[root@www ~]# rpm -ql nginx


```



核心配置-全局模型：

```
vi /etc/nginx/nginx.conf
user  nginx;用户
worker_processes  auto|2...;   ======定义worker进程的数量,跑几个进程
                  内存：几核

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid; 


events {
    worker_connections  1024;  ==
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;  ====保持连接，无状态web服务器，一次只能请求一个资源，这个多了，处理并发就会少。

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}

```

### nginx全局配置：1-5

#### 1、**worker_cpu_affinity**： cpu亲和性，用来绑定进程到那些cpu上面

```
Syntax:	worker_cpu_affinity cpumask ...;
worker_cpu_affinity auto [cpumask];
Default:	—
Context:	main
For example,

worker_processes    4;   四核cpu
worker_cpu_affinity 0001 0010 0100 1000;  ====四核， 00 01 两核
有四核 cpu的掩码0001代表绑定在第一个cpu上。。。。===0101在第一核和第三核上面绑定
```

#### **2、 worker_connections** ：同时处理多个用户请求，指的是一个worker进程可以处理多少个用户的请求

```
Syntax:	worker_connections number;
Default:	
worker_connections 512;
Context:	events==写在events里面
simultaneous同时 并发

[root@www ~]# echo "xxx" > /usr/share/nginx/html/index.html
[root@www ~]# systemctl start nginx
[root@www ~]# curl 192.168.18.11
进行压力测试
[root@www ~]# ab -n 10000 -c 1000 http://127.0.0.1/
总请求是10000，并发是1000
Time per request:       116.369 [ms] (mean)
Time per request:       0.116 [ms] (mean, across all concurrent requests)
Transfer rate:          7116.38 [Kbytes/sec] received
将配置文件改成50000个请求（改变系统打开文件的限制）
文件限制数改成100000个
[root@www ~]# ulimit -n 10000   linux系统设定打开文件的数量是1024，只针对当前会话生效

```

>ulimit -n 只针对当前会话生效，要修改就要修改linux的配置文件。

#### **3、worker_rlimit_nofile**：打开文件的限制，代表每个worker能打开文件的数量

```
Syntax:	worker_rlimit_nofile number;
Default:	—
Context:	main
```

#### **4、error_log 日志**  级别 `debug`, `info`, `notice`, `warn`, `error`, `crit`, `alert` 

```
Syntax:	error_log file [level];
Default:	
error_log logs/error.log error;
Context:	main, http, mail, stream, server, location

```

#### 5、pid：判断nginx启动与否

```
pid /run/nginx.pid;
当nginx启动后，查看/run/nginx.pid，会保存nginx的进程号
当nginx关闭后，查看/run/nginx.pid，进程号就没有了。
```

### httpd模型

#### **1、http**：写在main

```
Syntax:	http { ... }
Default:	—
Context:	main
```

#### **2、access_log**

```
Syntax:	access_log path [format [buffer=size] [gzip[=level]] [flush=time] [if=condition]];
access_log off;
Default:	
access_log logs/access.log combined;
Context:	http, server, location, if in location, limit_except
```

#### **3、log_format**

```
Syntax:	log_format name [escape=default|json|none] string ...;
Default:	
log_format combined "...";
Context:	http
网站卡怎么办： 代码 带宽 服务器性能问题
```

#### **4、server**===include /etc/nginx/conf.d/default.conf===设定的虚拟主机站点

```
Syntax:	server { ... }
Default:	—
Context:	http
cp /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.bak

vi /etc/nginx/conf.d/y2312.conf
server {
  listen 80;
  server_name www.y2312.com;
  root /y2312; # 设定站点根目录。
  ...
}
```

#### **5、location**：匹配用户请求的uri，根据匹配到的uri来设定相应的配置

```
location = / {
    [ configuration A ]===精确匹配，只有/
}

location / {
    [ configuration B ]==模糊匹配，带/
}

location /documents/ {====模糊匹配
    [ configuration C ]
}

location ^~ /images/ {===左前缀匹配
    [ configuration D ]
}

location ~* \.(gif|jpg|jpeg)$ {===正则匹配，不区分大小
            转义：以.(gif|jpg|jpeg)结尾
    [ configuration E ]
}
请求http://www.y2312.com/  ==就是A
the “/” request will match configuration A, 
the “/index.html” request will match configuration B, 
the “/documents/document.html” request will match configuration C,
the “/images/1.gif” request will match configuration D, 
and the “/documents/1.jpg” request will match configuration E.
优先级： 精确> 左前缀> 正则>模糊

```

```
Syntax:	location [ = | ~ | ~* | ^~ ] uri { ... }
location @name { ... }
Default:	—
Context:	server, location


[root@www ~]# vi /etc/nginx/conf.d/y2312.conf
server {
 listen 80;
 server_name www.y2312.com;
 root /y2312;
 location / {
  root /y2312root/ ;
 }
 location ~* \.(jpeg|png|gif)$ {
  root /y2312img/ ;
 }
}
[root@www ~]# mkdir /y2312/img # 里面保存
[root@www ~]# mkdir /y2312/root
[root@www ~]# echo "this is y2312 root" > /y2312/root/index.html
测试：当访问www.y2312.com出现this is root (匹配根)
     当访问www.y2312.com/li.jpeg时出现照片（匹配到/y2312/img中的文件）
```

#### **6、try_files找文件**，找不到就去另外一个目录找

```
Syntax:	try_files file ... uri;
try_files file ... =code;
Default:	—
Context:	server, location

[root@www y2312img]# vi /etc/nginx/conf.d/y2312.conf
server {
 listen 80;
 server_name www.y2312.com;
 root /y2312/;
location /images/ {  ==本来没有/images/这个目录，就会在设定的目录/y2312/找
    try_files $uri /images/li.jpeg;
  }
}
[root@www y2312img]# mkdir /y2312/images ; cp /y2312img/li.jpeg /y2312/images
vi /y2312/images/index.html <h1>this is try_files</h1>

然后再浏览器中输入： 192.168.32.110/images/index.html会出现this is try_files
				 192.168.32.110/images/会出现图片
                 192.168.32.110/images/xxxx等就会用li.jpeg响应。
```

>try-files如果能找到对应的数据就在$uri里面的资源进行响应，如果没有找到就用/images/li.jpeg这个资源来响应请求。

![](nginx_pic/image-20241107161232550.png)



![](nginx_pic/image-20241107161316381.png)

![](nginx_pic/image-20241107161355403.png)



![](nginx_pic/image-20241107161421373.png)



![](nginx_pic/image-20241107161445305.png)

```
http {
    include       /etc/nginx/mime.types; 文件类型，根据文件后缀名
    default_type  application/octet-stream;文件数据流

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                       客服端地址                                     请求行
                      '$status $body_bytes_sent "$http_referer" '
                      状态码    发送的响应体   请求的头部
                      '"$http_user_agent" "$http_x_forwarded_for"';
                      
    access_log  /var/log/nginx/access.log  main;
      日志
    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;保持连接，无状态，一次请求保持时长

    #gzip  on;

    include /etc/nginx/conf.d/*.conf; 虚拟主机（web站点）
}
~                      
```

```
[root@www ~]# mv /etc/nginx/conf.d/default.conf  /etc/nginx//conf.d/default.conf.bak复制到另外一处，写自己的

[root@www ~]# vi /etc/nginx/conf.d/y2312.conf
server {
 listen 80;
 server_name www.y2312.com;
 root /y2312;
}

[root@www ~]# mkdir /y2312
mkdir: 无法创建目录"/y2312": 文件已存在
[root@www ~]# echo "this y2312 nginx site" > /y2312/index.html
[root@www ~]# nginx -t
nginx: [emerg] unknown directive "sever" in /etc/nginx/conf.d/y2312.conf:1
nginx: configuration file /etc/nginx/nginx.conf test failed
[root@www ~]# vi /etc/nginx/conf.d/y2312.conf
[root@www ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@www ~]# systemctl start nginx


```

#### 7、认证：ngx_http_auth_basic_module

```
例：
location / {
    auth_basic           "closed site";
    auth_basic_user_file conf/htpasswd;
}

location /admin {
    auth_basic           "please input your name"; 填写用户名和密码才能登陆
    auth_basic_user_file /etc/nginx/.userfile;==保存密码的地址
}

mkdir /y2312/admin
yum install httpd-tools
htpasswd -c -m /etc/nginx/.userfile dage(用户名)===输入密码
echo "this is admin" > /y2312/admin/index.html
浏览器中输入：192.168.32.110/admin弹出认证框
```

#### 8、ngx_http_autoindex_module下载东西有时候需要把索引下载下来

```
location /download/ {
    autoindex on;
}
mkdir /y2312/download
cp /etc/passed /y2312/download/
cp /etc/fstab /y2312/download/
在浏览器输入192.168.32.110/download会出现错误的

```



![](nginx_pic/image-20241107164320172.png)

#### 9、alias别名 

```
location /mp3/ {  ===没有mp3这个路径，只是匹配下面的/y2312/images/里面
    alias /y2312/images/;
}
系统中并没有创建/y2312/mp3这个文件夹，会去/y2312/images下找资源。
浏览器中访问：192.168.32.110/mp3/

```



![](nginx_pic/image-20241107165122025.png)

![Snipaste_2026-04-11_16-44-16](nginx_pic/Snipaste_2026-04-11_16-44-16.png)

#### **10、ngx_http_rewrite_module重写（伪静态）**：访问的是index.html但重写为php====rewrite

```
重写：break last写在server中不再配备rewrite，但是要匹配locaton
	 break last写在location 中break直接访问，last匹配locaton

Syntax:	rewrite regex replacement [flag];
Default:	—
Context:	server, location, if
regex正则，匹配要重写的内容；replacement将重写的内容替换成要写的东西
[flag]标志位
location /mp3/ {
  rewrite (.*)\.html$ $1.jpeg; 将/images/mp3/下以html结尾的文件重写为以jpeg结尾的文件
 }
mkdir /y2312/mp3/index.html 
echo "this is mp3" > /y2312/mp3/index.html
cp /y2312/images/li.jpeg /y2312/mp3/index.jpeg  ====注意这里改为index.jpeg
浏览器访问：192.168.32.110/mp3/index.html 会出现index.jpeg图片
```



![](nginx_pic/image-20241107170845713.png)



![](nginx_pic/image-20241107171321230.png)



![](nginx_pic/image-20241107171437736.png)



![](nginx_pic/image-20241107172145419.png)



![](nginx_pic/image-20241107172334634.png)
