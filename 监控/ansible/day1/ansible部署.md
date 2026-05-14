## 远程部署工具： 用于传统自动部署

①ansible :  （不需要安装客服端）使用ssh协议

②puppet  

③saltstack

>
>
>ansible帮助手册：ansible-doc ping

```
ping ：远程执行命令
command：不能识别特殊符号
shell：可以识别特殊符号
1、file模块：
	mode：
	src： 源文件
	path：指定目标文件
	state（指定目标文件状态）：absent,  directory,  file,  hard,  link,  touch
						 删除文件，目录 ，判断文件是否存在  ，硬链接，  软链接 ，touch文件
```

#### 1、file模块

```
13：
[root@www ~]# yum install epel-release
[root@www ~]# yum install ansible
[root@www ~]# vi /etc/ansible/ansible.cfg ==配置文件
inventory=  ===存放配置清单
vi /etc/ansible/hosts======定义被控制管理的的主机或服务器（主机清单）
[webservers]===提供webserver，可以自己取名
192.168.18.11

主机间需要免密：
ssh-keygen -t rsa(所有服务器都要安装ssh)
ssh-copy-id 192.168.32.101|130|140
查看ping的参数：
[root@www ~]# ansible-doc ping ===测试主机是否能够正常联通
[root@www ~]# ansible webservers -m "ping"
====针对webserver -m：模块 执行ping的指令

[root@www ~]# ansible webservers -a "echo hello world"==远程登录11服务器并输出hello world
192.168.18.11 | CHANGED | rc=0 >>
hello world

[root@www ~]# ansible webservers -m command -a "echo 'hello' > /tmp/ansible"
==执行成功但在11服务器cat /tmp/ansible时看不到hello world 所以不能识别特殊符号

[root@www ~]# ansible webservers -m shell -a "echo 'hello' > /tmp/ansible"
==执行成功，在服务器可以看到---shell模块可以识别特殊符号

[root@www ~]# ansible webservers -m file -a "path=/tmp/ansible"
==判断文件是否存在
[root@www ~]# ansible webservers -m file -a "path=/tmp/xxxxx"==xxxx不存在会报错

[root@www ~]# ansible webservers -m file -a "path=/tmp/xxxxx state=touch"
====state=创建xxxxx文件

[root@www ~]# ansible webservers -m file -a "path=/tmp/xxx state=link src=/etc/passwd"===创建xxx链接文件 指向/etc/passwd 

```

####  2、copy复制文件模块：ansible-doc copy查看copy的操作命令

```
13：
vi nginx.conf
server{
 listen 80;
 server_name www.y2312.com ;
 location / {
    root /webroot/ ;
 }
}
dest=复制到哪里去
src=源文件放置地点
[root@www ~]# ansible webservers -m copy -a "src=/root/nginx.conf dest=/etc"
11： ls /etc/nginx.conf   =====可以看到复制过来了
```

#### 3、yum安装模块：

```
[root@www ~]# ansible webservers -m yum -a "name=httpd state=absent"==卸载httpd
[root@www ~]# ansible webservers -m yum -a "name=httpd state=present"==安装

启动：
[root@www ~]# ansible webservers -m service -a "name=httpd state=started"


```





```
11：
[root@www ~]# ll /tmp/xxx
lrwxrwxrwx 1 root root 11 3月   4 15:33 /tmp/xxx -> /etc/passwd

```

