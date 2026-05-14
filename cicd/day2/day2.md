## jenkins ci工具持续集成（迁移到k8s中）

#### 1、知识回顾：

```
1、发布流水线

拉去代码----编译构建-----ansible    部署测试环境

							打包镜像（deploy svc ingress pv pvc来部署） 

2、回滚流水线
```

#### 2、将jenkins部署到k8s中：

```
同步：systemctl restart chronyd
chronyc sources -v
secret：8b4f5788a8c240b0892f73eb44d906aa
将jenkins部署到k8s中
101和102的8java关闭
102：
[root@www ~]# mkdir -pv /data/k8s/jenkins

1、部署pv和pvc在100：jenkins-pv-pvc.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkins-local
  labels:
    app: jenkins
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 5Gi
  storageClassName: local-storage
  local:
    path: /data/k8s/jenkins ====》保存任务或者插件
  persistentVolumeReclaimPolicy: Retain
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - www.y2312node2.com
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-pvc
  namespace: kube-ops
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
[root@www cicd]# kubectl create ns kube-ops
2、部署jenkins：
vi jenkins-deploy-svc.yaml
---
apiVersion: v1=====>授权一个用户
kind: ServiceAccount
metadata:
  name: jenkins 
  namespace: kube-ops
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: jenkins
rules:
....
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins
  namespace: kube-ops
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: jenkins
subjects:
  - kind: ServiceAccount
    name: jenkins
    namespace: kube-ops
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  namespace: kube-ops
spec:
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      serviceAccount: jenkins
      initContainers:
        - name: fix-permissions
          image: busybox: latest
          imagePullPolicy: IfNotPresent
          command: ["sh", "-c", "chown -R 1000:1000 /var/jenkins_home"]
          securityContext:
            privileged: true
          volumeMounts:
            - name: jenkinshome
              mountPath: /var/jenkins_home
      containers:
        - name: jenkins
          image: jenkins/jenkins:2.414.1-lts
          imagePullPolicy: IfNotPresent
          env:
            - name: JAVA_OPTS
              value: "-Djava.awt.headless=true -Dhudson.model.DownloadService.noSignatureCheck=true"
          ports:
            - containerPort: 8080
              name: web
              protocol: TCP
            - containerPort: 50000
              name: agent
    protocol: TCP
          readinessProbe:
            httpGet:
              path: /login
              port: 8080
            initialDelaySeconds: 30
            timeoutSeconds: 5
            failureThreshold: 12
          volumeMounts:
            - name: jenkinshome
              mountPath: /var/jenkins_home
      volumes:
        - name: jenkinshome
          persistentVolumeClaim:
            claimName: jenkins-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins
  namespace: kube-ops
  labels:
    app: jenkins
spec:
  selector:
    app: jenkins
  ports:
    - name: web
      port: 8080
      targetPort: web
    - name: agent
      port: 50000
      targetPort: agent
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: jenkins
  namespace: kube-ops
spec:
  ingressClassName: nginx
  rules:
    - host: jenkins.y2312.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: jenkins
                port:
                  name: web
[root@www cicd]# kubectl apply -f jenkins-pv-pvc.yaml 
[root@www cicd]# kubectl apply -f jenkins-deploy-svc.yaml
```

#### 3、安装jenkins

```
首先win解析域名：jenkins.y2312.com
浏览器中输入：192.168.18.101：8080
安装插件chinese pipeline role kubernetes git、webhook
浏览器中输入：https://hub.docker.com/r/jenkins/inbound-agent------找到jenkins-inbound
1、1 jenkins中创建节点：slave1，动态slave：创建任务时才创建，不创建时就不
1、2 secret：7878cac35c39fce0f19fc2e066a48e44222b80ee048bf193c83eadd35e8e93a5 -
f191cb0d4299b1d58dae515efce6ef3a2e3ef35c8acf3d3175557c0a8d58560f
48756d29b8934212baec802b87821ec2
52e5b664d977317f69db4386704250e61ef751681769d4f1a48dbaac598e75f4
1、3 工作目录：workDir "/home/jenkins/workspace"
====目录：/home/jenkins/workspace--标签：y2312====保存==复制secret：
====打开端口：全局安装配置
```

##### 3.1安装静态slave：

```
vi jenkins-agint.yaml====此时只是静态slave
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins-agent-2
  namespace: kube-ops
spec:
  selector:
    matchLabels:
      app: jenkins-agent-2
  template:
    metadata:
      labels:
        app: jenkins-agent-2
    spec:
      containers:
        - name: agent
          image: jenkins/inbound-agent
          securityContext:
            privileged: true
          imagePullPolicy: IfNotPresent
          env:
            - name: JENKINS_URL
              value: http://jenkins:8080
            - name: JENKINS_SECRET
              value: 52e5b664d977317f69db4386704250e61ef751681769d4f1a48dbaac598e75f4====》secret
            - name: JENKINS_AGENT_NAME
              value: slave1===同jenkins中取得名字一样
            - name: JENKINS_AGENT_WORKDIR
              value: /home/jenkins/workspace
```

##### 3.2安装动态slave：

```
二、动态slave，工作就启动，没有就不启动====》安装插件：插件管理====》kubernetes和pipeline
节点和云管理：新建一个clouds：
1、name：kubernetes
2、地址：应该是api-server的地址，通过svc（在default的名称空间中）暴露出来的，而kubenetes在kube-ops名称空间中(要跨名称空间所以使用https)https://kubernetes.default.svc.cluster.local
ns:kube-ops
3、凭据：无
4、jenkins地址：http://jenkins:8080

使用clouds===》jenkins创建任务：test1===》流水线
创建申明是模板：浏览器中书写
pipeline {
  agent {==创建pod运行的模板
    kubernetes {
      yaml '''===定义pod模板的
        apiVersion: v1
        kind: Pod
        metadata:
          labels:
            some-label: some-label-value
        spec:
          containers:
          - name: busybox
            image: busybox
            command:
            - cat   ==运行在前端
            tty: true
        '''
      retries 2  ===尝试2秒
    }
  }
  stages {
    stage('拉去代码') {
      steps {
        container('busybox') {
          sh 'echo "拉去代码"'
        }
      }
    }
    stage ('编译构建') {
      steps {
        container('busybox') {
          sh 'echo "编译构建"'
    }
  }
}
}
}
k8s中的kube-ops中会跑起来一个busybox容器和jenkins-agent两个容器，前者是动态slave创建的后者创建起来与master相连。
emptyDir生命周期同pod，当pod挂掉，存储卷也会失效，数据也会丢与pod共享数据。
```

#### 4、安装代码仓库gogs

```
2、真正拉取代码：
部署私有的代码仓库：
gitlab（极狐中国版比较重）或者gogs（轻量级，代码管理工具）
gogs.io官网
dockerhub.com官网中搜索：gogs镜像

vi gogs-pv-pvc.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: gogs-local
  labels:
    app: gogs
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 5Gi
  storageClassName: local-storage
  local:
    path: /data/k8s/gogs
  persistentVolumeReclaimPolicy: Retain
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - www.y2312node2.com
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: gogs-pvc
  namespace: kube-ops
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
在102上面node2上面创建挂载点： mkdir -pv /data/k8s/jenkins

创建
vi gogs-deploy-svc.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gogs
  namespace: kube-ops
spec:
  selector:
    matchLabels:
      app: gogs
  template:
    metadata:
      labels:
        app: gogs
    spec:
      initContainers:
        - name: fix-permissions
          image: busybox:1.35.0
          command: ["sh", "-c", "chown -R 1000:1000 /data"]
          securityContext:
            privileged: true
          volumeMounts:
            - name: gogshome
              mountPath: /data
      containers:
        - name: gogs
          image: gogs/gogs:latest
          imagePullPolicy: IfNotPresent
          ports:
		  - containerPort: 3000
              name: web
              protocol: TCP
          volumeMounts:
            - name: gogshome
              mountPath: /data
      volumes:
        - name: gogshome
          persistentVolumeClaim:
            claimName: gogs-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: gogs
  namespace: kube-ops
  labels:
    app: gogs
spec:
  selector:
    app: gogs
  ports:
    - name: web
      port: 80
      targetPort: web
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: gogs
  namespace: kube-ops
  annotations:
    nginx.ingress.kubernetes.io/proxy-boby-size: 8m
spec:
  ingressClassName: nginx
  rules:
    - host: gogs.y2312.com
      http:
        paths:
          - path: /
            pathType: Prefix
			backend:
              service:
                name: gogs
                port:
                  name: web
安装gogs：浏览器中输入gogs.y2312.com
数据库类型： SQLite3
域名：gogs.y2312.com
应用URL:http://gogs.y2312.com

emptyDIR 生命周期通pod，当pod挂了挂载的也挂了
用户：gogsadmin Aa@123456
拉去代码：===代码放在gogs
解析域名：101上面 192.168.66.91 gogs.y2312.com

```

#### 5、jenkins中写pipeline

```
101服务器：存放代码仓库的服务器
yum install git -y 
vi /etc/hosts
192.168.18.101 gogs.y2312.com
[root@www ~]# mkdir firstproject
[root@www ~]# cd firstproject/
git有一个本地仓库和远程仓库，在本地初始化一个仓库再将它推送到远程仓库中
git init ===>本地初始化一个仓库
touch README.md
git add . =====>将下面所有的文件交给仓库管理
git commit -m "first commit"===确认
推送到远程仓库：git remote add origin(仓库名字) http://gogs.y2312.com/gogsadmin/firstproject.git(远程仓库地址) 
推送：git push -u origin master（master代表分支）
===填写用户密码：gogsadmin Aa@123456
gogs.y2312.com中查看就可以看到将README.md推送上来了

项目：
将图书管理系统拉下来（dockerhub官网拉去）再推送到代码仓库中去（gogs.y2312.com中）
真正部署的项目：
[root@www firstproject]# git clone https://mirror.ghproxy.com/https://github.com/withstars/Books-Management-System.git
[root@www ~]# cd Books-Management-System/
添加自己的仓库（在gogs.y2312.com中在新建一个仓库book）：book 
git remote add origin(仓库名字) http://gogs.y2312.com/gogsadmin/firstproject.git(远程仓库地址)
推送不了（BOOK图书管理系统太大了，发送不了报错为：send too large chunken boby,部署的时候选用ingress-nginx做反向代理到gogs（代码仓库中gogs-dep-svc-ing中增加注解））====增加一个注解（100服务器中cicd）：vi gogs-deploy-svc.yaml
 annotations:
    nginx.ingress.kubernetes.io/proxy-boby-size: "100m"
101： git push -u origin master===》重新提交

错误二：写的代码推送仓库时报错send too large chunken boby为代码过大推送不了到代码仓库，可以在ingress-nginx中增加一个注解
vi gogs-dep-svc-ingress.yaml中
annotations:
    nginx.ingress.kubernetes.io/proxy-boby-size: 8m
```

#### 6、k8s中修改代码

```
1、拉去代码的镜像：
hub.docker.com官网： 找docker
docker in docker：在docker内部再跑一个docker引擎
查看流水线语法：选择checkout：....control，SCM：选择插件git，URL:gogs.y2312.com中仓库BOOK图书馆系统的地址，然后生成流水线语法复制到配置拉取代码的步骤中
安装插件git
jenkins中coredns此时识别不了gogs.y2312.com
#进入pod内部：kubectl -n kube-system edit configmaps coredns
jenkins要解析域名：k9s----kube-system----configmaps----coredns（e）---192.168.18.101 gogs.y2312.com===重启一下pod
例如：
hosts {
	192.168.18.101 gogs.y2312.com
	fallthrough
}
此时会跑起来三个pod，一个jenkins-agent回去联系master，一个docker当中的git，一个busybox
然后再jenkins拉取镜像====特权模式===在jenkinspipeline语法中git镜像下面添加一个特权模式
securityContext:
	privileged: true
进入一个pod busybxy时查看/home/jenkins/agent/workspace/test1此时代码已经拉取下来了。

语法：

2、编译构建：准备maven===101服务器上
[root@www ~]# mkdir mvndocker
[root@www ~]# cd mvndocker/
[root@www mvndocker]# cp /root/jdk-8u144-linux-x64.tar.gz ./
[root@www mvndocker]# cp /root/apache-maven-3.9.4-bin.tar.gz ./
[root@www mvndocker]# cp /usr/local/mvn/conf/settings.xml ./
[root@www mvndocker]# cp /etc/profile.d/mvn.sh ./
[root@www mvndocker]# cp /etc/profile.d/java.sh ./
vi Dockerfile
FROM centos:7
ADD jdk-8u144-linux-x64.tar.gz /usr/local/==提供java和maven并解压
ADD apache-maven-3.9.4-bin.tar.gz /usr/local/
RUN ln -s /usr/local/jdk1.8.0_144 /usr/local/java===符号链接
RUN ln -s /usr/local/apache-maven-3.9.4 /usr/local/mvn
ENV JAVA_HOME=/usr/local/java MAVEN_HOME=/usr/local/mvn PATH=$PATH:/usr/local/mvn/bin:/usr/lcoal/java/bin ===创建环境变量
==提供环境
ADD settings.xml /usr/local/mvn/conf/

docker build . -t mvn:v1
docker run -it --rm mvn:v1 /bin/bash
打包镜像：===需要dockerhub仓库
[root@www mvndocker]# docker save mvn:v2 -o mvn.tar.gz====压缩
[root@www mvndocker]# scp mvn.tar.gz 192.168.18.102:/root/
102中解压：[root@www ~]# docker load -i mvn.tar.gz

jenkins中在指定一个打包镜像的容器：
- name: mvn
  image: mvn:v2
  command:(需要运行在前台)
  - cat:
    tty: true

步骤中编译构建中需要：
stage('编译构建'){
	steps {
		container('mvn'){
			sh 'mvn clean package'
			sh 'sleep 300'
		}
	}
}

```

>**遇到的困难**：在java环境中编译构建时，手动创建mvn镜像时书写dockerfile RUN java.sh和mvn.sh时，识别不了MVN_HOME同时也不支持$JAVA_HOME和$MVN_HOME等这样的 变量写法。修改dockerfile将RUN /etc/profile.d/java.sh 和mvn.sh改为：ENV JAVA_HOME=/usr/local/java MAVEN_HOME=/usr/local/mvn PATH=$PATH:/usr/local/mvn/bin:/usr/lcoal/java/bin

```
pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        metadata:
          labels:
            some-label: some-label-value
        spec:
          containers:
          - name: busybox
            image: busybox
            command:
            - cat
            tty: true
          - name: git
            image: docker:25-git
            securityContext:
              privileged: true
          - name: mvn
            image: mvn:v3
            command:
            - cat
            tty: true
        '''
      retries 2
    }
  }
  stages {
    stage('拉取代码') {
      steps {
        container('git') {
          checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'http://gogs.y2312.com/gogsadmin/book.git']])
        }
      }
    }
    stage('编译构建') {
      steps {
        container('mvn') {
            sh 'mvn clean package'
            sh 'sleep 60'
        }  
      }
    }
  }
}
```



```
错误二：写的代码推送仓库时报错send too large chunken boby为代码过大推送不了到代码仓库，可以在ingress-nginx中增加一个注解
vi gogs-dep-svc-ingress.yaml中
annotations:
    nginx.ingress.kubernetes.io/proxy-boby-size: 8m

错误三：在docker in docker运行时运行不起来，没有权限，可以在pipeline中改为在特权模式： 
	securityContext:
              privileged: true
```

>
>
>点击链接查看和 Kimi 的对话 https://www.kimi.com/share/19a96692-55e2-803d-8000-0000282a148c