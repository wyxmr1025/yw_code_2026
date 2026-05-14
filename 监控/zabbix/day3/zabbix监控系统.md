# Zabbix监控系统

作用：

Prometheus比较适用于传统监控，也比较适用于容器监控（使用更广泛）。

Zabbix比较老牌的监控系统，很多传统行业，如银行、国企还是基于Zabbix进行监控操作，适用于传统监控。

# 场景说明

运维工程师除了搭建架构环境，配置管理外，还需要保证业务的**==稳定==**运行。不稳定的情况包括很多方面,如:

* CPU负载过大
* 内存不够
* 磁盘空间满了
* 网络很卡
* 服务不能被访问

等等各种问题。我们运维工程师无法做到时刻盯着服务器查看各类状态，所以需要建立一套完善的**==自动化监控==**系统，将所有需要监控的服务器及其各种需要的状态数据都实时地**收集**, **图形展示**,**报警。**

![1578927121943](images/%E7%9B%91%E6%8E%A7%E6%9E%B6%E6%9E%84%E5%9B%BE.png)



# **学习目标**

- [ ] 知道监控的目的与目标

- [ ] 能够安装zabbix服务器
- [ ] 能够使用zabbix-agent监控本机与远程linux
- [ ] 能够说出模板的作用
- [ ] 掌握自定义监控项的方法
- [ ] 能够为添加的监控项创建图形
- [ ] 能够为监控项设定触发器
- [ ] 能够实现zabbix报警
- [ ] 能够通过自动发现与动作实现自动监控
- [ ] 能够说出导入导出功能的作用
- [ ] 能够通过proxy来实现监控



# 一、认识监控

## 1, **监控的目的**

* 实时收集数据并图形展示, 通过报警及时发现问题与处理问题。

* 为架构优化也提供依据。



## 2, 监控的目标

**生活中的监控:**

![1541674720378](images/1541674720378.png)

![1541674782863](images/1541674782863.png)



那么**请问linux系统中的监控主要监控什么**?

* **任何你所想要监控的数据**, 如cpu负载,cpu的idle时间,内存使用量,内存利用率,io,network等等。
* 现在很多开源监控方案已经把常见的监控做成了模板，我们可以直接套用
* 大型公司会有更多的监控需求, 那么就需要专业的开发人员来做监控开发(运维人员也可以开发)

![1578930808598](images/%E7%9B%91%E6%8E%A7%E4%BA%8B%E9%A1%B9.png)



## 3, 主流的开源监控平台介绍

* **mrtg**	(Multi Router Traffic Grapher)通过**snmp**协议得到设备的流量信息，并以包含PNG格式的图形的HTML文档方式显示给用户。

* **cacti**       (仙人掌) 用php语言实现的一个软件，它的主要功能是用snmp服务获取数据，然后用rrdtool储存和更新数据。官网地址: https://www.cacti.net/
* **ntop**      官网地址: https://www.ntop.org/
* **nagios**   能够跨平台,插件多,报警功能强大。官网地址: https://www.nagios.org/
* **centreon**  底层使用的就是nagios。是一个nagios整合版软件。官网地址:https://www.centreon.com/
* **ganglia**    设计用于测量数以千计的节点,资源消耗非常小。官网地址:http://ganglia.info/
* **open-falcon**  小米公司开源,高效率,高可用。用户基数相对小。官网地址: http://open-falcon.org/
* **==zabbix==**     跨平台,画图,多条件告警,多种API接口。用户基数大。官网地址: https://www.zabbix.com/
* **==prometheus==** 基于时间序列的数值数据的容器监控解决方案。官网地址: https://prometheus.io/



# 二、zabbix

https://www.zabbix.com/

![image-20241120141601325](images/image-20241120141601325.png)

## 1, zabbix基础概念初探

1. **主机(host)和主机群组(host group)**	

主机指被监控的一个设备(服务器,交换机等)，当被监控的主机数量巨大时，就需要分组

2. **zabbix用户(user)与用户群组(group)**

zabbix可以多个用户登录管理(和Linux操作系统一样有管理员和普通管理者)

3. **监控项(item)与应用集(application)**

监控的需求太多了,就拿监控cpu平均负载来说,就有监控1分钟内,5分钟内,15分钟内等三个常见的监控参数。

监控项(item)是从收集数据或监控的一个**最小单位**。把cpu1分钟内的平均负载就可以做成一个监控项。

应用集就是多个监控项的组。

4. **图形**

监控项收集的数据需要用图形直观地展示出来。

5. **触发器和报警**

当监控项收集的数据达到一个临界点时，就要触发报警通知管理人员。

如: 当根分区使用率超过80%时, 就通过发报警信息到管理人员。

6. **模板**

模板主要包括监控项,图形,触发器等概念，相当于是把要监控的东西做成一个合集。



## 2, 监控场景准备

**环境准备:** 这里为1台监控服务器和2台被监控端

![1579330958191](images/%E5%AE%9E%E9%AA%8C%E7%8E%AF%E5%A2%83%E5%9B%BE.png)

1. 静态ip 

2. 主机名

~~~powershell
各自配置好主机名
# hostnamectl set-hostname --static server

三台都互相绑定IP与主机名
# vim /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.88.201  server
192.168.88.202  agent1
192.168.88.203  agent2
~~~

3. 时间同步

~~~powershell
# yum install epel-release -y
# yum install ntpsec -y
# ntpdate cn.ntp.org.cn
-------------------------------------
# systemctl restart ntpd
# systemctl enable ntpd
~~~

4. 关闭防火墙,selinux

~~~powershell
# systemctl stop firewalld
# systemctl disable firewalld
# iptables -F

# setenforce 0
setenforce: SELinux is disabled
~~~

# 三、zabbix服务器安装

Zabbix Server/Zabbix Agent(采集)

Zabbix底层基于PHP + MySQL开发的，需要搭建LAMP 或 LNMP架构

参考: https://www.zabbix.com/documentation/current/manual/installation/install_from_packages/rhel_centos

**zabbix服务器结构图**

![1561796495280](images/zabbix%E5%9F%BA%E7%A1%80%E5%8E%9F%E7%90%86%E6%9E%B6%E6%9E%84%E5%9B%BE.png)

zabbix = zabbix-server（服务器端） +   zabbix-agent（类似Prometheus中的xxx_exporter）

## 1，MySQL8部署

第一步：下载MySQL

```powershell
# wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.37-1.el9.x86_64.rpm-bundle.tar
```

网速比较慢，也可以直接使用课件中的MySQL离线安装包直接安装！

第二步：解压MySQL

```powershell
# mkdir /root/mysql
# tar -xf mysql-8.0.37-1.el9.x86_64.rpm-bundle.tar -C /root/mysql

x：代表 "extract"，意思是提取归档文件中的内容。
f：告诉 tar 命令，接下来的参数是一个文件名，而不是要归档或提取的文件列表。
-C（注意大写）：这是一个选项，允许您指定在何处（即哪个目录）解压缩文件。
```

第三步：安装MySQL

```powershell
# cd /root/mysql
# rpm -ivh mysql-community-* --force --nodeps

-i: 安装软件包。
-v: 显示详细输出。
-h: 显示安装进度。
--force: 强制覆盖已经存在的文件。这可能会破坏系统，因为它会覆盖现有的文件而不进行任何检查。
--nodeps: 安装软件包时，不检查其依赖关系。这可能会导致软件包无法正常工作，因为它可能缺少必要的库或依赖项。
```

第四步：启动MySQL，并开机启动

```powershell
#启动MySQL
[root@localhost ~]# systemctl start mysqld
 
#MySQL加入开启自启
[root@localhost ~]# systemctl enable mysqld
```

第五步：查看初始密码

```powershell
#查询初始密码
[root@localhost ~]# cat /var/log/mysqld.log | grep password
lk_WKkHMg3d%
```

第六步：修改密码

如果你想要设置简单的密码，你需要先更换一个初始密码，然后再修改策略，再修改简单密码，所以先使用一个复杂的密码作为新密码。【注意：数据库中输入的指令注意“；”这个符合别漏掉】

```powershell
# mysql -uroot -p
Enter password:输入初始密码
 
# 使用root登入mysql，密码使用使用上方查询出来的密码→修改密码为Xa%^3T%T4!
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'Xa%^3T%T4!';
 
# 设置密码检查等级
mysql> set global validate_password.policy=0;
 
# 0 (或LOW)：允许密码包含最简单的密码（只包含小写字母、数字、特殊字符中的一类）。
# 1 (或MEDIUM)：要求密码至少包含小写字母、大写字母、数字、特殊字符中的三类。
# 2 (或STRONG)：要求密码至少包含小写字母、大写字母、数字、特殊字符中的四类。
 
# 密码的最短长度
mysql> set global validate_password.length=5;
 
# 密码至少要包含的小写字母个数和大写字母个数
mysql> set global validate_password.mixed_case_count=0;
 
# 再次修改密码为qwe!123，并退出数据库
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'qwe!123';
 
# 退出
mysql> exit
```

## 2，安装PHP8.0

第一步：加载remi库

```powershell
# sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm
 
 官方仓库：由Linux发行版的官方团队维护，包含经过严格测试和官方支持的软件包。
Remi仓库：由Remi个人或他的团队维护，提供更新版本的软件包或官方仓库中没有的软件包。这些软件包可能没有得到官方支持，但由Remi团队维护。
仓库：/etc/yum.repos.d目录
```

安装完成后，重建缓存

```powershell
#清除dnf的缓存【如RPM软件包文件、旧的headers和元数据（元数据描述了可用的软件包和它们的依赖关系）、dnf生成的其他临时文件】
[root@localhost ~]# dnf clean all 
 
#下载并缓存元数据
[root@localhost ~]# dnf makecache
```

第二步：安装PHP8和其相关的组件

```powershell
#下面是Zabbix server在使用PHP时需要用到的PHP相关组件的安装：
[root@localhost ~]# sudo dnf install -y php80 php80-php-cli php80-php-gd php80-php-json php80-php-mbstring php80-php-mysqlnd php80-php-xml

php80：
这是PHP 8.0的主要软件包，包含了PHP的核心解释器和基础功能。

php80-php-cli：
PHP的命令行接口（CLI），允许你从命令行运行PHP脚本。这在开发、测试和维护PHP应用程序时非常有用。

php80-php-gd：
#GD库是一个用于处理图像的开源代码库，它提供了一系列函数来动态创建和操作图像。在Zabbix中，它可能被用于图形显示。

php80-php-json：
提供了对JSON格式的解析和生成支持。JSON是Web服务中常用的数据交换格式。

php80-php-mbstring：
提供了对多字节字符串的支持，包括UTF-8等字符集。对于国际化应用非常重要。

php80-php-mysqlnd：
MySQL的原生驱动（Native Driver），用于与MySQL数据库进行交互。由于Zabbix通常使用MySQL作为其后端数据库，因此这个扩展是必须的。

php80-php-xml：
提供了对XML的支持，包括解析和生成XML文档。虽然Zabbix不直接依赖于XML处理，但某些PHP扩展或应用程序可能需要它。

php80-php-bcmath：
BC Math库提供了高精度数学计算的支持。虽然Zabbix本身可能不需要它，但某些PHP应用程序或扩展可能需要它。

php80-php-fpm：
FastCGI进程管理器（FPM）是一个用于管理PHP FastCGI进程的工具。它允许你将PHP作为独立的进程运行，而不是作为Apache或Nginx的模块。如果你打算使用Nginx作为Web服务器，并希望PHP作为FastCGI进程运行，那么这个扩展是必须的。
```

## 3，安装Zabbix

第一步：加载zabbix软件库

```powershell
# rpm -Uvh https://repo.zabbix.com/zabbix/7.0/alma/9/x86_64/zabbix-release-7.0-2.el9.noarch.rpm
简单理解就是加载了zabbix官方的应用商店
```

第二步：关闭无关的库

进入软件库文件夹/etc/yum.repos.d/打开epel.repo文件

![image-20241120144529849](images/image-20241120144529849.png)

`enabled = 1改成enabled = 0`

![image-20241120144606999](images/image-20241120144606999.png)

因为安装zabbix所需组件时候，如果不关闭其他库，系统会去其他库中寻找相关组件，导致版本不对等问题，使得安装出现失败，现象如下：

![image-20241120144723667](images/image-20241120144723667.png)

第三步：安装zabbix所需组件

```powershell
dnf install -y zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf zabbix-sql-scripts zabbix-selinux-policy zabbix-agent

zabbix-server-mysql：
这是Zabbix服务器的软件包，它用于收集和处理来自客户端（如Zabbix代理或SNMP设备）的监控数据。这个软件包是为MySQL数据库设计的，因此它依赖于MySQL或MariaDB来存储配置、历史和触发器等信息。

zabbix-web-mysql：
这个软件包包含了Zabbix的前端Web界面，允许用户通过Web浏览器来查看和管理监控数据。这个软件包也是为MySQL数据库设计的，因此它会与zabbix-server-mysql协同工作。

zabbix-apache-conf：
这个软件包提供了Apache HTTP服务器的配置文件，以便能够正确地为Zabbix的Web界面提供服务。如果你使用的是Apache作为Web服务器，这个软件包会帮助你将Zabbix Web界面集成到Apache中。

zabbix-sql-scripts：
这个软件包包含了创建Zabbix数据库结构所需的SQL脚本。在安装Zabbix服务器之前，你需要使用这些脚本来创建和初始化数据库结构。这些脚本是为MySQL和PostgreSQL等数据库设计的。

zabbix-selinux-policy：
如果你的系统启用了SELinux（Security-Enhanced Linux），这个软件包会提供必要的SELinux策略，以确保Zabbix能够正常运行而不会被SELinux阻止。SELinux是一个安全模块，用于提供强制访问控制。

zabbix-agent：
Zabbix代理是一个可以安装在被监控设备上的软件，用于收集设备上的监控数据并将其发送到Zabbix服务器。虽然这个命令是在服务器上执行的，但安装zabbix-agent也可以让你在同一台服务器上监控其本地资源。
```

第四步：重新启用epel.repo

![image-20241120145312460](images/image-20241120145312460.png)

## 4，配置Zabbix数据库

第一步：创建数据库

```powershell
#进入数据库
[root@localhost zabbix]# mysql -uroot -p
Enter password:输入之前设置root访问数据库的密码 => qwe!123
 
#创建了一个名为zabbix_proxy的新数据库，该数据库使用utf8mb4字符集和utf8mb4_bin校对规则，这使得它特别适合于存储包含特殊字符（如emoji）的数据，并且以二进制方式比较和排序字符串。
mysql> create database zabbix character set utf8mb4 collate utf8mb4_bin;

utf8mb4：
是一个Unicode字符集，可以存储多达4字节的字符。这个字符集特别有用，因为它可以存储像emoji这样的特殊字符，而标准的utf8字符集（实际上是utf8mb3）则不能。

utf8mb4_bin:
utf8mb4_bin是一个二进制校对规则，它基于字符的二进制值进行比较。这意味着它区分大小写，并且不会考虑字符的排序规则（如字母表顺序）。如果你想要一个不区分大小写的校对规则，你可能会选择utf8mb4_general_ci或utf8mb4_unicode_ci等。
```

第二步：设置密码并授权

```powershell
#创建一个数据库访问账号zabbix，密码为qwe!123
mysql> create user zabbix@localhost identified by 'qwe!123';
 
#授予用户 zabbix 在 zabbix 数据库上所有权限，并且这个权限仅适用于从 localhost 主机连接到 MySQL 服务器的用户。
mysql> grant all privileges on zabbix.* to zabbix@localhost;

授予权限：用户 zabbix 将获得在 zabbix 数据库上执行任何操作的能力，包括创建表、删除表、插入数据、更新数据、查询数据等。

指定数据库：这里的 zabbix.* 表示 zabbix 数据库中的所有对象（表、视图、存储过程等）。

指定用户：zabbix@localhost 表示用户名为 zabbix 的用户，并且这个用户只能从 localhost 连接到 MySQL 服务器。localhost 通常指的是 MySQL 服务器所在的机器。

权限范围：这个权限仅限于zabbix数据库，不影响其他数据库。
```

第三步：临时降低安全性

```powershell
#告诉 MySQL 信任你，即使你正在创建或修改一个非确定性函数，你也知道如何确保主服务器和从服务器之间的数据一致性。为了防止后面导入初始数据库时会出现报错，所以临时降低安全性。
mysql> set global log_bin_trust_function_creators = 1;
 
#退出
mysql> quit;
```

这个指令需要一定的数据库知识才能理解，这边尝试解释下，能理解理解下，不能理解照着输入即可：

非确定函数是指：那些在相同的输入下可能会返回不同结果的函数。它们通常依赖于数据库之外的信息，比如当前时间、随机数生成器、数据库的其他部分（表或行）的状态等。在MySQL中，你定义的存储函数可以是确定性的或非确定性的。但是，即使你的数据库是新建的，并且还没有定义任何存储函数，你仍然可能会在尝试定义一个非确定性的存储函数时遇到问题，特别是当二进制日志（binary logging）被启用时。

MySQL为了保持复制的一致性，默认不允许在启用二进制日志时创建非确定性的存储函数。这是因为在主服务器上执行的非确定性函数可能会在从服务器上产生不同的结果，从而导致数据不一致。

通过设置`log_bin_trust_function_creators`为1来允许创建这样的函数。但是，这样做之前，请确保你了解可能带来的风险，并且只在测试或开发环境中这样做。

如果不配配置中，后面再创建初始数据的时候可能会报错，所以先打开，初始数据创建完成后再关闭。

第四步：导入数据库

```powershell
#导入zabbixd的初始数据库
[root@localhost ~]# zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
 
Enter password:输入之前设置zabbix访问数据库的密码
```

官方提供的命令行，大致意思是读取`/usr/share/zabbix-sql-scripts/mysql/server.sql.gz`文件（Zabbix的MySQL初始化脚本）的内容，并将其直接导入到名为`zabbix`的MySQL数据库中，使用`zabbix`用户名和提示输入的密码进行身份验证，并确保在数据传输过程中使用`utf8mb4`字符集。

```powershell
#关闭修改非确定性函数
[root@localhost zabbix]# mysql -uroot -p
Enter password:输入之前设置root访问数据库的密码
mysql> set global log_bin_trust_function_creators = 0;
mysql> quit;
```

## 5，配置conf文件

修改zabbix应用程序的配置文件/etc/zabbix/中的zabbix_server.conf数据库密码和监听端口参数，找个空白处输入进去即可。

```powershell
vim /etc/zabbix/zabbix_server.conf
#设置数据库密码位上方设置的zabbix密码qwe!123：
DBPassword=qwe!123
 
#设置监听客户端端口位10051：
ListenPort=10051
```

![image-20241120152750613](images/image-20241120152750613.png)

## 6，开启端口与服务

如果生产环境，可以参考如下规则设置防火墙：

```powershell
# 查看80端口状态，NO表示没打开，YES表示打开。
[root@localhost ~]# firewall-cmd --zone=public --query-port=80/tcp
 
#开启80、10050、10051端口，重启依然生效
[root@localhost ~]# firewall-cmd --zone=public --add-port=80/tcp --permanent
[root@localhost ~]# firewall-cmd --zone=public --add-port=10050/tcp --permanent
[root@localhost ~]# firewall-cmd --zone=public --add-port=10051/tcp --permanent
 
# 重新加载防火墙配置，立即生效
[root@localhost ~]#firewall-cmd –reload
```

重启服务

```powershell
#重新启动zabbix-server、zabbix-agent、httpd、php-fpm服务
[root@localhost ~]# systemctl restart zabbix-server zabbix-agent httpd php-fpm
 
#加入开机自动启动
[root@localhost ~]# systemctl enable zabbix-server zabbix-agent httpd php-fpm

zabbix-server（zabbix服务端）：
这是 Zabbix 监控系统的服务器端。Zabbix 是一个开源的监控解决方案，用于监控网络、服务器、应用程序等。Zabbix 服务器负责接收来自代理和其他客户端的数据，并存储这些数据以供进一步的分析和报告。

zabbix-agent（zabbix客户端）：
这是 Zabbix 监控系统的代理端。Zabbix 代理部署在要监控的服务器上，负责收集该服务器的各种指标（如 CPU 使用率、内存使用情况、磁盘空间等），并将这些数据发送到 Zabbix 服务器。

httpd（httpd 负责处理HTTP请求和响应，而 php-fpm 负责执行PHP代码）：
这通常是 Apache HTTP 服务器的服务名称。Apache 是一个流行的 Web 服务器软件，用于托管网站和提供 Web 服务。如果您正在使用 Zabbix 的 Web 前端，那么 Apache 服务器可能用于提供对 Zabbix Web 界面的访问。

php-fpm（负责执行PHP代码）：
这是 PHP FastCGI 进程管理器（PHP-FPM）的服务名称。PHP-FPM 是一个 FastCGI 进程管理器，用于管理 PHP 解释器的进程。如果您的 Zabbix Web 前端是用 PHP 编写的，那么 PHP-FPM 将负责处理 PHP 脚本的执行
```

## 7，Web配置

第一步：打开浏览器，输入“[http://IP/zabbix/](http://ip/zabbix/)”进行访问（IP指的是服务器的IP地址），会自动跳转到配置页面，选择中文，点击下一步。

![image-20241120160921457](images/image-20241120160921457.png)

第二步：他会检测PHP的参数是否配置正确上面没有配置错误的话一般都是正常的，如果有问题就到/etc/php.ini修改这个文件的参数即可。

![image-20241120161015365](images/image-20241120161015365.png)

第三步：输入zabbix用户访问数据库的密码，然后下一步

![image-20241120161409193](images/image-20241120161409193.png)

第四步：设置主机名（自己设置即可），我喜欢暗色，所以主题选了暗色，你们自行选择，然后直接下一步到完成。

![image-20241120161607024](images/image-20241120161607024.png)

单击下一步，下一步，出现如下界面，代表安装完成：

![image-20241120161656931](images/image-20241120161656931.png)

第五步：登录Zabbix，默认账号：Admin（注意首字母大写），密码：zabbix

![image-20241120162000153](images/image-20241120162000153.png)

登录后，如下图所示：

![image-20241120162200545](images/image-20241120162200545.png)

## 8，解决语言包问题与图表字体乱码问题

第一步：安装中文语言包

```powershell
sudo yum install glibc-common -y
sudo yum install glibc-langpack-en glibc-langpack-zh -y
```

第二步：进入用户设置，找到User settings => Profile

![image-20241120174625579](images/image-20241120174625579.png)

第三步：监测 => 主机 => 图形设置

![image-20241120174730569](images/image-20241120174730569.png)

出现中文乱码

![image-20241120174819454](images/image-20241120174819454.png)

第四步：修复方案

注意：选择楷体，其他字体可能导致显示异常，图示是win11系统操作方法，win10类似，自行研究即可

![image-20241120175502932](images/image-20241120175502932.png)

上传字体于`/usr/share/zabbix/assets/fonts/`中

![image-20241120175817465](images/image-20241120175817465.png)

打开`/usr/share/zabbix/include/defines.inc.php`文件，修正字体指向配置。

![image-20241120180633656](images/image-20241120180633656.png)

刷新网页，如下图所示：

![image-20241120180912767](images/image-20241120180912767.png)

## 9，输入IP直接进入主页，不用加zabbix后缀

假如你希望[http://IP就能访问主页，而不是http://ip/zabbix访问主页](http://xn--iphttp-qr3j87j97vj11k//ip/zabbix自动重定向)，那么如下图所示配置即可：

`vim /etc/httpd/conf/httpd.conf`

![image-20241120181442964](images/image-20241120181442964.png)

更改完成后，重启Apache软件（httpd软件）

```powershell
systemctl restart httpd
```

# 四、zabbix服务器监控本机

概念:

* **主机(host)**: 指被监控的一个设备(服务器,交换机等)

* **主机群组(hostgroup)**: 指被监控的一组主机（主要应用在有特别多主机的情况，方便分组区分)



zabbix服务器端默认配置了监控本机，但还需要安装客户端收集工具:zabbix-agent。

![image-20250305234739670](./images/image-20250305234739670.png)

## 1, 服务器上安装zabbix-agent（可忽略）

~~~powershell
[root@server ~]# dnf install zabbix-agent
~~~

## 2, 启动zabbix-agent服务

请使用vi或vim打开agent端配置文件`/etc/zabbix/zabbix_agentd.conf`修改,修改后的结果如下

```powershell
[root@server ~]# egrep -vn '^#|^$' /etc/zabbix/zabbix_agentd.conf
13:PidFile=/var/run/zabbix/zabbix_agentd.pid
32:LogFile=/var/log/zabbix/zabbix_agentd.log
43:LogFileSize=0
98:Server=127.0.0.1							# zabbix服务器的IP，agent被动监控(默认模式)
139:ServerActive=127.0.0.1			# zabbix服务器的IP，agent主动监控
150:Hostname=server					   # zabbix服务器的主机名
290:Include=/etc/zabbix/zabbix_agentd.d/*.conf
```

**说明:** 

* 默认为相对于agent的被动监控,表示server找agent拿数据, 而不是agent主动给数据server
* 主动与被动只是数据传输的方式不同, 具体区别我们在最后的章节讨论
* 我这里只修改了第150行的主机名,其它参数都为默认值未修改

~~~powershell
[root@server ~]# systemctl restart zabbix-agent
[root@server ~]# systemctl enable  zabbix-agent

[root@server ~]# netstat -ntlup |grep :10050
tcp    0    0 0.0.0.0:10050      0.0.0.0:*       LISTEN      65171/zabbix_agentd
tcp6   0    0 :::10050           :::*            LISTEN      65171/zabbix_agentd
~~~

## 3, 确认本机监控状态

![image-20250308171835766](images/image-20250308171835766.png)

**监控状态不OK的排错思路:**

* 查看日志`cat /var/log/zabbix/zabbix_server.log`



# 五、监控远程linux服务器

目标：使用Zabbix实现远程linux服务器监控

## 1, agent1上安装zabbix-agent

~~~powershell
[root@agent1 ~]# rpm -Uvh https://repo.zabbix.com/zabbix/7.0/alma/9/x86_64/zabbix-release-7.0-2.el9.noarch.rpm
[root@agent1 ~]# dnf install zabbix-agent -y

注：如果/etc/yum.repos.d目录存在epel.repo，必须先把epel.repo关闭，然后安装完成后在启用！！！
~~~

## 2, 配置agent端并启动服务

配置`/etc/zabbix_agentd.conf`配置文件，配置结果如下:

~~~powershell
[root@agent1 ~]# egrep -vn '^#|^$' /etc/zabbix_agentd.conf
13:PidFile=/var/run/zabbix/zabbix_agentd.pid
32:LogFile=/var/log/zabbix/zabbix_agentd.log
43:LogFileSize=0
117:Server=192.168.88.201					    修改成zabbix监控服务器的IP,agent被动模式
171:ServerActive=192.168.88.201			  修改成zabbix监控服务器的IP,agent主动模式
182:Hostname=agent1								   修改为被监控端的主机名
~~~

~~~powershell
[root@agent1 ~]# systemctl restart zabbix-agent
[root@agent1 ~]# systemctl enable zabbix-agent

[root@agent1 ~]# netstat -ntlup |grep :10050
tcp    0   0 0.0.0.0:10050        0.0.0.0:*        LISTEN      7413/zabbix_agentd
tcp6   0   0 :::10050             :::*             LISTEN      7413/zabbix_agentd
~~~

## 3, web管理界面创建监控主机

回到web管理界面－－》点监测－－》点主机 －－》 点创建主机

![image-20241120183652823](images/image-20241120183652823.png)

设置主机名称、主机群组以及接口信息

![image-20241120190706915](images/image-20241120190706915.png)

选择模板信息，如下图所示：

![image-20241120190823311](images/image-20241120190823311.png)



## 4, 确认监控OK

![1579084898226](images/zabbix21.png)

**监控不OK的排错思路:**

* 检查server与agent1的网络是否OK，防火墙是否关闭
* 检查IP与端口是否写错
* 在agent端查看日志`cat /var/log/zabbix/zabbix_agentd.log`
* 在server端查看日志`cat /var/log/zabbix/zabbix_server.log`



# 六、模板

## 模板介绍与作用

**模板(template)**: 是包括监控项，应用集，触发器，图形，聚合图形，自动发现，web监测等的一组实体。

**==使用模板可以方便应用到主机，更改模板也会将更改应用到所有链接的主机==**。

server、agent1、agent2：公用一个Linux by Zabbix Agent，如果更改了这个模板，则所有主机监控项也会随之改变！

例: 比如我要把监控nginx相关的全部做成一个模板，有100台服务器需要监控nginx，我只需要链接模板到这100台机器即可。以后需要修改，只需要修改模板，这100台就会被同时修改。

数据采集 =》模板：

![image-20250308181806276](images/image-20250308181806276.png)





## 为主机添加或删除模板

zabbix自带了很多实用的模板, 对于一些要求不高的公司来说, 直接将模板添加到监控主机都几乎够用了。



![1579098438828](images/zabbix23.png)

![1579099902498](images/zabbix24.png)

![1579099983287](images/zabbix25.png)

![1579100084571](images/zabbix26.png)

![1579100154673](images/zabbix27.png)

![1579100242876](images/zabbix28.png)

## 创建自定义模板

数据采集->模板->创建模板：

![1579100352191](images/zabbix29.png)

![1579100568737](images/zabbix30.png)

![1579100785403](images/zabbix31.png)



**练习:** 请将agent1其它模板都清空，只保留刚刚自定义的`Template test`模板。

操作的最终结果如下:

![1579101068432](images/zabbix32.png)



# 七、监控项与应用集

**监控项(item)**: 是从主机收集的数据信息,代表收集数据或监控的一个**最小单位**。

比如cpu1分钟内平均负载,内存空闲值,磁盘使用率等等都可以做为监控项，可以说监控项有无限种可能。

**应用集(applications)**: 代表多个监控项目合成的组。



## 创建监控项的方式

创建监控项的方法有2种:

1. 在某一台被监控机上创建(如下图所示)，这样创建的监控项只对此监控机生效。

![1579102219637](images/zabbix33.png)

2. 在模板里创建(如下图所示), 这样创建的监控项对所有使用此模板的主机生效(**推荐方式**)。

![1579102343190](images/zabbix34.png)

![1579102406593](images/zabbix35.png)



## 创建自带键值监控项

创建监控项中最核心的概念就是**==键值(key)==**。

**键值就看作是开发好的用于收集数据的命令**，主要有两种:

- **zabbix自带的键值**(太多了,不用特意去记忆)
- **自定义开发的键值**(用linux基础命令就可以开发)



案例: 使用zabbix自带键值创建监控项实现监控cpu的1分钟内平均负载

键值写法可参考下图:

![1579103212553](images/zabbix36.png)



### 1, 在模板里创建监控项

![1579102343190](images/zabbix34.png)

![1579102406593](images/zabbix35.png)

### 2, 填写监控项相关信息

https://www.zabbix.com/documentation/7.0/zh/manual/config/items/itemtypes/zabbix_agent

![1579154201717](images/zabbix37.png)

![1579153272044](images/zabbix38.png)

### 3, 确认创建成功

![1579153367459](images/zabbix39.png)

![1579153471810](images/zabbix40.png)





**练习:** 将cpu五分钟内平均负载, cpu十五分钟内平均负载分别做成cpu_avg5,cpu_avg15两个监控项

最终结果如下:

![1579154373049](images/zabbix41.png)



## 基于AI大模型扩展监控项说明

目标：基于AI大模型生成自定义监控项

监控磁盘中根分区使用率，磁盘IO，网络进出情况

https://kimi.moonshot.cn/，输入提示词，越精准越好

Zabbix版本 + 具体指标 + 期望返回结果

```
比如我想创建zabbix监控项，如果是CPU，可以采集那些具体指标？如何实现
```

![image-20250309101357040](images/image-20250309101357040.png)

![image-20250309101515229](images/image-20250309101515229.png)



## 创建自定义键值监控项

以监控登录用户数为例，自带键值中有`system.users.num`这个键值，但我们不使用它，使用自定义的键值来实现。



### 1, 在被监控端agent1上操作

首先在agent1多打开几个终端,模拟多个登录用户,然后使用`who |wc -l`查询

```powershell
[root@agent1 ~]# who |wc -l
15							我这里确认登录用户数为15
```

然后在agent1上,定义UserParameter

```powershell
[root@agent1 ~]# vim /etc/zabbix_agentd.conf 

324 UserParameter=loginusers,who | wc -l

说明: loginusers是我自定义的一个键值名称（会在创建监控项时用到),后面的who |wc -l就要被监控的命令
```

重启zabbix-agent服务使之生效

~~~powershell
[root@agent1 ~]# systemctl restart zabbix-agent
~~~



### 2, 在zabbix监控端上操作（可选）

在zabbix服务器安装`zabbix-get`工具，可以远程测试能否通过自定义的键值得到数据

```powershell
[root@server ~]# yum install zabbix-get -y

[root@server ~]# zabbix_get -s 192.168.88.202 -k loginusers
15					可以确认得到的值确实为agent1的登录用户数
```

说明:

* -s后接agent端的IP
* -k接agent端自定义的键值

### 3, 在web管理界面创建监控项

还是在自定义模板里创建

![1579168009677](images/zabbix42-17321541731251.png)

![1579168267856](images/zabbix43-17321541731262.png)

![1579168481402](images/zabbix44-17321541731263.png)

### 4, 确认创建成功

![1579168970778](images/zabbix45-17321541731264.png)



![1579169131554](images/zabbix46-17321541731265.png)

小结：

监控项：其实即使我们平常所说的监控指标

支持的方式：系统键值、自定义键值、还可以通过AI大模型辅助

# 八、图形与仪表板（聚合图形）

监控项创建好了, 但是它监控收集的数据在哪里看呢? 答案就是**图形**

## 创建图形显示监控项数据

### 1, 在模板里创建图形

数据采集 =》 模板：

![1579169315713](images/zabbix47.png)

![1579169363263](images/zabbix48.png)

### 2, 配置图形对应监控项

![1579169642292](images/zabbix49.png)



![1579169745837](images/zabbix50.png)

### 3, 验证图形

监测 =》 主机 =》 点图形：

![1579169833817](images/zabbix51.png)



![1579169878549](images/zabbix52.png)

![1579169916611](images/zabbix53.png)

![1579169995554](images/zabbix54.png)

**练习: **请将前面自定义的登录用户数这个监控项也做成图形

最终结果如图: 

![1579172114342](images/zabbix55.png)



## 仪表板（监控大屏）

**仪表板:** 就是把多个重要常用的图形整合一起来显示,方便查看.

假设需要经常查看agent1的cpu负载与登录用户数这两张图,我们可以将其聚合到一起做成一张聚合图形



### 1, 创建仪表板

![image-20250308001916357](./images/image-20250308001916357.png)

![image-20250308001957668](./images/image-20250308001957668.png)

创建完成后，如下图所示：

![image-20250308002101051](./images/image-20250308002101051.png)

### 2, 编辑图形

添加要监控主机：

![image-20250308002431586](./images/image-20250308002431586.png)

添加监控项，包括1分钟负载、5分钟负载、15分钟负载

![image-20250308002627655](./images/image-20250308002627655.png)

设置完成后，如下图所示：

![image-20250308002843816](./images/image-20250308002843816.png)

添加System登录用户数

![image-20250308003100325](./images/image-20250308003100325.png)

添加监控项

![image-20250308003258770](./images/image-20250308003258770.png)

整合后如下图所示：

![image-20250308003351622](./images/image-20250308003351622.png)

### 3，单击保存设置

![image-20250308003842511](./images/image-20250308003842511.png)



# 九、触发器

虽然我们可以通过图形查看到监控的数据，但我们不可能一直盯着图形的变化。

所以需要定义监控项到达一个临界值(阈值)或者满足一个条件，就会发生状态变化的通知。

定义**触发器(trigger)**就是定义这个临界值(阈值)或条件.

监控项有无限种可能，触发器也一样有无限种可能。如: 

* cpu负载值大于某个值则通知
* 登录用户数大于某个值则通知
* 内存空闲率小于某个值则通知
* 磁盘使用率大于某个值则通知
* 主机名被修改则通知

等等，主要还是看需求。

## 创建登录用户数过多的触发器

### 1, 在模板里创建触发器

![image-20250308004256984](./images/image-20250308004256984.png)

![1579177103686](images/zabbix67.png)

### 2, 配置触发器

![1579177551294](images/zabbix68.png)



![1579177790405](images/zabbix69.png)

![1579178242790](images/zabbix70.png)

### 3, 验证创建成功

![1579178615519](images/zabbix71.png)

![1579178691679](images/zabbix72.png)

### 4, 验证触发器效果

先在agent1上再多打开几个终端，将登录用户数控制在20个以上(操作过程省略)

然后通过下图查看触发器通知

![1579178965780](images/zabbix73.png)



![1579240449765](images/zabbix73-1.png)

**自由思维与操作练习:  请将cpu负载的相关监控项也创建对应的触发器并验证。**



# 十、报警

触发器的通知信息显示在web管理界面, 运维工程师仍然没办法24小时盯着它。所以我们希望它能自动地通知工程师们，这就是报警。

zabbix的报警媒介支持email,jabber,sms(短信),微信,企业微信、钉钉、电话语音等。

## 报警过程原理

![1579191707006](images/zabbix74.png)



## 报警平台申请

自己配置报警过程比较复杂，需要配置触发器动作，用户与其报警媒介，最麻烦的是写程序对接邮件,微信,短信,电话等接口.

* 邮件容易被拒，当做垃圾邮件
* 微信需要企业微信号并开发程序对接
* 短信一般都需要付费买运营商相关服务
* 电话语言需要更专业的开发

以上要求对于没有开发能力和开发支持的运维工程师来说，难度较大。

所以我们这里选择专业的报警平台就可以帮助实现一体化报警方案。



如:onealeart   参考:http://www.onealert.com/

请先申请一个账号,绑定邮箱,手机,微信等(过程省略)。



登录进去后,按如下图示操作

![1568533070324](images/onealert%E6%96%B0%E7%89%88-1)

## 报警平台增加zabbix应用

![1568533415668](images/onealert%E6%96%B0%E7%89%88-2)



![1568533560005](images/onealert%E6%96%B0%E7%89%88-3.png)





![1568533629600](images/onealert%E6%96%B0%E7%89%88-4.png)

## server上安装报警agent

![image-20250309120139644](images/image-20250309120139644.png)

**按照提示进行安装**

```powershell
[root@server ~]# cd /usr/lib/zabbix/alertscripts
[root@server alertscripts]# wget https://download.aiops.com/ca_agent/zabbix/ca_zabbix_release-4.0.4.tar.gz

[root@server alertscripts]# tar -xzf ca_zabbix_release-4.0.4.tar.gz
[root@server alertscripts]# cd cloudalert/bin/

[root@server bin]# bash install.sh 2842d6d7-f7a1-fb97-254d-9be972403dd0
start to create config file...
Zabbix管理地址: http://192.168.88.201
Zabbix管理员账号: Admin
Zabbix管理员密码: zabbix
......
```

补充：如果安装时，秘钥填入错误，也可以对其卸载？

```powershell
cd /usr/lib/zabbix/alertscripts/cloudalert/bin/
bash uninstall.sh
```

## 验证安装

配置完onealert后，我们可以验证下它安装后到底对zabbix做了啥。简单来说，它做了三件事:

1. 增加了一个报警动作
2. 增加了一个用户和一个用户组用于报警
3. 增加了一个报警媒介类型

### 验证动作

![image-20250308010242235](./images/image-20250308010242235.png)

确认以上信息，合起来表示：当触发器发生状态变化时，通知信息通过此动作发送消息给cloudalert用户。

### 验证用户与用户群组

![image-20250308010609863](./images/image-20250308010609863.png)

### 验证报警媒介

![image-20250308010732247](./images/image-20250308010732247.png)



### 验证报警脚本

以下脚本看不懂没关系，我们只要知道是对接报警平台的API接口就OK了

~~~powershell
[root@server bin]# cat /usr/lib/zabbix/alertscripts/cloudalert/bin/alert.sh
#!/bin/bash
# PATH
DIR="$( cd "$( dirname "$0"  )" && pwd  )"
echo $DIR
source $DIR/log.sh
$(log INFO ZabbixActionParams "$3")
r=`curl -H "Content-Type:application/json"  -X POST -d "$3" http://api.aiops.com/alert/api/event/zabbix/v3`
$(log INFO ItsmAlertResponse "$r")
~~~



把以上验证的内容再连成一条复习一下:

**监控项** --》**图形** --》**触发器** --》**动作** --》**用户** --》**报警媒介** --》**报警脚本** --》**报警平台**



## 配置通知策略

在报警平台按需求配置通知策略(过程省略)

我这里主要配置的是任务时间任务报警立刻发送到我所绑定的邮箱，微信，手机短信，手机电话。

![1579239739000](images/zabbix79.png)

## 触发器触发报警

这里以前面配置过的"**登录用户数大于20个**"这个触发器来测试报警.

**注意:** ==要触发器有状态变化才能报警==。

如果在测试前就已经大于20个了是不会报警的，需要先降到20以下，再升到20以上让其触发。

![1579247441452](images/zabbix80.png)



![1579248035419](images/zabbix81.png)





# 十一、自动化批量监控

我们要监控的服务器数量很大的情况下，如何批量操作:

* 系统使用cobbler批量安装
* zabbix-agent安装与配置可以使用cobbler的postscript脚本实现，或者使用ansible来实现
* 监控主机元素: 监控项，图形，触发器等，统一使用模板



因为创建监控主机和添加模板都需要web界面操作，如何自动批量做?

答案: 自动发现或自动注册。



## 自动发现或自动注册

**自动发现:** 由Zabbix Server开启发现进程，每隔一段时间扫描网络中符合条件的主机。

**自动注册:** 与自动发现相反由Zabbix agent去找Server注册。

所以大家看到，和前面提过的主动监控与被动监控的概念很类似。



**自动发现案例:**

前面早就准备了一台agent2，一直还没使用，这里就尝试自动发现这台agent2，并通过动作将其创建为监控主机并添加模板。

### 1, agent2上安装zabbix-agent

~~~powershell
[root@agent2 ~]# rpm -Uvh https://repo.zabbix.com/zabbix/7.0/alma/9/x86_64/zabbix-release-7.0-2.el9.noarch.rpm
[root@agent2 ~]# dnf install zabbix-agent -y
~~~

### 2, 配置agent端并启动服务

配置`/etc/zabbix/zabbix_agentd.conf`配置文件，配置结果如下:

~~~powershell
[root@agent2 ~]# egrep -vn '^#|^$' /etc/zabbix_agentd.conf
13:PidFile=/var/run/zabbix/zabbix_agentd.pid
32:LogFile=/var/log/zabbix/zabbix_agentd.log
43:LogFileSize=0
98:Server=192.168.88.201						# 修改为Zabbix Server服务器IP地址
139:ServerActive=192.168.88.201		    # 修改为Zabbix Server服务器IP地址
150:Hostname=agent2.itcast.cn		    # 修改为Zabbix Agent所在主机名称（必须和hostname保持一致）
~~~

~~~powershell
[root@agent2 ~]# systemctl restart zabbix-agent
[root@agent2 ~]# systemctl enable zabbix-agent

[root@agent2 ~]# netstat -ntlup |grep :10050
tcp     0  0 0.0.0.0:10050      0.0.0.0:*       LISTEN      20447/zabbix_agentd
tcp6    0  0 :::10050           :::*            LISTEN      20447/zabbix_agentd
~~~

**再次说明: 在自动化运维体系里可以使用cobbler的postscript脚本或ansible来批量做以上2步**



### 3, 配置并启用自动发现规则

数据采集 =》自动发现

![image-20250308223011545](images/image-20250308223011545.png)

![image-20250308224005188](images/image-20250308224005188.png)

![image-20250308224123267](images/image-20250308224123267.png)

### 4, 确认自动发现到主机

监测 =》自动发现

![image-20250308224324549](images/image-20250308224324549.png)

### 5, 配置动作实现自动监控

告警 =》 动作 =》 发现动作 =》 点击 "Auto discovery. Linux servers."

![1579262645161](images/zabbix87-17414806229561.png)

![1579263327305](images/zabbix88-17414806229562.png)

![1579263535785](images/zabbix89-17414806229573.png)

![1579263637401](images/zabbix90-17414806229574.png)

### 6, 确认动作更新并启用

![1579263687621](images/zabbix91-17414806229575.png)

### 7, 验证最终效果

确认时间同步, 需要耐心等待一段时间。(可能几分钟到十几分钟)

最终效果如下:

![1579263963810](images/zabbix92-17414806229576.png)



**问题:** agent2上的"登录用户数"这个监控项的图形上没有数据, 为什么? 如何解决?

答：因为登录用户数，需要在zabbix_agentd.conf中添加key！！！



小结：

① 什么是自动发现？所谓的自动发现就是Zabbix Server主动发现安装了Zabbix Agent节点 => 网段

② 自动发现实施步骤

第一步：数据采集菜单 => 自动发现 => 设置自动发现网段信息192.168.88.1-254

第二步：配置动作 =》 告警 =》 动作 =》 发现动作（找到了这台机器，对它做啥）

![image-20250309150106147](images/image-20250309150106147.png)

![image-20250309150134238](images/image-20250309150134238.png)

第三步：返回，监测菜单 =》 主机 =》 可以发现新主机信息

![image-20250309150256725](images/image-20250309150256725.png)

**自动注册就不再演示了，仅了解即可。**



## 批量操作

把大量的服务器实现了自动监控后，后续还可能会做一些相关的批量操作，如:

* 批量启用主机
* 批量禁用主机
* 批量删除主机

数据采集 =》 主机，如下图所示：

![1579265383260](images/zabbix93-17414806229577.png)

**说明:**

*  因为我们建议使用模板来管理监控，所以批量更新功能也可以直接更新模板即可
*  导出功能在当前版本经测试只能导出单个主机的配置信息为`.yaml/.xml/.json`格式文件

## 模板导入导出

辛苦配置好的模板或主机，如果被误删除了怎么办? 或者我想搭建多个zabbix服务器，那么又要辛苦再配置一遍？

解决方法就是把配置的模板或主机导出成`.yaml/.xml/.json`格式文件，主要有两大好处:

* **备份**(防止误删除)
* **迁移**(导出后, 导入到另一个服务器)



数据采集 =》 模板：

![1579266298631](images/zabbix94-17414806229578.png)



![1579266464078](images/zabbix95-17414806229579.png)

![1579267456135](images/zabbix96-174148062295710.png)

![1579267742833](images/zabbix97-174148062295711.png)



![1579267805127](images/zabbix98-174148062295712.png)



# 十二、zabbix代理

## zabbix proxy应用场景

参考: https://www.zabbix.com/documentation/current/manual/distributed_monitoring/proxies



![1579329775005](images/zabbix99-174148062295714.png)

**应用场景1: 跨内外网监控**

当zabbix server与被监控机器不在同一个机房时,跨公网监控会很麻烦, 也会带来安全隐患

* 比如有防火墙的情况,需要防火墙开放的端口增多
* 像mysql数据库这类应用是不适合直接被公网连接的



**应用场景2: 分布式监控**

当监控机主机特别多,甚至分散在不同的地域机房。这个时候zabbix server压力很大，所以可以通过增加zabbix proxy来代理收集每个机房里的主机信息，再统一给zabbix server.

![1579329871119](images/zabbix100-174148062295713.png)

## zabbix proxy案例

**环境准备:**



![1579331331190](images/zabbix101-174148062295715.png)

1, **新增一台全新环境的服务器做proxy，修改主机名**

~~~powershell
[root@proxy ~]# hostnamectl set-hostname --static proxy
~~~

> 在代理服务器位置，主机名称特别重要，一定要明确是proxy还是proxy.itcast.cn

2, **==四台服务器全部==重新绑定主机名**

~~~powershell
# vim /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.88.201  server
192.168.88.202  agent1
192.168.88.203  agent2
192.168.88.204  proxy
~~~

3, 确认关闭防火墙,selinux

4, 确认时间同步



**操作步骤:**

### 1, 在proxy上安装软件包

~~~powershell
[root@proxy ~]# rpm -Uvh https://repo.zabbix.com/zabbix/7.0/centos/9/x86_64/zabbix-release-latest-7.0.el9.noarch.rpm
注意：如果/etc/yum.repos.d目录存在epel.repo文件，必须通过记事本打开，把enabled=1参数设置为enabled=0，然后在安装！
[root@proxy ~]# yum install mariadb-server zabbix-proxy-mysql zabbix-sql-scripts zabbix-selinux-policy zabbix-agent -y
~~~

### 2, 启动数据库并建库授权

~~~powershell
[root@proxy ~]# systemctl restart mariadb
[root@proxy ~]# systemctl enable mariadb

[root@proxy ~]# mysql

MariaDB [(none)]> create database zabbix_proxy default charset utf8;
MariaDB [(none)]> grant all privileges on zabbix_proxy.* to 'zabbix'@'localhost' identified by '123';
MariaDB [(none)]> flush privileges;
MariaDB [(none)]> quit
~~~

### 3, 导入sql数据

```powershell
cat /usr/share/zabbix-sql-scripts/mysql/proxy.sql | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix_proxy
```

### 4, 修改proxy端配置并启动服务（proxy默认主动推送数据到server端）

~~~powershell
[root@proxy ~]# egrep -vn '^#|^$' /etc/zabbix/zabbix_proxy.conf
32:Server=192.168.88.201						# 修改为zabbix服务器的ip
42:Hostname=proxy							    	# 修改为本代理服务器的主机名
84:LogFile=/var/log/zabbix/zabbix_proxy.log
95:LogFileSize=0
136:PidFile=/var/run/zabbix/zabbix_proxy.pid
146:SocketDir=/var/run/zabbix
173:DBName=zabbix_proxy
188:DBUser=zabbix
197:DBPassword=123								  # 打开注释并修改为连接数据库的密码,和上面授权对应
300:ProxyConfigFrequency=60		 		# proxy多久从server接收一次配置数据(打开注释并修改)
310:DataSenderFrequency=5		 		 # proxy多久发送一次收集的数据给server(打开注释并修改)
490:SNMPTrapperFile=/var/log/snmptrap/snmptrap.log
566:Timeout=4
653:LogSlowQueries=3000
759:StatsAllowedIP=127.0.0.1

[root@proxy ~]# systemctl restart zabbix-proxy
[root@proxy ~]# systemctl enable zabbix-proxy
~~~

**说明:** `ProxyConfigFrequency=60`和`DataSenderFrequency=5`这两个参数需要配置，否则最终结果会很久都看不到数据。



### 5, 配置agent端

除了agent1和agent2之外,server和proxy也可以被监控, 也就是说一共4台都可以被监控。

这4台都可以被proxy监控，也可以被server监控。本实验我选择以下方案:

* server监控自己(默认不变), proxy,agent1,agent2都被proxy监控，然后将数据给server

**所以==proxy,agent1,agent2这3台都做如下操作==:**

~~~powershell
# egrep -vn '^#|^$' /etc/zabbix/zabbix_agentd.conf
115:Server=192.168.88.204					# 修改为proxy的IP,而不是server的IP
169:ServerActive=192.168.88.204				# 修改为proxy的IP,而不是server的IP
180:Hostname=XXX							# 主机名分别为proxy,agent1,agent2

347:UserParameter=loginusers,who | wc -l	# 都加上此自定义监控项

# systemctl restart zabbix-agent
~~~

> 注：agent1,agent2以上文件如果找不到，就代表新版本迁移了配置文件位置，新位置：vim /etc/zabbix_agentd.conf



### 6, 创建proxy为被监控主机

![1579339965115](images/zabbix102-174148062295716.png)



![image-20250309163801000](images/image-20250309163801000.png)

![1579340398884](images/zabbix104-174148062295717.png)

### 7, 为3台被代理机添加模板

数据采集 =》 主机：

![1579340647921](images/zabbix105-174148062295718.png)



![1579340731476](images/zabbix106-174148062295719.png)

![1579341091375](images/zabbix107-174148062295720.png)

### 8, 创建proxy为代理

管理 =》 Proxy：

![image-20250309173037809](images/image-20250309173037809.png)



![image-20250309173331156](images/image-20250309173331156.png)

### 9, 批量更新代理

![1579343844743](images/zabbix110-174148062295723.png)

![image-20250309004657201](images/image-20250309004657201.png)

![1579344314901](images/zabbix112-174148062295824.png)

### 10, 验证

在被代理监控机上(agent1,agent2,proxy)做信息的改变, 比如改变登录用户数，然后在web管理界面的图形上能查看到相应变化，则表示代理一切OK。

过程省略, 请自行验证.



# 十三、主动监控与被动监控

**一共有4种模式:**

1. agent主动
2. agent被动(默认)
3. proxy主动(默认)
4. proxy被动



## agent被动

相对于agent的被动,也就是表示是server或proxy去找agent拿数据。

~~~powershell
# grep -n ^Server= /etc/zabbix_agentd.conf
Server=192.168.88.204						 agent被动模式, IP为server或proxy的IP
~~~

## agent主动

相对于agent的主动, 也就是表示是agent主动把数据传给server或proxy

~~~powershell
# grep -n ^ServerActive= /etc/zabbix_agentd.conf
ServerActive=192.168.88.204			     agent主动模式, IP为server或proxy的IP
~~~

**优点:**  当agent太多的情况下, server或proxy去找这么多agent搜集数据, 压力负载过大。用agent主动模式就可以缓解server或proxy的压力。

但用主动模式的问题是: 监控项也要转为主动式.

数据采集 =》模板 =》 监控项

![image-20250309180129269](images/image-20250309180129269.png)



![1579355081914](images/zabbix129-174148062295826.png)



![1579355176776](images/zabbix130-174148062295827.png)

![1579355227441](images/zabbix131-174148062295828.png)



## proxy主动与被动  

由`/etc/zabbix_proxy.conf`里的`ProxyMode`参数决定。

![1579355540387](images/zabbix132-174148062295829.png)

## 结论

默认情况下agent被动监控，监控方向如下图所示:



![1579353877666](images/zabbix126-174148062295830.png)

上图中:

* 由server找agent拿数据
* 这种情况server端压力较大, agent端压力较小



![1579353682697](images/zabbix127-174148062295831.png)

上图中:

* proxy找agent拿数据,又主动将agent的数据提交给server
* proxy压力最大



个人推荐默认的模式，完全不用修改。请讨论或思考为什么? 什么情况才有可能需要修改模式?



# 十四、web监测(拓展补充)

**web监测**: 类似一个 (可以包含多个小监控项),主要针对web服务器做监控场景（httpd、nginx）。

可以对一个url页面进行监测（监测它的状态码,页面匹配的字符串,响应时间,下载速度等）

![Snipaste_2025-10-10_17-48-13](images\Snipaste_2025-10-10_17-48-13.png)

## 1, 在agent1上创建web监测

![image-20250309181346021](images/image-20250309181346021.png)

## 2, 创建web场景

![1579348141193](images/zabbix114-174148062295833.png)



![1579348558182](images/zabbix115-174148062295834.png)

## 3, 添加步骤一

![1579349008676](images/zabbix116-174148062295835.png)



![image-20250309181708874](images/image-20250309181708874.png)

![image-20250309181835282](images/image-20250309181835282.png)



## 4, 添加步骤二

![1579350272676](images/zabbix119-174148062295838.png)



![image-20250309182002106](images/image-20250309182002106.png)



![image-20250309182118018](images/image-20250309182118018.png)



![1579350860502](images/zabbix122-174148062295841.png)

## 5, 验证步骤一

![image-20250309182318409](images/image-20250309182318409.png)

去agent1上安装httpd,创建主页,并启动服务

~~~powershell
[root@agent1 ~]# yum install httpd -y
[root@agent1 ~]# echo web1 > /var/www/html/index.html
[root@agent1 ~]# systemctl restart httpd
[root@agent1 ~]# systemctl enable httpd
~~~

再次验证

![1579351662610](images/zabbix124-174148062295843.png)

## 6, 验证步骤二

~~~powershell
[root@agent1 ~]# echo "welcome to itheima yunwei" > /var/www/html/test.txt
~~~

![1579352267490](images/zabbix125-174148062295844.png)



# 作业

## 监控系统

系统有4大子系统: CPU, 内存, 磁盘IO, 网络。除了这4大子系统外还有进程, 登录用户等等。

请有实力的同学在以下题目基础上做自由拓展。

要求：确认CPU可以监控哪些指标？内存可以监控哪些指标？磁盘IO可以监控哪些？网络可以监控哪些？

```
1、确认CPU可以监控哪些指标？
答：cpu利用率、负载（system.cpu.load[percpu,avg5]）、空闲时间（system.cpu.util[,idle]）、中断数（system.cpu.intr）、上下文切换率（system.cpu.switches）等指标。

2、内存可以监控哪些指标？
答：内存使用率（vm.memory.size[total]）、可用内存（vm.memory.size[available]）、缓存、交换区（Swap）使用情况（system.swap.size[total]） 等

3、磁盘IO可以监控哪些？
I/O 读写速率、I/O 操作次数、I/O 等待时间（vfs.dev.read.await[sda]）、I/O 设备利用率（vfs.dev.util[sda]）

4、网络可以监控哪些？
带宽使用率（net.if.in[eth0]）、收发速率、丢包率、错误、TCP 活跃连接数（net.tcp.established）

```



1, 监控所有进程数量,并设定触发器(当大于200就警告，当大于300就严重警告，超过400个就灾难）

```powershell
proc.num
```

2, 监控tcp连接数量, 并自定义触发器

```powershell
net.tcp.established
```

3, 监控某分区磁盘使用率，并自定义触发器

```powershell
vfs.fs.size[<fsname>,<mode>]
<fsname>	分区名称，例如 /, /data, /home
<mode>	total, used, free, pfree, pused 等
```

4, 监控可用内存，并自定义触发器

```powershell
vm.memory.size[available]
```



## 监控nginx

在前面讲模板章节中有提到zabbix4版本中有自带的nginx模板，如下图所示:

![1579357686547](images/zabbix133-174148062295845.png)



**不想用自带模板的, 也可以参考以下方式自定义监控nginx：**



nginx有一个状态页，通过查看状态页信息可以连接到nginx服务负载情况.

下面我们假设监控agent1的nginx

1,在agent1上安装nginx

~~~powershell
[root@agent1 ~]# yum install epel-release
[root@agent1 ~]# yum install nginx
~~~

2,在nginx里的server{}配置段里加上下面一段，然后重启服务

~~~powershell
[root@agent1 ~]# vim /etc/nginx/nginx.conf

		location /status {
                stub_status on;
                allow 127.0.0.1; # 允许本机访问,因为下一步脚本是在本机，通过127.0.0.1来得到状态信息的
                allow 10.1.1.1;	 # 加这个IP是为了windows宿主机访问用的，方便浏览器测试用的(可以不加这句)
                deny all;
                access_log off;
        } 
        
[root@agent1 ~]# systemctl restart nginx
[root@agent1 ~]# systemctl enable nginx
~~~

3, 通过浏览器访问http://192.168.88.102/status就能看到如下nginx状态信息

~~~powershell
Active connections: 1 
server accepts handled requests
 59 59 115 
Reading: 0 Writing: 1 Waiting: 0 

Active  connections：当前所有处于打开状态的活动连接数
accepts ：已经接收连接数
handled ： 已经处理过的连接数
requests ： 已经处理过的请求数，在保持连接模式下，请求数量可能会大于连接数量

Reading: 正处于接收请求的连接数
Writing: 请求已经接收完成，处于响应过程的连接数
Waiting : 保持连接模式，处于活动状态的连接数
~~~

4, 在agent1上准备一个脚本,并给执行权限

~~~powershell
[root@agent1 ~]# vim /opt/nginx_status.sh
#!/bin/bash

HOST="127.0.0.1"
PORT="80"

function ping {						# 这个不是ping，是判断nginx进程是否存在
    /sbin/pidof nginx | wc -l
}

function active {
    /usr/bin/curl "http://$HOST:$PORT/status/" 2>/dev/null| grep 'Active' | awk '{print $NF}'
}
function accepts {
    /usr/bin/curl "http://$HOST:$PORT/status/" 2>/dev/null| awk NR==3 | awk '{print $1}'
}
function handled {
    /usr/bin/curl "http://$HOST:$PORT/status/" 2>/dev/null| awk NR==3 | awk '{print $2}'
}
function requests {
    /usr/bin/curl "http://$HOST:$PORT/status/" 2>/dev/null| awk NR==3 | awk '{print $3}'
}
function reading {
    /usr/bin/curl "http://$HOST:$PORT/status/" 2>/dev/null| grep 'Reading' | awk '{print $2}'
}
function writing {
    /usr/bin/curl "http://$HOST:$PORT/status/" 2>/dev/null| grep 'Writing' | awk '{print $4}'
}
function waiting {
    /usr/bin/curl "http://$HOST:$PORT/status/" 2>/dev/null| grep 'Waiting' | awk '{print $6}'
}
$1

[root@agent1 ~]# chmod 755 /opt/nginx_status.sh 
~~~

5, 在agent1上定义UserParameter，并重启服务

~~~powershell
在配置文件里加上下面一句
[root@agent1 ~]# vim /etc/zabbix/zabbix_agentd.conf 
UserParameter=nginx_status[*],/opt/nginx_status.sh $1

[root@agent1 ~]# systemctl restart zabbix-agent
~~~

6, 在server上(如果使用了使用proxy则这里就在proxy上操作)zabbix_get测试

~~~powershell
[root@proxy ~]# yum install zabbix-get -y

[root@proxy ~]# zabbix_get -s 192.168.88.102 -k nginx_status[ping]
1
[root@proxy ~]# zabbix_get -s 192.168.88.102 -k nginx_status[handled]
76
~~~

7, 测试能成功监控取到值，说明监控OK。



说明: web管理界面添加监控项的过程请自行完成, 这里省略。



## 监控mariadb

数据库能做监控项的基本都在`show status`命令里



例: 自定义监控agent1上mariadb的当前登录用户数, 并设定触发器(当大于50个就警告)

~~~powershell
[root@agent1 ~]# yum install mariadb-server -y
[root@agent1 ~]# systemctl restart mariadb
~~~

方法一:

下面这条命令就可以得到当前登录用户数，然后自定义一个UserParameter就可以了

~~~powershell
[root@agent1 ~]# mysqladmin extended-status |grep Threads_connected |awk '{print $4}'
~~~



方法二:

~~~powershell
[root@agent1 ~]# vim /etc/zabbix/zabbix_agentd.conf

UserParameter=mysql.status[*],echo "show global status where Variable_name='$1';" | mysql -N | awk '{print $$2}'			# 这里显示方式有点特殊，需要$$2而不是$2

[root@agent1 ~]# systemctl restart zabbix-agent
~~~

**说明:** 这句配置在zabbix3版本里`/etc/zabbix/zabbix_agentd.d/userparameter_mysql.conf`配置文件默认自带,zabbix4自带的参数不能直接对mariadb使用了,所以我们手工再加上



在server或proxy上验证，`show status`命令里的理论上都可以验证

~~~powershell
[root@proxy ~]# zabbix_get -s 192.168.88.102 -k mysql.status[Threads_connected]

[root@proxy ~]# zabbix_get -s 192.168.88.102 -k mysql.status[uptime]
~~~

对于新版本，在zabbix7.0在centos9中安装，并没有生成/var/lib/zabbix目录（坑），这里需要我们手动创建目录和密码文件

```
mkdir -p /var/lib/zabbix
chown zabbix:zabbix /var/lib/zabbix
chmod 700 /var/lib/zabbix

vi /var/lib/zabbix/.my.cnf
[client]
user=zabbix
password=123

配置完成后，重启zabbix-agent
```

面试中zabbix监控mysql的指标：

```
1、在生产环境下，zabbix监控mysql，通常需要监控那些具体的指标？---->chatGpt
2、结合mysql.status[*],echo "show global status where Variable_name='$1';" | mysql -N | awk '{print $$2}'，自定义键值，获取慢查询数量、获取缓存命中率。获取最大连接数？

```





## 综合场景练习

请通过上网查资料, 设计一个监控场景自由发挥.

思路步骤:

* 规划要监控的主机与业务
* 按照监控项的类型创建不同的自定义模板
* 在自定义模板中配置监控项,图形与触发器等
* 实现报警

* 配置自动发现与动作实现自动监控新主机并添加模板
* 导出模板实现备份

更大规模架构监控:

* 多台zabbix server分担主机进行监控
* 使用proxy分担server压力
* 结合cobbler和ansible实现全自动化监控



答疑：

疑问1：课堂讲CentOS Stream 9/MySQL8.0，工作中用CentOS7或者MySQL5.7怎么办？

答案：技术朝前，CentOS 7的官方维护服务已于2024年6月30日结束，这意味着该系统将不再接收任何更新、补丁或安全修复 => 等保3.0，替换版本 => CentOS Stream 9/Rocky Linux监控CentOS8、CentOS9/Ubuntu

CentOS7 => CentOS8、CentOS9 => （firewall-cmd、firewalld）=> (yum => dnf) => (python2.7 => python3.9)

CentOS6.5 => CentOS7.6

---

MySQL5.7 => MySQL8.0

主从架构、逻辑备份、物理备份几乎没有区别！

MHA => MGR、克隆复制 => 3期有一个女孩叫刘欣 => DBA => 老版本系统MySQL5.7上，新项目几乎MySQL8.0

Java/大数据 => 深度MySQL使用学科 => 中州养老（MySQL8.0）、大数据所有项目（MySQL8.0）

MySQL8性能上提升比较大，MySQL8增加了很多利于分析的函数比如窗口函数等等！

---

同学简历投了一些公司，昨天郑州 => 实施 => 7-10k

自我介绍 + 项目介绍（具体介绍项目大致做的啥，用了哪些技术，团队规模，你负责的工作点）

总共35分钟，10分钟，大部分时间聊天、怎么看待加班、出差、你自己为什么值这么多钱。

Linux主要做了哪些？

MySQL/Oracle，会不会基本SQL操作，增删改查，问了左外连接和右外连接区别

MySQL主从工作原理

MySQL服务器如果负载过高，可能原因，解决方案

在公司大致工作流程

上家公司工资组成：基本 + 绩效 + 年终（绩效和年终）

---

大家比较懒，晚自习，9点40打扫卫生回家

总结项目文档 => 自我介绍 + 项目介绍 + 常见问题（最好分类：Linux命令 + MySQL备份、主从、高可用）+  Zabbix/Prometheus（具体监控啥，系统监控哪些 + 应用监控哪些）

玩手机 => 下课、中午午休、晚自习之前

CentOS Stream 9

MySQL5.7 / MySQL8.0，学会自己扩展

---

1个月 => 简历 + 项目文档（印象比较深刻问题），项目尽量表达流畅一些 + 常见面试题汇总

把记不住记录在项目文档 => 周四晚上
