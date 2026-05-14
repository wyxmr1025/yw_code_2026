## CICD：k8s部署dockerharbor

#### 1、部署镜像仓库：harbor

```
dockerharbor：端口443
docker镜像仓库：
docker harbor官网下载helm模板 官网：goharbor.io
harbor压缩包解压到cicd
yum install unzip -y
mkdir harbor
unzip harbor.zip -d harbor
cd harbor
开启外部存储的服务器nfs服务器

vi values-prod.yaml
externalURL: https://harbor.y2312.com
harborAdminPassword: Harbor12345
logLevel: debug

expose:===暴露
  type: ingress
  tls:
    enabled: true
  ingress:
    className: nginx # 指定 ingress class
    hosts:
      core: harbor.y2312.com
      notary: notary.y2312.com

persistence:
  enabled: true
  resourcePolicy: 'keep'
  persistentVolumeClaim:
    registry:
      # 如果需要做高可用，多个副本的组件则需要使用支持 ReadWriteMany 的后端
      # 这里我们使用nfs，生产环境不建议使用nfs
      storageClass: 'nfs-client'
      # 如果是高可用的，多个副本组件需要使用 ReadWriteMany，默认为 ReadWriteOnce
      accessMode: ReadWriteMany
      size: 5Gi
    chartmuseum:
      storageClass: 'nfs-client'
      accessMode: ReadWriteMany
      size: 5Gi
    jobservice:
      jobLog:
        storageClass: 'nfs-client'
        accessMode: ReadWriteMany
      size: 1Gi
    trivy:
      storageClass: 'nfs-client'
      accessMode: ReadWriteMany
      size: 2Gi
    redis:
      storageClass: 'nfs-client'
      accessMode: ReadWriteMany
      size: 2Gi
    database:
      storageClass: 'nfs-client'
      accessMode: ReadWriteMany
      size: 8Gi

database:
  type: internal
  internal:
    image:
      repository: goharbor/harbor-db
      tag: v2.9.0
      password: "changeit"
      shmSizeLimit: 512Mi
redis:
  type: internal
  internal:
    image:
      repository: goharbor/redis-photon
      tag: v2.9.0


# 默认为一个副本，如果要做高可用，只需要设置为 replicas >= 2 即可
portal:
  replicas: 1
core:
  replicas: 1
jobservice:
  replicas: 1
registry:
  replicas: 1
chartmuseum:
  replicas: 1
trivy:
  replicas: 1
notary:
  server:
    replicas: 1
  signer:
    replicas: 1
安装： helm upgrade --install harbor -f values-prod.yaml -n kube-ops ./
win7解析主机名：192.168.18.101 harbor.y2312.com
浏览器中输入：https://harbor.y2312.com
添加证书CA：
登录harbor仓库： admin Harbor12345 
 
```

#### 2、信任证书：

```
信任证书：kubectl -n kube-ops get secrets harbor-ingress -o jsonpath="{.data.ca\.crt}" | base64 -d > ca.crt
cat ca.crt
sz ca.crt
101:
docker tag mvn:v5 harbor.y2312.com/cicd/mvn:1.0.0===打上tag
解析主机名：vi /etc/hosts
192.168.18.101 harbor.y2312.com
推送镜像： docker push harbor.y2312.com/cicd/mvn:1.0.0===此时推不上，要信任
将100的ca.crt推到101中：scp ca.crt 192.168.18.101:/root/
100、101、102中：信任证书：mkdir /etc/docker/certs.d/harbor.y2312.com -pv
mv ca.crt /etc/docker/certs.d/harbor.y2312.com/
登陆仓库推镜像
docker login harbor.y2312.com ==admin harbor12345
docker push harbor.y2312.com/cicd/mvn:v1.0.0
此时宿主机上面可以推镜像了，
将证书复制到102服务器上
 scp -r /etc/docker/certs.d/ 192.168.18.102:/etc/docker/
 目前宿主机上面可以推镜像，要求运行的pod要解析到harbor.y2312.com这个域名及要求添加一条解析记录：

```

##### 2.1推送镜像到harbor遇到的问题：

**遇到那个的困难：** 创建好镜像仓库harbor后，将自己创建好的mvn镜像推送上去时，出现tls:failed to verify certifcate: x509: certifcate signed by unknow authority,原因：docker上传下载默认只支持https协议，搭建的私有镜像仓库是http协议。解决方案如下两种，选择其一。

```
方法1：
vi /etc/docker/daemon.json
{
...
"insecure-registers": ["192.168.66.91:5000"]
...
}
方法2：直接信任证书
信任证书：mkdir /etc/docker/certs.d/harbor.y2312.com -pv
mv ca.crt /etc/docker/certs.d/harbor.y2312.com/
```



#### 3、编译构建好的war包 打包成docker镜像

```
一、编译构建好的war包 打包成docker镜像：
 1 需要Dockerfile，写在代码仓库里面
 cd Book-management-system
 export JAVA_HOME=/usr/local/java8
 mvn clean package
 2 写dockerfile====mvn跑在tomcat里面的，此时需要tomcat镜像==dockerhub官网搜tomcat8
 vi Dockerfile
 FROM tomcat:8.5.100-jdk
 ADD target/*.war /usr/local/tomcat/webapps/ROOT.war
 3 拉去镜像： docker build . -t book:v1
 4 测试：docker run -p 38080:8080 --rm book:v1
 5 浏览器中访问： 192.168.18.101:38080====图书馆的页面
 将tomcat上传到到自己的harbor.y2312.com的仓库中，先拉取在打包：
 docker pull tomcat:8.5.100-jdk8==拉去
 docker tag tomcat:8.5.100-jdk8 harbor.y2312.com/cicd/tomcat:8.5-jdk8==打包
 docker push harbor.y2312.com/cicd/tomcat:8.5-jdk8===上传
 rm -rf target/
 换成自己的镜像
 vi Dockerfile
 FROM harbor.y2312.com/cicd/tomcat:8.5-jdk8
 ADD target/*.war /usr/local/tomcat/webapps/ROOT.war
 docker build . -t book:v1
 提交到自己代码仓库去：
 git add .
 git commit -m "add a dockerfile"
 git push -u origin master  ====>gogsadmin Aa@123456
 历览器中输入：gogs.y2312.com
 
 
```

#### 4、打包有dind和doutd

```
二、打包有dind和doutd
 要实现docker客户端和docker服务端通信
 基于/var/run/docker.sock通信
 先备份：mv /var/run/docker.sock /var/run/docker.sock.bak
 将sock文件挂到容器里去，里面的容器就可以与外面的docker服务器进行通信：
 mv /var/run/docker.sock.bak /var/run/docker.sock
 dockerhub官网找寻docker
 docker pull docker:25.0.5-cli===docker-cli的客服端
 -v:挂载
  docker run -it --rm -v /var/run/docker.sock:/var/run/docker.sock --entrypoint /bin/sh docker:25.0.5-cli
 docker images此时可以在容器内查看镜像
 在容器内部登录在推镜像：
 docker login --username admin --password Harbor12345 harbor.y2312.com==登录
 打包镜像： docker tag harbor.y2312.com/cicd/mvn:v1.0.0 harbor.y2312.com/cicd/mvn:v1.0.1
 docker push harbor.y2312.com/cicd/mvn:v1.0.1===推送
 exit===退出bin模式
 
 docker tag docker:25.0.5-cli harbor.y2312.com/cicd/docker:25.0.5-cli
 docker push harbor.y2312.com/cicd/docker:25.0.5-cli
 coredns中解析域名： 192.168.18.101 harbor.y2312.com
 在jenkins当中写pipeline---》拉取代码---编译构建---打包镜像
 
 
 
docker images | grep book==查看镜像在哪个节点上
 
jenkins凭据管理（保存用户名和密码），username：admin，passwd：Harbor12345

流水线语法中选择：withCredentials:Bind credential to variables,绑定：username和password分开；用户名变量：huser，密码变量：hpass====》生成流水线语法脚本
打包镜像语法上：sh "echo ${hpass} | docker login harbor.y2312.com --username ${huser} --password-stdin"   =====>前提是要先登陆成功才能推送镜像
 
保证几个节点都能推送：
linux系统信任证书：cp /etc/docker/certs.d/harbor.y2312.com/ca.crt /etc/pki/ca-trust/source/anchors/
update-ca-trust===更新证书
复制到102服务器上：scp /etc/pki/ca-trust/source/anchors/ca.crt 192.168.66.102:/etc/pki/ca-trust/source/anchors/
 更新证书列表：update-ca-trust系统信任证书
```



jenkins中pipeline（docker out docker）

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
          volumes:
          - name: dsock
            hostPath:
             path: /var/run/
             type: Directory
          containers:
          - name: docker-cli
            image: harbor.y2312.com/cicd/docker:25.0.5-cli
            command:
            - cat
            tty: true
            volumeMounts:
            - mountPath: /var/run/
              name: dsock
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
            sh 'sleep 10'
        }  
      }
    }
    stage('打包镜像') {
      steps {
         container('docker-cli') {
            sh 'docker build . -t  harbor.y2312.com/bookmanage/book:v0.0.1'
            withCredentials([usernamePassword(credentialsId: 'harbor', passwordVariable: 'hpass', usernameVariable: 'huser')]) {
            sh "echo ${hpass} | docker login harbor.y2312.com --username ${huser} --password-stdin" 
            sh 'docker push  harbor.y2312.com/bookmanage/book:v0.0.1'
            }
         }
      }
    }
  }
}
```

