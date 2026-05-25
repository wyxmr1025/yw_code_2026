# MangoDB非关系型数据库

![Snipaste_2026-04-09_17-39-43](images/Snipaste_2026-04-09_17-39-43.png)

运维架构：

![Snipaste_2026-04-09_17-41-23](images/Snipaste_2026-04-09_17-41-23.png)

![Snipaste_2026-04-09_17-45-24](images/Snipaste_2026-04-09_17-45-24.png)

![Snipaste_2025-09-24_17-55-55](images/Snipaste_2025-09-24_17-55-55.png)

![Snipaste_2025-09-25_15-39-33](images/Snipaste_2025-09-25_15-39-33.png)

![Snipaste_2025-09-25_15-42-02](images/Snipaste_2025-09-25_15-42-02.png)

![Snipaste_2025-09-25_15-42-59](images/Snipaste_2025-09-25_15-42-59.png)

![Snipaste_2025-09-26_10-58-32](images/Snipaste_2025-09-26_10-58-32.png)

>
>
>先暂停mongod服务：pkill mongod
>
>查看配置文件是否有问题： bin/mongod --config /etc/mongodb.conf

![Snipaste_2025-09-25_15-44-21](images/Snipaste_2025-09-25_15-44-21.png)

>![Snipaste_2025-09-25_15-47-45](images/Snipaste_2025-09-25_15-47-45.png)

安装mongod客户端：

![Snipaste_2025-09-25_15-50-24](images/Snipaste_2025-09-25_15-50-24.png)

>
>
>

![](images/Snipaste_2025-09-25_15-51-42.png)

![Snipaste_2025-09-25_15-53-59](images/Snipaste_2025-09-25_15-53-59.png)

![Snipaste_2025-09-25_15-55-06](images/Snipaste_2025-09-25_15-55-06.png)

![Snipaste_2025-09-25_15-56-05](images/Snipaste_2025-09-25_15-56-05.png)

![Snipaste_2025-09-25_15-57-07](images/Snipaste_2025-09-25_15-57-07.png)

![Snipaste_2025-09-25_15-58-15](images/Snipaste_2025-09-25_15-58-15.png)

![Snipaste_2025-09-25_16-01-02](images/Snipaste_2025-09-25_16-01-02.png)

![Snipaste_2025-09-25_16-01-53](images/Snipaste_2025-09-25_16-01-53.png)

![Snipaste_2025-09-25_16-03-16](images/Snipaste_2025-09-25_16-03-16.png)

![Snipaste_2025-09-25_16-07-50](images/Snipaste_2025-09-25_16-07-50.png)

![Snipaste_2025-09-25_16-11-51](images/Snipaste_2025-09-25_16-11-51.png)

![Snipaste_2025-09-25_16-12-52](images/Snipaste_2025-09-25_16-12-52.png)

![Snipaste_2025-09-25_16-15-19](images/Snipaste_2025-09-25_16-15-19.png)

![Snipaste_2025-09-25_16-16-39](images/Snipaste_2025-09-25_16-16-39.png)

![Snipaste_2025-09-25_16-17-42](images/Snipaste_2025-09-25_16-17-42.png)

![Snipaste_2025-09-25_16-25-50](images/Snipaste_2025-09-25_16-25-50.png)

![Snipaste_2025-09-25_16-28-36](images/Snipaste_2025-09-25_16-28-36.png)

![Snipaste_2025-09-25_16-30-01](images/Snipaste_2025-09-25_16-30-01.png)

![Snipaste_2025-09-25_16-32-07](images/Snipaste_2025-09-25_16-32-07.png)

![Snipaste_2025-09-25_16-33-11](images/Snipaste_2025-09-25_16-33-11.png)

![Snipaste_2025-09-25_16-44-59](images/Snipaste_2025-09-25_16-44-59.png)

>
>
>安装设置时先把安全设置关闭，再去设置账号和密码，设置完成后再去开始安全设置！！！

![Snipaste_2025-09-25_16-47-42](images/Snipaste_2025-09-25_16-47-42.png)

>
>
>登录：mongosh mongodb://192.168.66.6:27017/
>
>安装设置时先把安全设置关闭，再去设置账号和密码，设置完成后再去开始安全设置！！！

![](images/Snipaste_2025-09-25_16-48-12.png)

![Snipaste_2025-09-25_16-50-23](images/Snipaste_2025-09-25_16-50-23.png)

>注意：先在终端配置账号，再去修改/etc/mongodb.conf中安全配置！！！！
>
>设置完密码进入mongod服务时：1、mongosh mongodb://192.168.66.8:27017/admin
>
>2、终端：db.auth("admin","123456")
>
>或者直接在终端登录： mongosh mongodb://admin:123456@192.168.66.8:32017/
>
>建议使用：登陆后使用密码输入

```
删除用户：终端输入 db.dropUser("admin")
更新用户: db.updateUser("admin",{pwd:"123456"})
```



![Snipaste_2025-09-25_16-56-55](images/Snipaste_2025-09-25_16-56-55.png)

![Snipaste_2025-09-25_17-17-46](images/Snipaste_2025-09-25_17-17-46.png)

![Snipaste_2026-04-10_17-40-45](images/Snipaste_2026-04-10_17-40-45.png)

>总结：
>
>mongodb 不等同于数据全存内存
>
>mongodb和redis不同，他不是纯内存数据库，只是极度依赖内存做加速。真正的数据可靠性任靠磁盘存储和日志机制保障。

![Snipaste_2025-09-25_17-18-25](images/Snipaste_2025-09-25_17-18-25.png)

![Snipaste_2026-04-10_17-49-25](images/Snipaste_2026-04-10_17-49-25.png)

>mongodb客户端终端，如果想要进行用户验证，使用db.auth()验证账号和密码，有前提：必须要切换到admin数据库验证。否则会导致验证失败。

```
use admin
db.auth("admin",123456)
```



![Snipaste_2025-09-25_17-19-36](images/Snipaste_2025-09-25_17-19-36.png)

![Snipaste_2025-09-25_17-20-31](images/Snipaste_2025-09-25_17-20-31.png)

![Snipaste_2025-09-25_17-22-33](images/Snipaste_2025-09-25_17-22-33.png)

![Snipaste_2025-09-25_17-24-43](images/Snipaste_2025-09-25_17-24-43.png)

![Snipaste_2025-09-25_17-25-28](images/Snipaste_2025-09-25_17-25-28.png)

![Snipaste_2025-09-26_17-42-45](images/Snipaste_2025-09-26_17-42-45.png)

>fluentd将日志收集起来发给mongodb进行处理，安装插件fluent-plugin-mongo适合海量存储、收集非规则日志。

![Snipaste_2025-09-26_17-45-38](images/Snipaste_2025-09-26_17-45-38.png)

## 3、配置Fluentd

>编辑： /etc/td-agent/td-agent.conf

![Snipaste_2025-09-26_17-48-42](images/Snipaste_2025-09-26_17-48-42.png)

![Snipaste_2025-09-26_17-50-45](images/Snipaste_2025-09-26_17-50-45.png)





![Snipaste_2025-09-26_17-52-21](images/Snipaste_2025-09-26_17-52-21.png)





# 配置文件

![Snipaste_2025-09-26_17-54-51](images/Snipaste_2025-09-26_17-54-51.png)

```
实操：
1、安装fluent: yum install tg-agent.....rpm -y
2、安装fluent连接mongodb的插件： yum install fluent-plugin-mongo
3、安装nginx：yum install nginx 
	echo "web page" > /usr/share/nginx/html/index.html
	ps:访问会产生日志：/var/log/nginx/access.log， 之只要访问就会产生日志，以此来测试fluent来采集		mongodb数据
4、配置fluent：
	告诉fluent在哪里可以采集数据：/etc/td-agent/td-agent.conf
	
```

```
vi /etc/td-agent/td-agent.conf
<source>
  @type tail
  path /var/log/nginx/access.log  ===>定义收集的路径
  pos_file /var/log/td-agent/nginx.pos
  tag nginx.access
  <parse>
    @type nginx
  </parse>
</source>

<match nginx.access>
  @type stdout   ====输出的地方为：标准输出
</match>
5、重启td-agent： systemctl restart td-agent、systemctl enable td-agent
ps注意： 如果数据没有写入到终端，可能写入到了td-agent日志文件中：
cat /var/log/td-agent/td-agent.log
测试： 使用curl http://localhost或者浏览器访问nginx
6、将日志保存至mongodb中：修改配置文件添加mongodb的输出

```

```
vi /etc/td-agent/td-agent.conf
...
<match nginx.access>
  @type mongo
  database logs   ===>数据库
  collection nginx_access ====>mongodb中的表
  host 192.168.66.8  ====输出地址、用户名端口、密码
  port 27017
  user admin
  password 123456
  auth_source admin
  flush_interval 5s
</match>
7、进入mongodb中授权用户及分配权限
```

![Snipaste_2025-09-26_17-56-26](images/Snipaste_2025-09-26_17-56-26.png)

```
8、td-agent和mongodb设置完成后，重启td-agent
systemctl restart td-agent
9、数据库中就可以查看数据是否过来
```

# 整合mongodb到项目

>fluentd:轻量级日志采集工具
>
>ELK:重量级日志采集工具==elasticsearch+logshash+kibana

![Snipaste_2025-09-27_11-07-44](images/Snipaste_2025-09-27_11-07-44.png)

```
修改td-agent配置文件（主要是web服务器要收集日志的path，host等）
重启td-agent：systemctl restart td-agent
			systemctl enable td-agent(td-agent没有报错就是成功)

查看数据库：mongosh mongodb://192.168.66.8:27017/admin
use admin
db.auth("admin","123456")
use logs 
show collection
db.nginx_access.find()
```

>一般web服务器能承受2-3w的并发，有5台大概能承受10w左右的并发。能承受多少并发一般是由web服务器所决定的。