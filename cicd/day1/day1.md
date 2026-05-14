## CICD：持续集成 持续部署

```
项目需求---任务下发---功能开发--拉取代码----编译构建（java环境）--打包镜像---推送仓库--部署测试环境
ci工具：jenkins gitlab-ci drone（直升机）
```

#### 1、安装java环境：

```
linux：部署java环境
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum upgrade
# Add required dependencies for the jenkins package
sudo yum install fontconfig java-17-openjdk
sudo yum install jenkins
sudo systemctl daemon-reload
[root@www ~]# tar -xf jdk-17_linux-x64_bin.tar.gz -C /usr/local/

安装java环境：
[root@www ~]# ln -s /usr/local/jdk-17.0.10/ /usr/local/java
vi /etc/profile.d/java.sh
export JAVA_HOME=/usr/local/java
export PATH=$PATH:$JAVA_HOME/bin
[root@www ~]# source /etc/profile.d/java.sh 
启动jenkins：jenkins
初始密码：55628de3e8c346568ff0e052a607a273
e297925852f6447ca0ca9086ecfbd9fe
浏览器中输入：192.168.18.100:8080==出现jenkins页面
安装插件：第二个
```

#### 2、安装jenkins：

```
二、授权用户：
全局安全配置---基于角色的插件----role插件---基于角色分配
基于角色：
安装插件
laoda----针对yproject---只能看到y开头的项目

三、master负责调度，真正工作的是slave
创建slave：
node---创建slave1===全局安全配置中勾选tcp---就可以查看node中slave1
101：安装jdk和目录

四、pipeline：流水线
有两种语法：有申明式的和脚本式的Declarative versus Scripted Pipeline syntax，建议使用Declarative（申明）
安装pipeline插件

在101服务器上安装jenkins需要安装Java环境等，新建挂载点mkdir -pv /var/lib/jenkins(存放拉取代代码的地址)
```

#### 3、项目实战之图书管理系统：

```
图书管理系统：
一、拉取代码
1、在github中找到BOOK MANAGE	拉去代码地址
https://github.com/withstars/Books-Management-System.git
yum install git -y
101中拉取镜像：[root@www ~]# git clone https://mirror.ghproxy.com/https://github.com/withstars/Books-Management-System.git
或者使用jenkins拉取代码：test2----配置---流水线语法----版本控制（checkout：...control）---Git---复制BOOKurl地址----生成流水线脚本----复制到pipeline上面的拉取代码上--立即构建(拉去代码不成功加代理)

二、编译构建：（需要maven）有两个环境：java17和java8 101
[root@www ~]# tar -xf apache-maven-3.9.4-bin.tar.gz -C /usr/local/
[root@www ~]# ln -s /usr/local/apache-maven-3.9.4/ /usr/local/mvn
vi /etc/profile.d/mvn.sh
export MAVEN_HOME=/usr/local/mvn
export PATH=$PATH:$MAVEN_HOME/bin
给mvn添加国内镜像：
vi /usr/local/mvn/conf/settings.xml
		<mirror>
            <id>alimaven</id>
            <name>aliyun maven</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
            <mirrorOf>*</mirrorOf>
        </mirror>
pipeline：拉取代码已经成功（先注释掉）上面编译构建(在101上面重启一下jenkins)====》sh 'mvn clean package'===修改Java环境（编译构建需要使用java8的环境，修改java_home环境）===流水线语法中找到withEnv:...variables====语法：JAVA_HOME=/usr/local/java8====生成流水线语法复制到编译构建上

三、部署：（使用jenkins和ansible部署测试）部署java8的war包
3.1部署ansible：101
[root@www ~]# yum install epel-release -y
[root@www ~]# yum install ansible -y
3.2 ansible要相互信任：
ssh-keygen -t rsa
[root@www ~]# ssh-copy-id 192.168.18.102
[root@www ~]# vi /etc/ansible/hosts
[webserver]
192.168.18.102
以上101服务器

102服务器：安装java和tomcat(和100服务器一样，复制过去scp----jdk和java.sh)
安装tomcat
vi /etc/profile.d/tomcat.sh
export CATALINA_HOME=/usr/local/tomcat
export PATH=$PATH:$CATALINA_HOME/bin
source /etc/profile.d/tomcat.sh
catalina.sh start
浏览器中输入：192.168.18.102：8080===出现tomcat页面

101：写剧本
cd /var/lib/jenkins/workspace/test2/
vi play1.yaml
- hosts: webservers
  tasks:
  - name: deploy war
    copy:
      src: target/book-1.0-SNAPSHOT.war
      dest: /usr/local/tomcat/webapps/ROOT.war
[root@www test2]# ansible-playbook play1.yaml

102：删除ROOT*
[root@www ~]# rm -rf /usr/local/tomcat/webapps/ROOT*
部署到jenkins中

遇到的困难1：jenkins需要java17的环境，当部署一个Java应用的时候需要java8环境，在pipeline语法时需要临时输出java8（withEnv临时修改环境JAVA_HOME=/usr/local/java8）最后才运行成功
```

