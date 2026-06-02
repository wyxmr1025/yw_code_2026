# 学习目标

- [ ] 能够说出docker容器和虚拟机的主要区别

- [ ] 能够说出docker用到的内核技术

- [ ] 能够安装部署docker

- [ ] 掌握docker镜像的常见操作

- [ ] 掌握镜像仓库的搭建与使用

- [ ] 掌握常见的容器操作命令

- [ ] 能够使用docker运行基础应用

- [ ] 能够使用dockerfile构建容器镜像（类似playbook）

- [ ] 能够通过link连接容器（容器之间可以相互通信）

- [ ] docker数据可视化（docker的web管理平台）

  

# PaaS介绍

云计算 = 公有云 + 私有云 + 混合云 + 容器云

公有云：阿里云、华为云、百度云

私有云：OpenStack（底层）

混合云：公有云 + 私有云

容器云：Docker/Containerd + K8S

做云的优势：提高资源利用率,将资源打包做成服务给用户使用,资源提供商极少需要与用户交互打交道。

Redis集群：阿里云，都是直接搭建好的，直接调用即可。

![](./images/%E4%BA%91%E8%AE%A1%E7%AE%97%E5%B1%82.jpg)

IaaS（基础即服务） =>  OpenStack（搭建云平台），阿里云、华为云，物理层、网络层 => 云平台基础层（底层）

PaaS（系统即服务）=>  基于操作系统构建应用，搭建一个项目，需要1个软件1个软件配置 => ECS

SaaS（软件即服务）=>  用户不需要1个软件1个软件配置，提供了一套完整解决方案，只需要登录就可以直接使用 => RDS

FC函数服务 => 主流大模型，如DeepSeek

---

由于hypervisor虚拟化技术仍然存在一些性能和资源使用效率方面的问题，因此出现了一种称为容器技术（Container）的新型虚拟化技术来帮助解决这些问题。



容器: 是PaaS的一种实现,相对于虚拟机来说有更好的性能，更高的资源利用率。



# 一、认识容器技术

在生活中，瓶子，罐子，盆，试管，缸等都是用来装东西的容器。

![1545141320499](./images/%E7%94%9F%E6%B4%BB%E4%B8%AD%E7%9A%84%E5%AE%B9%E5%99%A8.png)

![1545143022786](./images/%E9%9B%86%E8%A3%85%E7%AE%B1.png)

在集装箱没有被使用以前，海上运输货物效率不高（货物大小与形状不一)。有了集装箱后，货物可以统一规格来存放与运输了，极大地提高了效率。

在IT技术中:

虚拟化技术可以在宿主机上安装多个不同的操作系统，运行多套不同的应用。但可能就是为了运行一个nginx,却还要在虚拟机里运行一个完整的操作系统,内核和其它无关程序，这种做法资源利用不高。

**所以我们希望更多的关注应用程序本身,而不再分精力去关注操作系统与无关程序,操作系统内核直接与宿主机共享** 



Linux容器技术是一种**==轻量级==**的虚拟化技术。主要特点有:

1. 轻量:只打包了需要的bins/libs(也就是命令和库文件)。与宿主机共享操作系统,直接使用宿主机的内核.
2. 部署快: 容器的镜像相对虚拟机的镜像小。部署速度非常快，秒级部署
3. 移植性好: Build once,Run anywhere(一次构建,随处部署运行)。  build,ship,run
4. 资源利用率更高: 相对于虚拟机，不需要安装操作系统，所以几乎没有额外的CPU,内存消耗



![1545191782393](./images/%E8%99%9A%E6%8B%9F%E5%8C%96%E4%B8%8E%E5%AE%B9%E5%99%A8%E5%AF%B9%E6%AF%94.png)

面试题：Docker容器 与 传统虚拟化（VM）之间有什么区别？（记住）

![1563882676718](./images/%E5%AE%B9%E5%99%A8%E4%B8%8E%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%AF%B9%E6%AF%94%E5%9B%BE.png)

Docker应用场景：部署各种各样应用，让我们实现秒级部署、迁移性比较强、性能几乎无额外损失！



# 二、docker介绍

![1545199633720](./images/docker%E5%9B%BE%E6%A0%87.png)



docker就是目前最火热的能实现容器技术的软件,使用go(golang)语言开发。

参考:https://www.docker.com/



## Docker版本

**2017之前版本**

1.7,1.8,1.9,1.10,1.11,1.12, 1.13

**2017年的3月1号之后**，Docker的版本命名开始发生变化，同时将CE版本和EE版本进行分开。

Docker社区版（CE）：为了开发人员或小团队创建基于容器的应用,与团队成员分享和自动化的开发管道。docker-ce提供了简单的安装和快速的安装，以便可以立即开始开发。docker-ce集成和优化，基础设施。 

- 17-03-ce
- 17-06-ce
- 18-03-ce
- 18-06-ce
- 18-09-ce
- 19-03-ce
- 最新版本：28.0.1-ce（这是目前Docker社区版的最新版本，提供了更新的特性、性能和修复）。

Docker企业版（EE）：专为企业的发展和IT团队建立。docker-ee为企业提供最安全的容器平台，以应用为中心的平台。





## docker用到的内核技术

dockerA => Nginx

dockerB => Tomcat

相对独立，默认不会互相影响？

docker容器本质上是宿主机的**==进程==**. 可以把docker容器内部跑的进程看作是宿主机的线程。

Docker通过==namespace实现了资源隔离==，通过==cgroups实现了资源限制==



### NameSpace

Linux内核实现namespace的一个主要目的就是实现轻量级虚拟化(容器)服务。在同一个namespace下的进程可以感知彼此的变化，而对外界的进程一无所知。

Linux 在很早的版本中就实现了部分的 namespace,比如内核 2.4 就实现了mount namespace。

大多数的namespace支持是在内核 2.6 中完成的，比如 IPC、Network、PID、和 UTS。还有个别的namespace 比较特殊，比如User，从内核 2.6 就开始实现了，但在内核 3.8 中才宣布完成。

同时，随着 Linux 自身的发展以及容器技术持续发展带来的需求，也会有新的 namespace 被支持，比如在内核 4.6 中就添加了Cgroup namespace。

inux内核提拱了6种namespace隔离的系统调用

| namespace | 系统调用参数  | 隔离内容                            |
| --------- | ------------- | ----------------------------------- |
| UTS       | CLONE_NEWUTS  | 主机名或域名                        |
| IPC       | CLONE_NEWIPC  | 信号量、消息队列和共享内存          |
| PID       | CLONE_NEWPID  | 进程编号                            |
| net       | CLONE_NEWNET  | 网络设备接口,IP路由表、防火墙规则等 |
| mount     | CLONE_NEWNS   | 挂载点(文件系统)                    |
| user      | CLONE_NEWUSER | 用户和用户组                        |

UTS: 	    每个NameSpace都拥有独立的主机名或域名，可以把每个NameSpace认为一个独立主机。

IPC: 	     每个容器依旧使用linux内核中进程交互的方法，实现进程间通信

PID:	      每个容器都拥有独立的进程树，而容器是物理机中的一个进程，所以容器中的进程是物理机的线程

Net:	     每个容器的网络是隔离

Mount:	每个容器的文件系统是独立的

User:	   每个容器的用户和组ID是隔离，每个容器都拥有root用户



小结: 应用程序运行在一个隔离的空间(namespace)内, 每个隔离的空间都拥有独立的UTS,IPC,PID,Net,Mount,User.



### Control Group

控制组（CGroups）是Linux内核的一个特性，**主要用来对共享资源进行隔离、限制、审计等**。

只有能控制分配到容器的资源，才能避免多个容器同时运行时对宿主机系统的资源竞争。

控制组可以提供对容器的内存、CPU、磁盘IO等资源进行限制和计费管理。



案例可参考优化课程文档



### LXC与Docker区别

LXC为Linux Container的简写。可以提供轻量级的虚拟化.

==Docker的底层就是使用了LXC来实现的。Docker以LXC为基础，实现了更多更强的功能==。

资源隔离：NAMESPACE

资源限制：CGroupS



前面内容小结:

* 容器属于typeIII虚拟化,属于PaaS
* 容器是一种轻量级，进程级的虚拟机
* 相比于虚拟机的优势
  * 不需要安装OS，和宿主机共享
  * 镜像存储空间小
  * 启动速度快(容器为秒级,虚拟机一般需要10秒左右)
  * 移植性更好,更轻便
  * 性能更好
* docker是一个实现容器的软件，底层使用LXC
* docker主要使用namespace命名空间技术实现资源隔离,使用cgroups实现资源限制







# 三、docker环境准备

普及

宿主机：简单来说，就是运行了Docker程序的这个操作系统就是宿主机。

如果在Windows操作系统中运行Docker，则Windows就是一个宿主机

如果在Linux操作系统运行Docker，则Linux就是一个宿主机

---

建议直接在宿主机上跑docker（当然也可以在虚拟机里跑docker)

不能直接在windows上跑docker(因为namespace,cgroups是linux内核的特性,windows没有,所以需要在windows跑linux虚拟机,再跑docker)

1. 要求能访问公网
2. 关闭防火墙,selinux

注意：克隆VMware虚拟机要记得更改mac地址



## docker软件安装

**docker-ce的yum源下载(任选其一)**

* 下载docker官方ce版（国外服务器）

~~~powershell
[root@daniel ~]# wget https://download.docker.com/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
~~~

* 或者使用aliyun的docker-ce源（中国服务器）

~~~powershell
[root@daniel ~]# yum install wget -y
[root@daniel ~]# wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo

yum仓库路径：/etc/yum.repos.d/
~~~





**docker安装**

~~~powershell
[root@daniel ~]# yum clean all
[root@daniel ~]# yum install docker-ce -y
~~~

PS: 注意要安装docker-ce版,不要安装docker(否则可能安装1.13老版本)

**启动服务**

~~~powershell
[root@daniel ~]# systemctl start docker
[root@daniel ~]# systemctl enable docker
[root@daniel ~]# systemctl status docker
~~~

**查看版本信息**

~~~powershell
[root@daniel ~]# docker -v
Docker version 28.0.1, build 068a01e
[root@daniel ~]# docker info
Client: Docker Engine - Community
 Version:    28.0.1
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.21.1
    Path:     /usr/libexec/docker/cli-plugins/docker-buildx
  compose: Docker Compose (Docker Inc.)
    Version:  v2.33.1
    Path:     /usr/libexec/docker/cli-plugins/docker-compose

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 28.0.1
 Storage Driver: overlay2
  Backing Filesystem: xfs
  Supports d_type: true
  Using metacopy: false
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: systemd
 Cgroup Version: 2
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: bcc810d6b9066471b0b6fa75f557a15a1cbf31bb
 runc version: v1.2.4-0-g6c52b3f
 init version: de40ad0
 Security Options:
  seccomp
   Profile: builtin
  cgroupns
 Kernel Version: 5.14.0-511.el9.x86_64
 Operating System: CentOS Stream 9
 OSType: linux
 Architecture: x86_64
 CPUs: 4
 Total Memory: 7.473GiB
 Name: docker01.itcast.cn
 ID: c683ddb4-f2eb-4d28-a54f-30f5df923ced
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Experimental: false
 Insecure Registries:
  ::1/128
  127.0.0.0/8
 Live Restore Enabled: false
~~~

~~~powershell
[root@daniel ~]# docker version
Client: Docker Engine - Community
 Version:           28.0.1
 API version:       1.48
 Go version:        go1.23.6
 Git commit:        068a01e
 Built:             Wed Feb 26 10:42:23 2025
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          28.0.1
  API version:      1.48 (minimum version 1.24)
  Go version:       go1.23.6
  Git commit:       bbd0a17
  Built:            Wed Feb 26 10:40:43 2025
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.7.25
  GitCommit:        bcc810d6b9066471b0b6fa75f557a15a1cbf31bb
 runc:
  Version:          1.2.4
  GitCommit:        v1.2.4-0-g6c52b3f
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
~~~



## docker daemon管理（可选）

docker本身是C/S，拥有Docker Client 与 Docker Server服务器端。

Docker Client 与 Docker Server之间的桥梁：docker daemon

![image-20250314121505568](images/image-20250314121505568.png)



可以将client与server进行分离，实现**远程docker连接**。为了实现它, 就需要对docker daemon进行相应的配置。

~~~powershell
远程客户端主机# docker -H 容器宿主机IP version
Client: Docker Engine - Community
 Version:           27.3.1
 API version:       1.47
 Go version:        go1.22.7
 Git commit:        ce12230
 Built:             Fri Sep 20 11:42:48 2024
 OS/Arch:           linux/amd64
 Context:           default
Cannot connect to the Docker daemon at tcp://10.1.1.11:2375. Is the docker daemon running?
~~~



**配置过程**

1, 修改docker配置文件前，请先关闭docker守护进程

~~~ powershell
[root@daniel ~]# systemctl stop docker
~~~



2, 通过/etc/docker/daemon.json文件对docker守护进程文件进行配置

```powershell
[root@daniel ~]# vim /etc/docker/daemon.json
{
  "hosts": ["tcp://0.0.0.0:2375","unix:///var/run/docker.sock"]
}
[root@daniel ~]# ss -naltp | grep 2375
[root@daniel ~]# ls /var/run/docker.sock
```

PS: docker daemon默认侦听使用的是unix格式，侦听文件：UNIX:///run/docker.sock,添加tcp//0.0.0.0:2375可实现远程管理。



3, 添加/etc/docker/daemon.json后会导致docker daemon无法启动, 请先修改如下文件内容：

~~~powershell
修改前：
[root@daniel ~]# vim /usr/lib/systemd/system/docker.service
[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock


修改后：
[root@daniel ~]# vim /usr/lib/systemd/system/docker.service
[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd
~~~

4, 修改完成后，一定要加载此配置文件

~~~powershell
[root@daniel ~]# systemctl daemon-reload
~~~

5, 重新开启docker守护进程

~~~powershell
[root@daniel ~]# systemctl start docker
[root@daniel ~]# ss -naltp | grep 2375
tcp6       0      0 :::2375                 :::*                    LISTEN      3318/dockerd
~~~

6, 实例远程连接方法

~~~powershell
远程客户端主机# docker -H 容器宿主机IP version
~~~

注意: 客户端远程连接不需要加端口号

**==特别注意: 远程客户端主机远程操作的权限非常大,请测试完后还原==**



**小结:**  docker engine分为client和server,默认都在本地 







# 四、镜像,容器,仓库

镜像(image):  镜像就是打包好的环境与应用。

容器(contanier):  容器就是运行镜像的实例.    镜像看作是静态的,容器是动态的。

仓库(repository):  存放多个镜像的一个仓库。

> 镜像：类似日常生活中的光盘（静态）；容器：基于光盘播放的内容（动态）；仓库：相当于光盘专卖店；



![1545199865792](./images/docker%E6%9E%B6%E6%9E%84%E5%9B%BE.png)

![1545313404886](./images/docker%E9%95%9C%E5%83%8F%E5%AE%B9%E5%99%A8%E4%BB%93%E5%BA%93.png)



# 五、镜像常见操作

## 仓库

默认docker连接官网仓库：http://hub.docker.com（科学上网）

## 镜像加速器

Docker近期由于未知原因关闭，给我们带来极大不便。网上的镜像源隔三岔五的失效，为了解决这个问题，我们自建一个DockerHub镜像加速器

1, 前置条件，准备一下账号

github 账号（全球最大的代码仓库，属于微软） =>  https://github.com/

cloudflare 账号（在线云服务器厂商，类似国内阿里云、腾讯云、华为云）

2, 访问 CF-Workers-docker.io 项目，并fork如下项目到自己的仓库

![image-20241202164158162](./images/image-20241202164158162.png)

![image-20241202164225345](./images/image-20241202164225345.png)

3, 部署到cf的pages服务中

注册Cloudflare

https://dash.cloudflare.com/sign-up

登录Cloudflare

https://dash.cloudflare.com/login

点击Worker和Pages

![image-20241202164804285](./images/image-20241202164804285.png)

再点概述

![image-20241202164829377](./images/image-20241202164829377.png)

点击 "点击pages"再点"连接到Git"

![image-20241202164949434](./images/image-20241202164949434.png)

连接 Github，首先添加Github账号，如下图所示：

![image-20241202165229150](./images/image-20241202165229150.png)

选择Github账号，找到`CF-Workers-docker.io`库

![image-20241202165413747](./images/image-20241202165413747.png)

点击开始设置，默认即可

![image-20241202165607461](./images/image-20241202165607461.png)

点击保存并部署

![image-20241202165811548](./images/image-20241202165811548.png)

部署完成后，如下图所示：

![image-20241202165842810](./images/image-20241202165842810.png)

点击继续处理项目，进入如下界面：

![image-20241202170038922](./images/image-20241202170038922.png)



点击访问，如下图所示：

![image-20241202170125581](./images/image-20241202170125581.png)

4，配置docker镜像加速器

~~~powershell
[root@daniel ~]# vim /etc/docker/daemon.json
{
	"registry-mirrors": ["https://cf-workers-docker-io-38g.pages.dev/"]
}
[root@daniel ~]# systemctl daemon-reload
[root@daniel ~]# systemctl restart docker

[root@daniel ~]# docker pull hello-world
[root@daniel ~]# docker pull cf-workers-docker-io-38g.pages.dev/hello-world
~~~



## 镜像分类 

1. 操作系统类(如centos,ubuntu)
2. 应用程序类

## 查看镜像列表（重点）

通过docker images命令查看当前镜像列表; 使用man docker-images得到参数说明

~~~powershell
[root@daniel ~]# docker images
~~~

## 搜索镜像（重点）

通过docker search查找官方镜像; 使用man docker-search得到参数说明

~~~powershell
[root@daniel ~]# docker search centos
或
[root@daniel ~]# docker search cf-workers-docker-io-38g.pages.dev/centos
NAME                           DESCRIPTION                                     STARS     OFFICIAL
centos                         DEPRECATED; The official build of CentOS.       7757      [OK]
centos/postgresql-10-centos7   PostgreSQL is an advanced Object-Relational …   20
corpusops/centos               centos corpusops baseimage                      0
centos/redis-5-centos8                                                         0
centos/postgresql-96-centos7   PostgreSQL is an advanced Object-Relational …   45
centos/httpd-24-centos8                                                        3
centos/postgresql-10-centos8                                                   0
centos/systemd                 systemd enabled base container.                 115
centos/mysql-80-centos8                                                        0
centos/mongodb-36-centos7      MongoDB NoSQL database server                   12
centos/mysql-56-centos7        MySQL 5.6 SQL database server                   23
centos/nginx-112-centos7       Platform for running nginx 1.12 or building …   16
centos/mariadb-103-centos8                                                     2
centos/postgresql-12-centos8                                                   0
centos/ruby-25-centos7         Platform for building and running Ruby 2.5 a…   3
centos/mariadb-101-centos7     MariaDB 10.1 SQL database server                13
centos/mariadb-102-centos7     MariaDB 10.2 SQL database server                6
centos/mysql-57-centos7        MySQL 5.7 SQL database server                   95
centos/redis-32-centos7        Redis in-memory data structure store, used a…   6
centos/httpd-24-centos7        Platform for running Apache httpd 2.4 or bui…   46
centos/php-56-centos7          Platform for building and running PHP 5.6 ap…   34
centos/nginx-18-centos7        Platform for running nginx 1.8 or building n…   14
centos/redis-5-centos7         Redis in-memory data structure store, used a…   0
centos/nginx-116-centos7       Platform for running nginx 1.16 or building …   3
centos/python-35-centos7       Platform for building and running Python 3.5…   39
~~~

## 拉取镜像（重点）

通过docker pull拉取(下载)镜像; 使用man docker-pull得到参数说明

~~~powershell
此镜像大概200多M，网速要好
[root@daniel ~]# docker pull centos-stream-9
或
[root@daniel ~]# docker pull cf-workers-docker-io-38g.pages.dev/centos-stream-9   名字为search查找时得到的全名

如果网速慢,可以试试阿里,腾讯,百度,网易等国内的镜像仓库，比如:
[root@daniel ~]# docker pull cf-workers-docker-io-38g.pages.dev/library/centos-stream-9:latest
~~~

~~~powershell
[root@daniel ~]# docker images
REPOSITORY                    TAG            IMAGE ID        CREATED          SIZE
docker.io/centos              latest         1e1148e4cc2c    13 days ago      202 MB
~~~

如果拉取不了只有3种方式：

① 通过科学上网工具

② 按照笔记构建代理服务器（免费10万次请求）

③ 购买国外服务器，如日本、新加坡、美国

---

从网上搜索一些免费的代理，可能不稳定！

```powershell
vim /etc/docker/daemon.json
{
  "hosts": ["tcp://0.0.0.0:2375","unix:///var/run/docker.sock"],
  "registry-mirrors": [
	"https://docker.1panel.live",
	"https://docker.fxxk.dedyn.io",
	"https://hub.rat.dev",
	"https://docker.anyhub.us.kg",
	"https://dockerhub.jobcher.com",
	"https://dockerhub.icu",
	"https://mirror.aliyuncs.com",
	"https://mirror.baidubce.com",
	"https://docker.nju.edu.cn",
	"https://docker.mirrors.sjtug.sjtu.edu.cn",
	"https://docker.mirrors.ustc.edu.cn",
	"https://mirror.iscas.ac.cn",
	"https://docker.rainbond.cc",
	"https://atomhub.openatom.cn",
	"https://dockerpull.com"
  ]
}
```



## 删除镜像（重点）

通过docker rmi删除镜像; man docker-rmi查看参数帮助

~~~powershell
[root@daniel ~]# docker rmi centos:latest
rmi：两个单词合成，remove移除，image镜像
~~~



**问题:如果镜像pull非常慢,怎么解决?**

1. docker镜像加速器
2. 可以从网速好的宿主机上pull下来,然后**导出**给网速慢的宿主机**导入**



## 镜像导出

使用docker save保存(导出)镜像为一个tar文件

~~~powershell
[root@daniel ~]# docker save centos -o /root/dockerimage_centos.latest
~~~

## 镜像导入

使用docker load导入

测试时可以将导出的文件scp传输到另一台宿主机测试。或者先删除本地的镜像再导入测试

~~~powershell
[root@daniel ~]# docker load < /root/dockerimage_centos.latest
~~~



## 镜像重命名

如果导入后看不到名称,可以使用`docker tag`命令改名称 

~~~powershell
[root@daniel ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED           SIZE
<none>              <none>              9f38484d220f        3 months ago      202 MB

[root@daniel ~]# docker tag 9f38484d220f docker.io/centos:latest

[root@daniel ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED          SIZE
docker.io/centos    latest              9f38484d220f        3 months ago      202 MB
~~~

>注意： docker tag 并不是把镜像复制一份，而是为其创建了一个链接，这个连接指向了原有镜像文件。
>
>如果删除时，会有何影响？
>
>答:
>
>第一个：如果是通过名称——tag标签删除，则只删除引用，底层镜像并不会正真删除。
>
>第二个：如果是通过IMAGE ID 删除。则会清理所有引用关系，然后删除镜像本身。

# 六、镜像仓库

## 官方自建镜像仓库（了解）

docker hub为最大的公开仓库,也就是官方仓库: https://hub.docker.com/

> 注：需要科学上网才能登录

1, 没有账号的先上网申请账号

![1545313684527](./images/docker%E5%AE%98%E6%96%B9%E4%BB%93%E5%BA%93.png)

![1545315721942](./images/docker%E5%AE%98%E6%96%B9%E4%BB%93%E5%BA%932.png)

![1545313995008](./images/docker%E5%AE%98%E6%96%B9%E4%BB%93%E5%BA%933.png)

![1545314066856](./images/docker%E5%AE%98%E6%96%B9%E4%BB%93%E5%BA%934.png)

![1545314123454](./images/docker%E5%AE%98%E6%96%B9%E4%BB%93%E5%BA%935.png)

![1545314531697](./images/docker%E5%AE%98%E6%96%B9%E4%BB%93%E5%BA%936.png)

![1545314630600](./images/docker%E5%AE%98%E6%96%B9%E4%BB%93%E5%BA%937.png)

2, 回到宿主机登录账号与密码


~~~powershell
# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: linuxdaniel
Password: 
Login Succeeded
~~~

3, tag你的镜像

我们从dockerhub上下载的公开镜像是不能直接上传的，要先tag(打标签,**类似于重新指定路径并命名**)

~~~powershell
[root@daniel ~]# docker push centos:latest
Error response from daemon: You cannot push a "root" repository. Please rename your repository to docker.io/<user>/<repo> (ex: docker.io/<user>/centos)

[root@daniel ~]# docker tag centos:latest linuxdaniel/daniel_docker_repo:V1
[root@daniel ~]# docker push linuxdaniel/daniel_docker_repo:V1
~~~

4, push镜像到仓库

~~~powershell
[root@daniel ~]# docker push linuxdaniel/daniel_docker_repo:V1
~~~

5, 验证



![1545315358701](./images/docker%E5%AE%98%E6%96%B9%E4%BB%93%E5%BA%938.png)

6, 登出账号

~~~powershell
[root@daniel ~]# docker logout
Removing login credentials for https://index.docker.io/v1/
~~~

## 阿里云镜像仓库（重点）

登录阿里云账号，主页面上找产品-->容器--> 容器镜像服务-->然后使用账号登录

![image-20250314172310809](images/image-20250314172310809.png)

登录成功后,做下列操作

1, 创建命名空间(命令空间和镜像仓库名称合到一起组成镜像的路径名称)

![1545316723523](./images/%E9%98%BF%E9%87%8C%E4%BA%91%E5%88%9B%E5%BB%BA%E4%BB%93%E5%BA%93.png)

![1545316801301](./images/%E9%98%BF%E9%87%8C%E4%BA%91%E5%88%9B%E5%BB%BA%E4%BB%93%E5%BA%932.png)

2, 创建镜像仓库,指定仓库名称


![1545317012951](./images/%E9%98%BF%E9%87%8C%E4%BA%91%E5%88%9B%E5%BB%BA%E4%BB%93%E5%BA%933.png)

![1545317073711](./images/%E9%98%BF%E9%87%8C%E4%BA%91%E5%88%9B%E5%BB%BA%E4%BB%93%E5%BA%934.png)

![1545317515141](./images/%E9%98%BF%E9%87%8C%E4%BA%91%E5%88%9B%E5%BB%BA%E4%BB%93%E5%BA%935.png)

登录前操作

![image-20250314174108962](images/image-20250314174108962.png)

> 早期docker login登录镜像仓库使用阿里云登录密码，最近版本更新后，要求增加一个固定密码，必须按照以上设置，否则无法登录仓库

登录阿里云镜像仓库

```powershell
docker login --username=pybigdata crpi-fxm825mzhlhavg49.cn-hangzhou.personal.cr.aliyuncs.com
输入访问凭证中的固定密码
```

给本地镜像打Tag标签，如果没有Tag无法上传

```powershell
docker tag 482a3731ce54 crpi-fxm825mzhlhavg49.cn-hangzhou.personal.cr.aliyuncs.com/linuxdaniel_namespace/centos:v1
docker push crpi-fxm825mzhlhavg49.cn-hangzhou.personal.cr.aliyuncs.com/linuxdaniel_namespace/centos:v1
```

登出方法

~~~powershell
使用docker logout接地址
# docker logout crpi-fxm825mzhlhavg49.cn-hangzhou.personal.cr.aliyuncs.com
~~~

具体参考时，一定要参考自己仓库设置，需要更改参数如下：

![image-20250315084631293](images/image-20250315084631293.png)

## harbor私有镜像仓库（重点）

docker => dockerfile => docker-compose



Harbor是VMware公司开源了企业级Registry项目, 可以帮助用户快速搭建一个企业级的Docker registry服务.

harbor由python语言开发, 需要使用`docker-compose`工具进行启动

说明: docker-compose是容器编排工具,会在后面的docker三剑客中讲解



### **环境准备**

再准备一台新的虚拟机(192.168.88.20)做harbor服务器



### 安装过程

**1, 安装docker-compose**

==以下2种方法任选其一==

安装方法1:

安装python3-pip,然后通过pip安装docker-compose模块

~~~powershell
[root@harbor ~]# yum install epel-release -y
[root@harbor ~]# yum install python3-pip -y
[root@harbor ~]# pip install -i https://pypi.tuna.tsinghua.edu.cn/simple docker-compose
指定清华源安装速度较快

[root@harbor ~]# docker-compose -v
~~~

安装方法2（推荐）:

~~~powershell
[root@harbor ~]# mv docker-compose-linux-x86_64 /usr/local/bin/docker-compose
[root@harbor ~]# chmod +x /usr/local/bin/docker-compose
~~~



**2, 在harbor服务器上安装docker-ce并启动docker服务**


~~~powershell
[root@harbor ~]# wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
[root@harbor ~]# yum install docker-ce

[root@harbor ~]# systemctl start docker 
[root@harbor ~]# systemctl enable docker
~~~



**3,  安装harbor**

harbor分为离线包和在线包两种。在线包较小，但需要连网下载。我这里使用离线包

下载地址: https://github.com/goharbor/harbor/releases

我这里提供了**harbor-offline-installer-v2.12.2.tgz**给大家拷贝到harbor服务器上

~~~powershell
[root@harbor ~]# tar xf harbor-offline-installer-v2.12.2.tgz -C /usr/local/
[root@harbor ~]# cd /usr/local/harbor/
[root@harbor ~]# cp harbor.yml.tmpl harbor.yml
[root@harbor ~]# vim harbor.yml
  5 hostname: harbor.itcast.cn						将hostname改成harbor服务器主机名称
  
  7 # http related config
  8 http:
  9   # port for http, default is 80. If https enabled, this port will redirect to https port
 10   port: 80
 11
 12 # https related config
 13 #https:												 					关闭https
 14   # https port for harbor, default is 443  关闭https
 15   # port: 443														关闭https
 16   # The path of cert and key files for nginx
 17   # certificate: /your/certificate/path
 18   # private_key: /your/private/key/path
 19   # enable strong ssl ciphers (default: false)
 20   # strong_ssl_ciphers: false

 47 harbor_admin_password: 123					admin用户的默认密码，我这里改为简单的123



[root@harbor harbor]# ./install.sh
......
......
......
✔ ----Harbor has been installed and started successfully.----

Now you should be able to visit the admin portal at http://192.168.88.20.
For more details, please visit https://github.com/goharbor/harbor .
~~~

**4,浏览器访问 http://192.168.88.20,登录进行配置**

![image-20250314181645151](images/image-20250314181645151.png)

![1545394272687](./images/Harbor2.png)

![1545394369711](./images/Harbor3.png)

公开项目: 下载镜像不需要docker login登录，但上传镜像还是需要docker login登录

私有项目: 都需要docker login登录才以上传下载

### 镜像上传下载操作

**5, 在docker宿主机配置非https连接(docker服务器)**

因为docker用https通讯,所以还需要做证书,太麻烦。

配置**"insecure-registries": ["harbor服务器IP"]**来使用http通讯


~~~powershell
[root@daniel ~]# vim /etc/docker/daemon.json
{
	"registry-mirrors": ["https://cf-workers-docker-io-38g.pages.dev/"],     这里有一个逗号
	"insecure-registries": ["192.168.88.20"]
}
[root@daniel ~]# systemctl restart docker
~~~

**6, 在docker宿主机登下载一个测试镜像,并tag成**  **harborIP/项目名/镜像名:TAG**

~~~powershell
[root@daniel ~]# docker pull hello-world
[root@daniel ~]# docker tag hello-world 192.168.88.20/test/hello-world:v1

打标签：docker tag，上传到哪里，docker tag打标签是有要求的
docker tag 镜像名称/镜像ID harbor的IP地址/项目/仓库:版本号
~~~

**7, 登陆服务器,并push上传镜像**

登录之前，在所有服务器中添加主机与IP映射

```powershell
vim /etc/hosts
192.168.88.10 docker01.itcast.cn
192.168.88.20 harbor.itcast.cn
```

然后使用docker login登录harbor

~~~powershell
[root@daniel ~]# docker login 192.168.88.20
Username: admin
Password: 					密码为前面修改好的123
Login Succeeded

[root@daniel ~]# docker push 192.168.88.20/test/hello-world:v1

不用了可以logout
[root@daniel ~]# docker logout 192.168.88.20
~~~

**8, 浏览器界面验证**

![1545398764531](./images/Harbor4.png)

**9, docker宿主机想要pull上传的镜像,可以这样做**

~~~powershell
删除镜像再重新从harbor仓库上下载
[root@daniel ~]# docker rmi 192.168.88.20/test/hello-world:v1

私有项目里的镜像需要先登录,再pull(公共项目里的镜像不用登录就可以直接pull,请自行测试)
[root@daniel ~]# docker login 192.168.88.20
Username: admin
Password: 					密码为前面设置的123
Login Succeeded

[root@daniel ~]# docker pull 192.168.88.20/test/hello-world:v1
~~~



小结: 远程仓库

* 官方仓库  缺点:网络问题
* 国内云运营商提供的镜像仓库   优点:网速较好  缺点: 安全性考虑

* 自建仓库  优点: 网速好,安全性也好  缺点: 自己维护,需要服务器和存储成本



答疑：

问题1：为什么要打标签 => docker tag

答案：因为docker默认连接官方镜像仓库，如果不打标签，则拉取与上传都是使用官方仓库。但是由于某些原因，官方镜像仓库无法访问，我们可以使用阿里云或者私有镜像仓库

问题2：明明更改了/etc/docker/daemon.json文件，但是没有效果？

答案：没有重启docker

问题3：明明昨天还可以访问harbor，今天突然无缘无故无法访问？神操作：`systemctl restart docker`

答案：因为harbor是通过docker-compose安装，docker-compose是容器编排工具，整个harbor，后台有很多个容器同时运行（nginx容器、mysql容器等等），如果重启docker，会导致所有容器，全部停止。

```powershell
[root@harobr ~] # cd /usr/local/harbor
[root@harobr ~] # docker compose down
[root@harobr ~] # docker compose up -d
```



# 七、容器常见操作

目标：镜像 =》 运行 =》容器（进行相关操作）

## 查看容器列表

列表所有状态的容器,现在为空列表
使用man docker-ps得到参数说明

~~~powershell
[root@daniel ~]# docker ps -a
~~~

## 运行第一个容器

通过hello-world这个镜像,运行一个容器(没有定义容器名称,则为随机名称)

* 当前docker-host(容器宿主机)如果有hello-world这个镜像,则直接使用
* 如果没有相关镜像,则会从docker hub去下载(配置了镜像加速器的优先找加速器)

使用man docker-run得到参数说明

~~~powershell
[root@daniel ~]# docker run hello-world
~~~

再次查看容器列表,多了一个容器,但它的状态是exited，此容器就是运行了一句Hello from Docker!就退出了(我这里容器名随机为silly_lovelace)

~~~powershell
[root@daniel ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
6e3f991b9e8a        hello-world         "/hello"            3 minutes ago       Exited (0) 2 minutes ago                       silly_lovelace

上面列表格式比较长，看起来有些不舒服。可以简单处理下,只查看重要的几列
[root@daniel ~]# docker ps -a |awk -F"[  ]{2}*" '{print $1"\t\t"$3"\t\t"$5"\t"$NF}'
CONTAINER ID		 COMMAND			 	STATUS						NAMES
6e3f991b9e8a		 "/hello"			 	Exited (0) 3 minutes ago	silly_lovelace
~~~

问题: 为什么容器运行完hello-world后就退出了,而不是继续运行?

我们前面把容器比喻为轻量级虚拟机,但是容器实际上只是**==进程==**。进行完了当然就退出了, 除非是类似服务那样的守护进程。

所以综上所述：容器大致分为两种类型

一次性运行：如hello-world、busybox:latest 只运行一次就会自动结束

永久性运行：如nginx tomcat、mysql容器。一旦启动后，就会一直运行



## 容器运行命令或脚本

带守护进程的镜像也可以选择运行方式：① 临时运行1次（为了得到某个结果）② 永久运行（持续不断提供服务）

---

指定使用docker.io/centos镜像运行"echo haha"命令
latest是默认的TAG标签,可以省略;如果是其它TAG就不能省略，否则会默认为latest

~~~powershell
[root@daniel ~]# docker run centos:latest echo haha
haha
~~~

docker运行一个不间断的脚本, -d表示后台运行(后台运行表示不输出结果到屏幕)

~~~powershell
[root@daniel ~]# docker run -d centos /bin/bash -c "while true; do echo haha;sleep 3;done"
-d: 后台运行
/bin/bash -c: “执行shell命令”
~~~

查看刚才运行的容器

~~~powershell
只有不间断运行脚本的容器还在UP状态，其它都为Exited状态
[root@daniel ~]# docker ps -a |awk -F"[  ]{2}*" '{print $1"\t\t"$3"\t\t"$5"\t"$NF}'
CONTAINER ID	COMMAND			 		STATUS					   NAMES
21086dab3efa	"/bin/bash -c 'whi..."	Up 1 minutes			   sleepy_ride
495310e96d9f	"echo haha"			 Exited (0) 3 minutes ago	 unruffled_jepsen
6e3f991b9e8a	"/hello"			 Exited (0) 15 minutes ago    silly_lovelace
~~~



## 查看容器运行结果

~~~powershell
后面接容器ID,也可以接容器名称
[root@daniel ~]# docker logs 21086dab3efa
~~~

> 以上方式除了可以正常查看日志以外，还可以帮我们排查容器故障！
>
> 备注：
>
> docker logs: 查看容器运行时产生的日志信息
>
> docker inspect : 查看这个容器自身的一些信息，基于那个镜像，网络，版本等。

## 停止容器

~~~powershell
[root@daniel ~]# docker stop 21086dab3efa
~~~

## 启动容器

~~~powershell
[root@daniel ~]# docker start 21086dab3efa
~~~

## 查看容器的相关信息

~~~powershell
[root@daniel ~]# docker inspect 21086dab3efa
~~~

> inspect主要用于查看容器信息，一定要记住！！！



## 进入容器并交互式操作

>永久运行的容易，往往支持直接进入到容器内部，进行一系列的操作

~~~powershell
使用下面命令启动容器
docker run
-i指交互;-t指tty终端;--name是用来指定容器名称

[root@daniel ~]# docker run -i -t --name=c1 centos:latest /bin/bash
[root@f736fe36002c /]# cat /etc/redhat-release 
CentOS Stream release 9 			可以看到我们下载的centos是stream 9版本
[root@f736fe36002c /]# uname -r
5.14.0-542.el9.x86_64				查看的内核却与宿主机centos stream 9一样,说明是共享宿主机的内核

在容器内操作(我这里创建一个文件，然后退出)
[root@f736fe36002c /]# touch /root/daniel
[root@f736fe36002c /]# exit
exit
~~~

> 如果容器中输入了exit，则会导致容器主进程结束，最终容器也会Exited！如果退出后，不想让容器也随之退出，使用快捷键：Cttrl + P + Q



**实验: 交互式操作退出后如何再查看或修改先前在容器里创建的文件?**

~~~powershell
不要再使用下面的命令了，因为名称冲突，会报错（换一个名称会启动一个新的容器)
[root@daniel ~]# docker run -i -t --name=c1 centos:latest /bin/bash
~~~

1，使用下面命令查看到c1容器已经为Exited状态;-l表示列表最近的容器

~~~powershell
[root@daniel ~]# docker ps -l
CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS                      PORTS               NAMES
f736fe36002c        centos:latest             "/bin/bash"              2 minutes ago      Exited (0) 1 minutes ago                       c1
~~~

2，启动容器

~~~powershell
[root@daniel ~]# docker start c1
~~~

3，然后使用attach指令连接UP状态的容器（Exited状态的容器无法attach)

~~~powershell
[root@daniel ~]# docker attach c1
或者使用：docker exec -it c1 /bin/bash
~~~

4，验证文件,可以按需求修改

~~~powershell
[root@f736fe36002c /]# ls /root/daniel -l
-rw-r--r-- 1 root root 0 Dec 19 12:30 /root/daniel
[root@f736fe36002c /]# exit
exit
~~~

5, commit提交成一个新的镜像;c63d63bff173为容器ID;test_image为新的镜像名称

~~~powershell
[root@daniel ~]# docker commit c63d63bff173 test_image
sha256:bfb4e268cc6b683e1fc19346daf446ddd85dc7d75bcaa5cfd80978ac271a913f
~~~

6，验证新的镜像

~~~powershell
[root@daniel ~]# docker images |grep test_image
test_image             latest          bfb4e268cc6b        47 seconds ago      202 MB
~~~



## 容器外指定容器运行命令

可以在宿主机通过exec指令传命令到容器中执行，但要求容器为UP状态

~~~powershell
[root@daniel ~]# docker start c1

在container1容器里创建文件并验证;man docker-exec查看帮助
[root@daniel ~]# docker exec c1 touch /root/123
[root@daniel ~]# docker exec c1 ls -l /root/123

或者用下面的命令连接上去交互式操作
[root@daniel ~]# docker exec -it c1 /bin/bash
~~~



## 删除容器

UP状态的容器要先停止才能删除

~~~powershell
[root@daniel ~]# docker stop 21086dab3efa
[root@daniel ~]# docker rm 21086dab3efa
~~~

批量删除所有容器

~~~powershell
加-q参数只查看所有容器的ID
[root@daniel ~]# docker ps -aq
f736fe36002c
21086dab3efa
495310e96d9f
6e3f991b9e8a
停止所有容器
[root@daniel ~]# docker stop $(docker ps -aq)
删除所有容器
[root@daniel ~]# docker rm $(docker ps -aq)
~~~





小结: 

* docker ps   -a : 列出本地的所有容器信息
* docker run 参数选项 --name 容器名 镜像名:TAG  传给容器内部执行的命令
* docker logs 容器名或容器ID:  输出容器内执行命令的结果
* docker stop 容器名或容器ID: 停止容器
* docker start 容器名或容器ID: 启动容器
* docker attach 容器名或容器ID: 连接一个UP状态的容器,可以进去交互(有bash环境的才可以)
* docker exec 容器名或容器ID  命令:   不用连接容器,可以外部传命令给容器内部操作
* docker exec -it 容器名或容器ID /bin/bash   连接容器交互
* docker inspect 容器名或容器ID: 查看容器的属性
* docker rm 容器名或容器ID: 删除容器
* docker commit 容器名或容器ID 新的镜像名:TAG          将容器提交为一个镜像



**==需要记忆的核心命令==:**

* docker ps -a
* docker run （**重难点**)
* docker start  ;  docker stop;
* docker commit
* docker exec;  docker exec -it 容器名或容器ID /bin/bash
* docker rm



# 八、docker存储驱动（理解）

## 写时复制与用时分配

通过上面的学习,我们知道了一个镜像可以跑多个容器,如果每个容器都去复制一份镜像内的文件系统,那么将会占用大量的存储空间。docker使用了**==写时复制cow(copy-on-write)==**和**==用时分配(allocate-on-demand)==**技术来**==提高存储的利用率==**。

**写时复制**: 

* 写时复制技术可以让多个容器共享同一个镜像的文件系统, 所有数据都从镜像中读取

* 只有当要对文件进行写操作时，才从镜像里把要写的文件复制到自己的文件系统进行修改。所以无论有多少个容器共享同一个镜像，所做的写操作都是对从镜像中复制到自己的文件系统中的副本上进行，并不会修改镜像的源文件

* 多个容器操作同一个文件，会在每个容器的文件系统里生成一个副本，每个容器修改的都是自己的副本，相互隔离，相互不影响

**用时分配:**

启动一个容器，并不会为这个容器预分配一些磁盘空间，而是当有新文件写入时，才按需分配新空间



## 联合文件系统

联合文件系统(UnionFS)就是把不同物理位置的目录合并mount到同一个目录中.

比如你可以将一个光盘与一个硬盘上的目录联合挂载到一起,然后对只读的光盘文件进行修改,修改的文件不存放回光盘进行覆盖,而是存放到硬盘目录。这样做达到了不影响光盘原数据,而修改的目的。

**思考: 把光盘看作是docker里的image,而硬盘目录看作是container,你再想想看?**

docker就支持aufs和overlay两种联合文件系统。



## aufs

Docker最开始采用AUFS作为文件系统，也得益于AUFS分层的概念，实现了多个Container可以共享同一个image。


aufs(Another UnionFS)，后来叫Alternative UnionFS，后来可能觉得不够霸气，叫成Advance UnionFS.

Docker最开始采用AUFS作为文件系统，也得益于AUFS分层的概念，实现了多个Container可以共享同一个image

![1564114179352](./images/aufs%E5%88%86%E5%B1%82%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

除了最上面的一层为读写层之外，下面的其他的层都是只读的镜像层.



## overlay

由于AUFS未并入Linux内核，且只支持Ubuntu，考虑到兼容性问题，在Docker 0.7版本中引入了存储驱动。 目前，Docker支持AUFS,OverlayFS,Btrfs,Device mapper,ZFS五种存储驱动.

目前, 在ubuntu发行版上默认存储方式为AUFS,CentOS发行版上的默认存储方式为Overlay或Overlay2

~~~powershell
# docker info |grep "Storage Driver"	
Storage Driver: overlay2
~~~

~~~powershell
# lsmod |egrep 'aufs|overlay'			
overlay                71964  7	
~~~

centos上加载了overlay模块,从3.18版本内核开始,就进入了Linux内核主线



Overlay是Linux内核3.18后支持的(当前3.10内核加载模块也可以使用),也是一种Union FS,和AUFS的多层不同的是Overlay只有两层：一个upper文件系统和一个lower文件系统，分别代表Docker的容器层和镜像层..

OverlayFS底层目录称为lowerdir,高层目录称为upperdir。合并统一视图称为merged。当需要修改一个文件时，使用cow将文件从只读的Lower复制到可写的Upper进行修改，结果也保存在Upper层。在Docker中，底下的只读层就是image，可写层就是Container

下图分层图，镜像层是lowdir，容器层是upperdir,统一的视图层是merged层. 

视图层就是给用户提供了一个统一的视角，隐藏了多个层的复杂性，对用户来说只存在一个文件系统。

![1564115927302](./images/overlay%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86.png)



从上图中可以看到: 

* 如果upperdir和lowerdir有同名文件时会用upperdir的文件 

* 读文件的时候，文件不在upperdir则从lowerdir读

* 如果写的文件不在uppderdir在lowerdir,则从lowerdir里面copy到upperdir。

* 不管文件多大,copy完再写,删除镜像层的文件只是在容器层生成whiteout文件标志(标记为删除,并不是真的删除)  （后面的dockefile章节会体现出效果)

  





## aufs,overlay,overlay2对比

aufs: 使用多层分层

overlay: 使用2层分层, 共享数据方式是通过**硬连接**，只挂载一层,其他层通过最高层通过硬连接形式共享(**增加了磁盘inode的负担**)

overlay2: 使用2层分层, 驱动原生地支持多层lower overlay镜像(最多128层),与overlay驱动对比,消耗更少的inode



## 不同阶段观察存储情况

新准备一台VM实例,重新安装docker(安装过程参考前面章节),进行如下测试



### docker第1次启动前

在刚安装docker-ce第1次启动服务之前,/var/lib/下并没有docker这个目录

### docker启动后

而第1次`systemctl start docker`启动后,则会产生`/var/lib/docker`目录

~~~powershell
[root@vm2 ~]# systemctl start docker

[root@vm2 ~]# ls -l /var/lib/docker
total 0
drwx------ 2 root root 24 Jun 23 15:30 builder
drwx------ 4 root root 92 Jun 23 15:30 buildkit
drwx------ 2 root root  6 Jun 23 15:30 containers
drwx------ 3 root root 22 Jun 23 15:30 image
drwxr-x--- 3 root root 19 Jun 23 15:30 network
drwx------ 3 root root 40 Jun 23 15:39 overlay2
drwx------ 4 root root 32 Jun 23 15:30 plugins
drwx------ 2 root root  6 Jun 23 15:39 runtimes
drwx------ 2 root root  6 Jun 23 15:30 swarm
drwx------ 2 root root  6 Jun 23 15:41 tmp
drwx------ 2 root root  6 Jun 23 15:30 trust
drwx------ 2 root root 25 Jun 23 15:30 volumes
~~~

~~~powershell
[root@vm2 ~]# cd /var/lib/docker/overlay2/
[root@vm2 overlay2]# ls
total 0
brw------- 1 root root 8, 3 Jul 26 14:10 backingFsBlockDev
drwx------ 2 root root    6 Jul 26 14:10 l
~~~





### 下载镜像后

~~~powershell
[root@vm2 overlay2]# docker pull centos
Using default tag: latest
latest: Pulling from library/centos
8ba884070f61: Pull complete
Digest: sha256:a799dd8a2ded4a83484bbae769d97655392b3f86533ceb7dd96bbac929809f3c
Status: Downloaded newer image for centos:latest

~~~



~~~powershell
[root@vm2 overlay2]# pwd
/var/lib/docker/overlay2
[root@vm2 overlay2]# ls
23664d7a4167e74ee04838d87cd3568cc82be49f781bba2212b9bff942bb8fa4  backingFsBlockDev  l

[root@vm2 overlay2]# ls 23664d7a4167e74ee04838d87cd3568cc82be49f781bba2212b9bff942bb8fa4/
diff  link

[root@vm2 overlay2]# ls 23664d7a4167e74ee04838d87cd3568cc82be49f781bba2212b9bff942bb8fa4/diff/
anaconda-post.log  bin  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

[root@vm2 overlay2]# cat 23664d7a4167e74ee04838d87cd3568cc82be49f781bba2212b9bff942bb8fa4/link
5D7D6BY2V3FKMZHUU6VHK7ILWL

[root@vm2 overlay2]# ll l
total 0
lrwxrwxrwx 1 root root 72 Jul 26 14:16 5D7D6BY2V3FKMZHUU6VHK7ILWL -> ../23664d7a4167e74ee04838d87cd3568cc82be49f781bba2212b9bff942bb8fa4/diff
~~~

* 下载完镜像,overlay2目录里多了一个`23664....`这样的目录,只有1层
* 此目录内部的diff子目录记录每一层自己的数据
* link记录该层链接目录(和overlay目录下l子目录里记录的链接是一致的)
* l子目录中包含了很多软链接，使用短名称指向了其他层，短名称用于避免mount参数时达到页面大小的限制



### 运行容器后

~~~powershell
[root@vm2 overlay2]# docker run -i -t --name=c1 centos:latest /bin/bash
[root@9169c38e6424 /]
同时按ctrl+p+q三键退出,保持容器为UP状态
~~~

~~~powershell
[root@vm2 overlay2]# docker ps -a
CONTAINER ID     IMAGE      COMMAND     CREATED         STATUS        PORTS        NAMES
9169c38e6424  centos:latest  "/bin/bash"  12 seconds ago  Up 10 seconds           c1
~~~

容器运行后,再次查看overlay2目录下的情况 

![1564122840623](./images/%E5%AE%B9%E5%99%A8%E8%BF%90%E8%A1%8C%E5%90%8Eoverlay2%E7%9B%AE%E5%BD%95%E4%B8%8B%E7%8A%B6%E6%80%81.png)



![1564123372970](./images/%E5%AE%B9%E5%99%A8%E8%BF%90%E8%A1%8C%E5%90%8Eoverlay2%E6%8C%82%E8%BD%BD%E4%BF%A1%E6%81%AF.png)

~~~powershell
[root@vm2 overlay2]# ll 3652e67b65ebe7eaed7cc879f8470983181fc19fee2f3d72173e67666e28ec74/
total 8
drwxr-xr-x 2 root root  6 Jul 26 14:30 diff
-rw-r--r-- 1 root root 26 Jul 26 14:30 link
-rw-r--r-- 1 root root 57 Jul 26 14:30 lower
drwxr-xr-x 1 root root  6 Jul 26 14:30 merged
drwx------ 3 root root 18 Jul 26 14:30 work
~~~

* lower指定了下层
* work用来完成如copy-on_write的操作。

~~~powershell
[root@vm2 overlay2]# cat 3652e67b65ebe7eaed7cc879f8470983181fc19fee2f3d72173e67666e28ec74/lower
l/UQF6DWQE64JXS3TEBUFOBSD3TY:l/5D7D6BY2V3FKMZHUU6VHK7ILWL
~~~

~~~powershell
[root@vm2 overlay2]# ls -l l/
total 0
lrwxrwxrwx 1 root root 72 Jul 26 14:16 5D7D6BY2V3FKMZHUU6VHK7ILWL -> ../23664d7a4167e74ee04838d87cd3568cc82be49f781bba2212b9bff942bb8fa4/diff
lrwxrwxrwx 1 root root 72 Jul 26 14:30 KBKN2UECH7JGUJBUGIVHYV4ERX -> ../3652e67b65ebe7eaed7cc879f8470983181fc19fee2f3d72173e67666e28ec74/diff
lrwxrwxrwx 1 root root 77 Jul 26 14:30 UQF6DWQE64JXS3TEBUFOBSD3TY -> ../3652e67b65ebe7eaed7cc879f8470983181fc19fee2f3d72173e67666e28ec74-init/diff
~~~

**通过上面的测试总结:**

* 用户看到的文件系统层为:

~~~powershell
/var/lib/docker/overlay2/3652e67b65ebe7eaed7cc879f8470983181fc19fee2f3d72173e67666e28ec74/merged
~~~

* 它由

~~~powershell
/var/lib/docker/overlay2/3652e67b65ebe7eaed7cc879f8470983181fc19fee2f3d72173e67666e28ec74-init/diff/
和
/var/lib/docker/overlay2/23664d7a4167e74ee04838d87cd3568cc82be49f781bba2212b9bff942bb8fa4/diff/
~~~

联合挂载而成

* 以下目录用于copy-on-write操作

~~~powershell
/var/lib/docker/overlay2/3652e67b65ebe7eaed7cc879f8470983181fc19fee2f3d72173e67666e28ec74/work
~~~

* 如果使用`docker attach 容器名`连接后在容器内创建文件的话,它会存放在

~~~powershell
/var/lib/docker/overlay2/3652e67b65ebe7eaed7cc879f8470983181fc19fee2f3d72173e67666e28ec74/diff
~~~



**小结:**

* docker在centos7-9目前使用的存储驱动为overlay2
* docker通过写时复制(cow)和用时分配来提高存储的效率
* aufs和ovelay属于联合文件系统
* aufs是多层分层
* overlay主要为2两层: lowerdir和upperdir
* overlay2相比于overlay节省innode，所以新版docker都是基于overlay2，并已经写入到linux内核中

>总结：
>
>docker如何占用更少的空间？
>
>答：写时复制用时分配，同时配合overlay2联合文件系统可以大大提升存储的应用






# 九、容器里跑应用

前面我们熟悉了容器的常见操作，但容器中并没有跑过应用程序.而生产环境是要用容器来跑应用程序的。

在**宿主机上打开ip_forward**, 因为我们下面要映射容器的端口到宿主机,只有打开ip_forward才能映射成功

~~~powershell
[root@daniel ~]# vim /etc/sysctl.conf
net.ipv4.ip_forward = 1
[root@daniel ~]# sysctl -p
~~~



## 容器中运行httpd应用

回顾镜像:

* 系统镜像
* 应用镜像

首先我们通过系统镜像来跑httpd



### **案例1**: 端口映射

利用官方centos镜像运行容器跑httpd服务,因为官方centos镜像里默认并没有安装httpd服务,所以需要我们自定义安装

docker内部跑httpd启动80端口,需要与docker_host(宿主机)进行端口映射,才能让客户端通过网络来访问



1, 运行容器httpd1;  -p 8000:80的意思是把容器里的80端口映射为docker_host(宿主机)的8000端口

~~~powershell
[root@daniel ~]# docker run -it -p 8000:80 --name=httpd1 centos:latest /bin/bash
[root@b0a9623d3920 /]# yum install httpd httpd-devel -y
Failed to get D-Bus connection: Operation not permitted
启动服务.这里用systemctl start httpd启动服务会报错，所以直接使用命令启动
[root@b0a9623d3920 /]# httpd -k start
[root@b0a9623d3920 /]# ss -an |grep :80
tcp    LISTEN     0      0        :::80                   :::*
~~~

2, 这里如果exit退出的话,启动的服务也会关闭。同时按下**==ctrl+p+q==**三键，可以实现退出容器并保持容器**后台运行**

~~~powershell
可以查看到容器仍然是UP状态
[root@daniel ~]# docker ps -l
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                  NAMES
b0a9623d3920        centos:latest       "/bin/bash"         1 minutes ago      Up 1 minutes       0.0.0.0:8000->80/tcp   httpd1
~~~

3, 使用另一台机器浏览器访问 **http://宿主机IP:8000**测试

![image-20250315145230743](images/image-20250315145230743.png)



### **案例2**: 自定义httpd并提交为镜像（重点）

cenots镜像里并没有httpd,所以需要安装.但是如果每次启动一个容器都要安装一遍httpd是让人受不了的.所以我们在一个容器里安装一次,把想自定义的全做了,然后将此容器commit成一个新的镜像。

以后就用这个新镜像运行容器就可以不用再重复装环境了。

1, 运行容器httpd2,安装httpd相关软件并自定义配置

~~~powershell
[root@daniel ~]# docker run -it --name=httpd2 centos:latest /bin/bash
[root@82b985aea72c /]# yum install httpd httpd-devel -y

[root@82b985aea72c /]# mkdir /www
[root@82b985aea72c /]# echo "main page" > /www/index.html

修改124行和136行的家目录为/www
[root@82b985aea72c /]# vi /etc/httpd/conf/httpd.conf
[root@82b985aea72c /]# exit
exit
~~~

> CentOS8镜像：https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.534d1b11mocgfv

2, exit退出后此容器就变为了Exited状态

~~~powershell
[root@daniel ~]# docker ps -l
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
82b985aea72c        centos:latest       "/bin/bash"         5 minutes ago      Exited (0) 6 seconds ago                       httpd2 
~~~

3, 将搭建好的环境commit成新的镜像(**此镜像相当于是自定义的,生产环境中可以按需求push到镜像仓库**)

~~~powershell
[root@daniel ~]# docker commit httpd2 httpd_image
~~~

4, 将commit提交的镜像启动一个新的容器,并将端口80映射到宿主机的8001

~~~powershell
[root@daniel ~]# docker run -d -p 8001:80 --name=httpd3 httpd_image /usr/sbin/httpd -D FOREGROUND
dcaca836b94655364749c064519ad66c8229657262465e7ea8194f2616980b61
[root@daniel ~]# lsof -i:8001
COMMAND     PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
docker-pr 23130 root    4u  IPv6 183572      0t0  TCP *:vcom-tunnel (LISTEN)
~~~

5, 使用另一台机器浏览器访问 **http://宿主机IP:8001**测试

问题: `/usr/sbin/httpd -D FOREGROUND`能否做成自动传参?

答案: 可以，在dockerfile构建镜像章节会讨论。

扩展：通过此案例，我们将来可以在centos镜像的基础上，安装我们所需的各种环境，如nginx、tomcat、prometheus+grafana、elk环境到容器中，然后打包成镜像，这样以后我们只需运行镜像就可以快速完成应用安装部署了！

---

答疑：为什么Apache只能前台运行？

想象一下，Docker容器就像一个迷你电脑，它有一个重要特性：`容器的生命周期与主进程绑定。`

容器的生命规则很简单：

容器内的主进程活着 → 容器就活着

主进程结束了 → 容器也就结束了

后台运行 vs 前台运行

举个栗子：假设你在容器里启动一个网站服务：

后台运行方式：

```powershell
systemctl start nginx
```

**问题**：服务被放到后台，主进程（启动命令）立即结束，容器也就立刻停止了！

前台运行方式：

```powershell
nginx -g "daemon off;"  # 服务在前台运行，不返回命令行
```

**好处**：服务变成了主进程，只要服务在运行，容器就会继续运行。

通俗比喻

把容器想象成一个房间：

- 前台运行：你站在房间里维持灯亮着
- 后台运行：你按下灯的开关后离开房间，但Docker会认为"房间里没人了"而关闭整个房间

其他好处

- **查看日志更方便**：前台运行的程序，日志直接输出到容器的标准输出
- **排查问题更简单**：可以直接用`docker logs`查看
- **符合Docker设计理念**：一个容器只运行一个应用

---



### **案例3**: docker数据卷挂载

作用：保证Docker容器结束后，里面的数据不会随之消失，实现数据持久化存储。

问题: 当我有如下需求:

* 容器内配置文件需要修改
* 容器内数据(如: 如httpd家目录内的数据)需要保存
* 不同容器间数据需要共享(如: 两个httpd容器家目录数据共享)

当容器删除时,里面的相关改变的数据也会删除,也就是说数据不能持久化保存。

我们可以将服务的配置文件,数据目录,日志目录等与宿主机的目录映射，把数据保持到宿主机上实现**==数据持久化==**.

宿主机的目录也可以共享给多个容器使用。 

**将宿主机的目录(数据卷)挂载到容器中(配置文件也可以挂载)**

1,先在宿主机创建一个目录，并建立一个内容不同的主页

~~~powershell
[root@daniel ~]# mkdir /docker_www
[root@daniel ~]# echo daniel > /docker_www/index.html
~~~

2,运行容器httpd4, 将宿主机的/docker_www/目录挂载到容器中的/www/目录

~~~powershell
[root@daniel ~]# docker run -d -p 8002:80 -v /docker_www/:/www --name=httpd4 httpd_image /usr/sbin/httpd -D FOREGROUND

[root@daniel ~]# docker ps -l
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                  NAMES
484d7432d7ef        httpd_image         "/usr/sbin/httpd -..."          21 seconds ago      Up 20 seconds       0.0.0.0:8002->80/tcp   httpd4
~~~

3, 使用另一台机器浏览器访问 **http://宿主机IP:8002**测试

(注意: 如果访问不到主页,请检查是否关闭了selinux)

4, 尝试修改宿主机/docker_www/index.html的内容, 访问的结果也会随着修改而改变



**拓展:**

默认的centos镜像时间与我们差8小时, 所以这是时区的差异，可以用如下方式解决

~~~powershell
# docker run -it -v /etc/localtime:/etc/localtime --name c2 centos /bin/bash
~~~

说明:

* 因为我们是从docker官方pull下来的centos:latest镜像,它默认为UTC时区
* 我们需要改成自己的时区,可以在启动容器时用`-v /etc/localtime:/etc/localtime`挂载映射
* 如果你觉得每次都要挂载时区文件很麻烦,可以自定义把时区文件改好,保持为新的镜像再使用



~~~powershell
# docker run -d  --name httpd1 -p 8000:80 \
> -v /httpd_www:/www  -v /test/httpd.conf:/etc/httpd/conf/httpd.conf \
> httpd_image /usr/sbin/httpd -DFOREGROUND
~~~

说明: 

* /httpd_www/目录可以不用提前创建,它会自动帮我们创建
* /test/httpd.conf此文件需要提前准备(配置文件里的家目录要改为/www)

* 挂载后,通过修改宿主机的数据来达到修改容器内部数据的目地





### 案例4: 官方httpd镜像运行容器

参考: https://hub.docker.com/_/httpd

1, pull官方httpd镜像

~~~powershell
[root@daniel ~]# docker search httpd
[root@daniel ~]# docker pull httpd
~~~

2, 运行容器

~~~powershell
/data/www目录可以提前创建，也可以不用创建(它会帮我们自动创建)
[root@daniel ~]# mkdir -p /data/www
[root@daniel ~]# docker run -dit --name=httpd4 -p 8003:80 -v /data/www:/usr/local/apache2/htdocs/ httpd:2.4
说明：最后什么时候添加/bin/bash? 答：开发调试环境，一般要添加；如果是上线部署，一般很少添加
docker run -it + /bin/bash
docker run -d 可以不写

[root@daniel ~]# echo "new page" > /data/www/index.html
~~~

3, 使用另一台机器浏览器访问 **http://宿主机IP:8003**测试





## 容器中运行mysql或mariadb应用

说明：虽然本案例是以mysql为例，但是生产环境，一律禁止使用docker运行数据库！！！

### 案例1:官方mysql镜像运行容器

1, 先拉取mysql镜像

~~~powershell
[root@daniel ~]# docker pull mysql:8.0.41
~~~

2, 启动容器

更多参数和详细说明请参考: https://hub.docker.com/_/mysql

~~~powershell
--restart=always表示重启docker服务后会自动重启
-e MYSQL_ROOT_PASSWORD=123 指定mysql的root用户密码

[root@daniel ~]# docker run -d -p 3306:3306 --name=mysql1 -v /data/mysql:/var/lib/mysql --restart=always -e MYSQL_ROOT_PASSWORD=123 mysql:8.0.41
 
启动后查看宿主机的/data/mysql/目录,发现已经初始化数据了
[root@daniel ~]# ls /data/mysql/
auto.cnf  ibdata1  ib_logfile0  ib_logfile1  mysql  performance_schema
~~~

3,  连接上去，按需求自由使用

~~~powershell
[root@daniel ~]# yum install mysql -y
[root@daniel ~]# mysql -h 127.0.0.1 -u root -p123
~~~



### 案例2:centos镜像自定义mariadb环境

~~~powershell
[root@daniel ~]# docker run -it -d -p 3307:3306 -v /data/mysql2:/var/lib/mysql --restart=always --name=mariadb2 centos:latest /bin/bash

[root@daniel ~]# docker attach mariadb2

[root@7dccf1c72315 /]# yum install mariadb-server -y
[root@7dccf1c72315 /]# mysql_install_db --datadir=/var/lib/mysql/ --user=mysql
[root@7dccf1c72315 /]# mysqld_safe --defaults-file=/etc/my.cnf &
[root@7dccf1c72315 /]# mysql
MariaDB [(none)]> grant all on *.* to 'abc'@'%' identified by '123';
MariaDB [(none)]> flush privileges;
MariaDB [(none)]> quit
[root@7dccf1c72315 /]#
最后按ctrl+P+Q退出并保持容器后台运行
~~~

2, 找另一台远程连接测试OK

~~~powershell
# mysql -h docker宿主机IP -u abc -p123 -P 3307
~~~



## 容器中运行nginx应用

参考: https://hub.docker.com/_/nginx

### 案例1: 官方nginx镜像运行容器

1, pull拉取镜像

~~~powershell
[root@daniel ~]# docker pull nginx
~~~

2, 准备一个nginx.conf配置文件

~~~powershell
[root@daniel ~]# mkdir /data/nginx/etc -p
准备下面的配置文件(自己写一个，或者拷贝一个都可以)
[root@daniel ~]# ls /data/nginx/etc/nginx.conf
~~~

3, 运行容器,并准备一个主页文件

~~~powershell
[root@daniel ~]# docker run -d -p 8004:80 --restart=always --name=nginx1 -v /data/nginx/html:/usr/share/nginx/html -v /data/nginx/etc/nginx.conf:/etc/nginx/nginx.conf -v /data/nginx/log:/var/log/nginx nginx:latest

[root@daniel ~]# echo "nginx main page" > /data/nginx/html/index.html
~~~

4,  使用另一台机器浏览器访问 **http://宿主机IP:8003**测试

5，如果想要修改nginx配置文件,可以按下面步骤来实现

~~~powershell
按需求先修改配置文件
[root@daniel ~]# vim /data/nginx/etc/nginx.conf
再重启容器
[root@daniel ~]# docker stop nginx1
[root@daniel ~]# docker start nginx1
~~~



**练习:** 请使用centos镜像自定义nginx环境



## 容器中运行tomcat应用

参考: https://hub.docker.com/_/tomcat

### 案例1: 官方tomcat镜像运行容器

~~~powershell
[root@daniel ~]# docker run -d -p 8888:8080 --name=tomcat9 tomcat:9.0
[root@daniel ~]# docker ps -a
[root@daniel ~]# docker exec -it tomcat9 /bin/bash
ll webapps
rm -rf webapps
mv webapps.dist webapps
~~~

访问http://`docker宿主机IP`:8888验证



**课后兴趣练习: 搜索官方各种应用镜像,并应用**



### 扩展：docker cp命令

作用：实现宿主机与容器之间文件的交换操作（上传文件、下载文件）

复制宿主机文件到容器

```
docker cp readme.txt httpd1:/root/
```

再在容器中的问及那到本地

```
docker cp httpd1:/root/readme.txt /root/
```



# 十、使用Dockerfile构建镜像（重点）

除了使用docker commit把自定义容器提交成镜像外，还可以使用Dockerfile来构建自定义镜像。

> 什么是Dockerfile?
>
> 答:Dockerfile把构建镜像的步骤都写出来,然后按顺序执行实现自动构建镜像。就类似于脚本文件,ansible的playbook,saltstack的sls文件等。

## dockerfile核心命令速查表

docker安装 => docker pull一个镜像一个镜像拉取 => docker run => 容器  => 配置操作 => 最终容器 => 导出为最终镜像

docker安装 => dockerfile => 构建最终镜像

| **命令**      | **作用**                                         | **Nginx 典型示例**                      |
| ------------- | ------------------------------------------------ | --------------------------------------- |
| **`FROM`**    | 指定基础镜像                                     | `FROM nginx:1.25-alpine`（推荐轻量版）  |
| **`WORKDIR`** | 设置容器工作目录（非必须，但便于管理）           | `WORKDIR /etc/nginx`                    |
| **`COPY`**    | 复制本地文件到容器（覆盖默认配置或添加静态资源） | `COPY nginx.conf /etc/nginx/nginx.conf` |
| **`RUN`**     | 执行命令（安装依赖、修改默认行为）               | `RUN yum install curl`                  |
| **`EXPOSE`**  | 声明容器监听端口（需配合`-p`映射到宿主机）       | `EXPOSE 80 443`                         |
| **`VOLUME`**  | 挂载数据卷（持久化日志或静态文件）               | `VOLUME ["/var/log/nginx"]`             |
| **`CMD`**     | 启动Nginx（通常不需修改，直接用基础镜像命令）    | `CMD ["nginx", "-g", "daemon off;"]`    |

四句真诀：

```powershell
一、FROM基础起，WORKDIR定目录      【基础环境】
二、COPY放文件，RUN执构建命        	  【代码准备】 
三、ENV设环境，EXPOSE开端口             【运行配置】
四、CMD启动程，镜像构建成                   【启动命令】
```

## dockerfile指令

通过`man dockerfile`可以查看到详细的说明,我这里简单的翻译并列出常用的指令

1, **FROM**

FROM指令用于指定其后构建新镜像所使用的基础镜像。

FROM指令必是Dockerfile文件中的首条命令。

FROM指令指定的基础image可以是官方远程仓库中的，也可以位于本地仓库，优先本地仓库。

```powershell
格式:FROM <image>:<tag>
例:FROM centos:latest
```

2, **RUN**

RUN指令用于在**构建**镜像中执行命令，有以下两种格式:

* shell格式

~~~powershell
格式:RUN <命令>
例:RUN echo daniel > /var/www/html/index.html
~~~

* exec格式

~~~powershell
格式:RUN ["可执行文件", "参数1", "参数2"]
例:RUN ["/bin/bash", "-c", "echo daniel > /var/www/html/index.html"]
~~~

**注意:** 按优化的角度来讲:当有多条要执行的命令,不要使用多条RUN,尽量使用&&符号与\符号连接成一行。因为多条RUN命令会让镜像建立多层(总之就是会变得臃肿了:smiley:)。

~~~powershell
RUN yum install httpd httpd-devel -y
RUN echo daniel > /var/www/html/index.html
可以改成
RUN yum install httpd httpd-devel -y && echo daniel > /var/www/html/index.html
或者改成
RUN yum install httpd httpd-devel -y  \
    && echo daniel > /var/www/html/index.html
~~~

3, **CMD**

CMD不同于RUN,CMD用于指定在容器启动时所要执行的命令,而RUN用于指定镜像构建时所要执行的命令。

~~~powershell
格式有三种:
CMD ["executable","param1","param2"]
CMD ["param1","param2"]
CMD command param1 param2
~~~

每个Dockerfile只能有一条CMD命令。如果指定了多条命令，只有最后一条会被执行。

如果用户启动容器时候指定了运行的命令，则会覆盖掉CMD指定的命令。

~~~powershell
什么是启动容器时指定运行的命令?
# docker run -d -p 80:80 镜像名 运行的命令
~~~

4, **EXPOSE**

EXPOSE指令用于指定容器在运行时监听的端口

~~~powershell
格式:EXPOSE <port> [<port>...]
例:EXPOSE 80 3306 8080
~~~

上述运行的端口还需要使用docker run运行容器时通过-p参数映射到宿主机的端口.

5, **ENV**

ENV指令用于指定一个环境变量. 

~~~powershell
格式:ENV <key> <value> 或者 ENV <key>=<value>
例:ENV JAVA_HOME /usr/local/jdkxxxx/
~~~

6, **ADD**

ADD指令用于把宿主机上的文件拷贝到镜像中

~~~powershell
格式:ADD <src> <dest>
<src>可以是一个本地文件或本地压缩文件，还可以是一个url,
如果把<src>写成一个url，那么ADD就类似于wget命令
<dest>路径的填写可以是容器内的绝对路径，也可以是相对于工作目录的相对路径
~~~

7, **COPY**

COPY指令与ADD指令类似,但COPY的源文件只能是本地文件（文件或目录）

~~~powershell
格式:COPY <src> <dest>
~~~

ADD与COPY最佳实践：

默认优先使用COPY

仅当需要解压本地压缩包时使用

```powershell
ADD app.tar.gz /app/
等价于
COPY app.tar.gz /app/ && tar -xzf /app/app.tar.gz -C /app/
```



8, **ENTRYPOINT**

ENTRYPOINT与CMD非常类似

相同点：
一个Dockerfile只写一条，如果写了多条，那么只有最后一条生效
都是容器启动时才运行

不同点：
如果用户启动容器时候指定了运行的命令，ENTRYPOINT不会被运行的命令覆盖，而CMD则会被覆盖

~~~powershell
格式有两种:
ENTRYPOINT ["executable", "param1", "param2"]
ENTRYPOINT command param1 param2
~~~

9,  **VOLUME**

VOLUME指令用于把宿主机里的目录与容器里的目录映射. 

只指定挂载点,docker宿主机映射的目录为自动生成的。

~~~powershell
格式:VOLUME ["<mountpoint>"]
~~~

10, **USER**

USER指令设置启动容器的用户(像hadoop需要hadoop用户操作，oracle需要oracle用户操作),可以是用户名或UID

~~~powershell
USER daemon
USER 1001
~~~

**注意**：如果设置了容器以daemon用户去运行，那么RUN,CMD和ENTRYPOINT都会以这个用户去运行
镜像构建完成后，通过docker run运行容器时，可以通过-u参数来覆盖所指定的用户

11, **WORKDIR**

WORKDIR指令设置工作目录,类似于cd命令。不建议使用`RUN cd /root` ,建议使用WORKDIR

~~~powershell
WORKDIR /root
~~~



**步骤：**

1、创建一个文件夹（目录）

2、在文件夹（目录）中创建Dockerfile文件(并编写)及其它文件

3、使用`docker build`命令构建镜像

4、使用构建的镜像启动容器



## 案例1:Dockerfile构建httpd镜像v1

1, 准备一个目录(自定义)

~~~powershell
[root@daniel ~]# mkdir /dockerfile
~~~

2, 编写dockerfile

~~~powershell
[root@daniel ~]# cd /dockerfile
[root@daniel dockerfile]# vim dockerfile_httpd 
FROM cf-workers-docker-io-38g.pages.dev/dokken/centos-stream-9

LABEL maintainer="daniel <daniel@itcast.cn>"

RUN yum install httpd httpd-devel -y \
	&& echo "container main page" > /var/www/html/index.html

EXPOSE  80
CMD ["/usr/sbin/httpd","-D","FOREGROUND"]
~~~

3, 使用`docker build`构建镜像,注意最后有一个点(代表当前目录)

~~~powershell
[root@daniel dockerfile]# docker build -f dockerfile_httpd -t my_httpd:v1 .
-f:file ,指定构建的dockerfile文件
-t:target。目标镜像文件名称及版本号（tag标签）
. : 最后一个点代表基于当前目录作为构建的上下文环境
~~~

4, 验证镜像

~~~powershell
[root@daniel dockerfile]# docker images |grep my_httpd
my_httpd            v1                  e316739796ae        1 minutes ago       348 MB
~~~

5, 使用构建好的镜像创建容器

~~~powershell
[root@daniel dockerfile]# docker run -d -p 8005:80 my_httpd:v1

[root@daniel dockerfile]# docker ps -a |grep my_httpd
c539e6161463        my_httpd:v1         "/usr/sbin/httpd -..."   1 minutes ago       Up 8 minutes        0.0.0.0:8005->80/tcp   boring_goldstine
~~~

6, 客户端访问http://`docker宿主机IP`:8005测试

## 案例2:Dockerfile构建httpd镜像v2

1, 编写dockerfile

~~~powershell
[root@daniel dockerfile]# vim dockerfile_httpd2 
FROM cf-workers-docker-io-38g.pages.dev/dokken/centos-stream-9

LABEL maintainer="daniel <daniel@itcast.cn>"

RUN yum install httpd httpd-devel -y

VOLUME ["/var/www/html/"]

EXPOSE  80
CMD ["/usr/sbin/httpd","-D","FOREGROUND"]
~~~

2, 使用`docker build`构建镜像

~~~powershell
[root@daniel dockerfile]# docker build -f dockerfile_httpd2 -t my_httpd:v2 .
~~~

3, 验证镜像

~~~powershell
[root@daniel dockerfile]# docker images |grep my_httpd |grep v2
my_httpd            v2                  3146d5503f39        1 minutes ago         382MB
~~~

4, 使用构建好的镜像创建容器

~~~powershell
将宿主机上的/data/www2/挂载到容器里的/var/www/html/
[root@daniel dockerfile]# docker run -d -p 8006:80 -v /data/www2:/var/www/html my_httpd:v2

[root@daniel dockerfile]# echo httpd2 > /data/www2/index.html
~~~

思考: 我们使用了`-v /data/www2:/var/www/html`挂载了数据卷,那么`VOLUME ["/var/www/html/"]`的作用体现在哪里?

~~~powershell
没有手动-v指定数据卷挂载,则docker会在宿主机/var/lib/docker/volumes/产生一个卷目录挂载到你指定的目录
~~~



5, 客户端访问http://`docker宿主机IP`:8006测试



## 案例3: Dockerfile构建tomcat镜像

1, 编写新的dockerfile

~~~powershell
[root@daniel dockerfile]# vim dockerfile_tomcat

FROM cf-workers-docker-io-38g.pages.dev/dokken/centos-stream-9

LABEL maintainer="daniel <daniel@itcast.cn>"

COPY jdk-8u191-linux-x64.tar.gz .
COPY apache-tomcat-9.0.97.tar.gz .

RUN tar xf jdk-8u191-linux-x64.tar.gz -C /usr/local &&  \
    tar xf apache-tomcat-9.0.97.tar.gz -C /usr/local && \
    mv /usr/local/apache-tomcat-9.0.97 /usr/local/tomcat && \
    rm -rf jdk-8u191-linux-x64.tar.gz && \
    rm -rf apache-tomcat-9.0.97.tar.gz && \
    sed -i 1a"export JAVA_HOME=/usr/local/jdk1.8.0_191" /usr/local/tomcat/bin/catalina.sh

EXPOSE  8080
CMD /usr/local/tomcat/bin/catalina.sh run
~~~

2, 使用`docker build`构建镜像

~~~powershell
[root@daniel dockerfile]# docker build -f dockerfile_tomcat -t my_tomcat:v1 .
~~~

3, 验证镜像

~~~powershell
[root@daniel dockerfile]# docker images |grep tomcat |grep v1
my_tomcat              v2                  b9b1aae7a02f        5 minutes ago       815MB
~~~

4, 使用构建好的镜像创建容器

~~~powershell
[root@daniel dockerfile]# docker run -d -p 8082:8080 --ulimit nofile=65536:65536 --name=tomcat1 my_tomcat:v1
或者
[root@daniel dockerfile]# docker run -d --name tomcat3 -v /data/tomcat_data:/usr/local/tomcat/webapps/ROOT -p 8082:8080 my_tomcat:v1

ps：

~~~

![Snipaste_2025-10-15_17-24-42](images\Snipaste_2025-10-15_17-24-42.png)

![Snipaste_2025-10-15_17-25-51](images\Snipaste_2025-10-15_17-25-51.png)

5, 客户端访问http://`docker宿主机IP`:8082测试



**改进**

解压后的jdk与tomcat软件包删除也不会使镜像大小变小(因为overlay文件系统的特性),所以这里就不再删除了

环境变量可以使用ENV来指定

~~~powershell
[root@daniel dockerfile]# vim dockerfile_tomcat2

FROM cf-workers-docker-io-38g.pages.dev/dokken/centos-stream-9

LABEL maintainer="daniel <daniel@itcast.cn>"

ENV JAVA_HOME=/usr/local/jdk1.8.0_191

COPY jdk-8u191-linux-x64.tar.gz .
COPY apache-tomcat-9.0.14.tar.gz .

RUN tar xf jdk-8u191-linux-x64.tar.gz -C /usr/local &&  \
    tar xf apache-tomcat-9.0.14.tar.gz -C /usr/local && \
    mv /usr/local/apache-tomcat-9.0.14 /usr/local/tomcat

EXPOSE  8080
CMD /usr/local/tomcat/bin/catalina.sh run
~~~

**再次改进**

需要提前在宿主机上解压jdk和tomcat

~~~powershell
FROM cf-workers-docker-io-38g.pages.dev/dokken/centos-stream-9

LABEL maintainer="daniel <daniel@itcast.cn>"

ENV JAVA_HOME=/usr/local/jdk1.8.0_191

ADD jdk1.8.0_191 /usr/local/jdk1.8.0_191
ADD tomcat /usr/local/tomcat

EXPOSE 8080
CMD /usr/local/tomcat/bin/catalina.sh run
~~~



## 案例4: Dockerfile构建mariadb镜像

1, 准备1个脚本执行mysql的初始化与启动

~~~powershell
[root@daniel dockerfile]# vim mariadb.sh

#!/bin/bash

mysql_install_db --datadir=/var/lib/mysql/ --user=mysql
sleep 3
mysqld_safe --defaults-file=/etc/my.cnf &
sleep 3

mysql -e "grant all privileges on *.* to 'root'@'%' identified by '123';"
mysql -e "grant all privileges on *.* to 'abc'@'%' identified by '123';"
mysql -e "flush privileges;"
~~~

说明: 

* 使用脚本而不直接使用dockerfile里的RUN指令的原因是: 启动mysql服务需要使用`&`放到后台,但把后台符号放在RUN里会造成RUN命令有问题,所以单独使用脚本来做
* sleep 3秒是因为初始化和启动服务需要一定的时间,等待3秒缓冲一下

2, 创建dockerfile

~~~powershell
[root@daniel dockerfile]# vim dockerfile_mariadb
FROM cf-workers-docker-io-38g.pages.dev/dokken/centos-stream-9

LABEL maintainer="daniel <daniel@itcast.cn>"

RUN yum install mariadb-server mariadb -y

COPY mariadb.sh .
RUN sh mariadb.sh

EXPOSE 3306
CMD mysqld_safe --defaults-file=/etc/my.cnf
~~~

3, `docker build`

~~~powershell
[root@daniel dockerfile]# docker build -f dockerfile_mariadb -t my_mariadb:v1 .
~~~

4, 使用build的镜像启动容器

~~~powershell
[root@daniel dockerfile]# docker run -d -p 3306:3306 --name mariadb2 my_mariadb:v1
~~~

## dockerfile最佳实践（获取一个最小镜像）

① 构建层最小化：合并RUN指令减少镜像层数

② 缓存优化：将变化少的指令放在前面 

③ 最小安装：只安装必要的包和依赖

④ 使用特定标签：避免使用latest标签

⑤ 多阶段构建：分离构建环境和运行环境

⑥ 非root用户：使用最小权限原则

⑦ 合理排序：依赖安装在代码复制前  

⑧ 使用.dockerignore：排除不需要的文件

⑨ 注意安全更新：定期更新基础镜像

## 十一、docker容器资源限制

![Snipaste_2025-10-15_22-35-00](images\Snipaste_2025-10-15_22-35-00.png)



![Snipaste_2025-10-15_22-37-06](images\Snipaste_2025-10-15_22-37-06.png)



![Snipaste_2025-10-15_22-37-53](images\Snipaste_2025-10-15_22-37-53.png)

![Snipaste_2025-10-15_22-39-07](images\Snipaste_2025-10-15_22-39-07.png)

![Snipaste_2025-10-15_22-40-28](images\Snipaste_2025-10-15_22-40-28.png)

![Snipaste_2025-10-15_22-41-13](images\Snipaste_2025-10-15_22-41-13.png)

![Snipaste_2025-10-15_22-42-25](images\Snipaste_2025-10-15_22-42-25.png)

![Snipaste_2025-10-15_22-43-36](images\Snipaste_2025-10-15_22-43-36.png)

![Snipaste_2025-10-15_22-44-27](images\Snipaste_2025-10-15_22-44-27.png)

![Snipaste_2025-10-15_22-45-33](images\Snipaste_2025-10-15_22-45-33.png)

# 十二、docker的web管理平台

我想初学者都被docker的复杂命令搞得晕头转向了,希望有一个图形化的管理平台能轻松管理容器。类似的开源web管理平台主要有: DockerUI,Portainer,Shipyard等。



## DockerUI

1,拉取dockerui的镜像

~~~powershell
[root@daniel ~]# docker pull uifd/ui-for-docker
~~~

2,运行容器

**注意:**需要将docker宿主机的`/var/run/docker.sock`与容器的`/var/run/docker.sock`对应,才能管理

~~~powershell
[root@daniel ~]# docker run -d --name dockerui -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock uifd/ui-for-docker
23112ee4132b974af2647762e155592da00ab30def797953cb03b1bcade18434

[root@daniel ~]# lsof -i:9000
COMMAND     PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
docker-pr 13154 root    4u  IPv6 178230      0t0  TCP *:cslistener (LISTEN)
~~~

3,使用浏览器访问 http://`docker宿主机IP`:9000

![1545584320723](./images/DockeUI.png)



## portainer

1, 拉取portainer镜像

~~~powershell
[root@daniel ~]# docker pull portainer/portainer
~~~

2,  运行容器

**注意:**需要将docker宿主机的`/var/run/docker.sock`与容器的`/var/run/docker.sock`对应,才能管理

~~~powershell
[root@daniel ~]# docker run -d -p 9001:9000 --name=portainer -v /var/run/docker.sock:/var/run/docker.sock  portainer/portainer

[root@daniel ~]# lsof -i:9001
COMMAND     PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
docker-pr 70010 root    4u  IPv6 293808      0t0  TCP *:etlservicemgr (LISTEN)
~~~

使用浏览器访问 **http://宿主机IP:9001**

![1545585050659](./images/portainer.png)

![1545585284908](./images/portainer2.png)

![1563973346416](./images/portainer3.png)



![Snipaste_2025-10-15_22-56-16](images\Snipaste_2025-10-15_22-56-16.png)
