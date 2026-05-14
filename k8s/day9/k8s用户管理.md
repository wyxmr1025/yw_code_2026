## k8s用户管理：

​	api-server是集群入口，默认监听在https：//6443端口

​	认证：api-server===配置文件/etc/kubernetes/admin.conf复制到~/.kube/ 改名为config文件

```
[root@www ~]# mv ~/.kube/config  ~/.kube/config.back
[root@www ~]# kubectl get node====此时就没有查到node
 ~/.kube/config 保存的是k8s一些认证的东西
```

#### 1、用户管理基础知识：

```
vi ~/.kube/config
certificate-authority-data：集群证书
server: https://192.168.18.100:6443：集群的ip地址
users：用户
kubernetes-admin：用户名称
client-certificate-data：用户证书
context:上下文，将用户与集群进行绑定
current-context：当前使用的是哪个上下文

```



```
k8s基于双向tls证书认证
		用户基于证书认证
		1、集群外部用户
			useraccunt
			例：kubectl k9s
				配置文件放在：~/.kube/config
		2、集群内部用户
			serveraccount：用于集群内部与pod进行通信
```

#### 2、创建集群：

```
get-clusters      显示在 kubeconfig 中定义的集群，
列出当前有哪些集群：
[root@www ~]# kubectl config get-clusters
NAME
kubernetes

get-users ：当前的用户
[root@www ~]# kubectl config get-users
NAME
kubernetes-admin

get-contexts：当前的会话
[root@www ~]# kubectl config get-contexts
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   

set-cluster：创建集群
set-context： 创建上下文，将用户与集群进行绑定
set-credentials：创建用户

1、创建集群：
 NAME：名字
 [--server=server] ：服务器地址
 [--certificate-authority=path/to/certificate/authority] ：证书
 [--insecure-skip-tls-verify=true]要不要校验证书
 [--tls-server-name=example.com]：指定服务器的名称
  --embed-certs=false：将证书里面的内容隐藏
 /etc/kubernetes/pki/：各类证书的存放位置
创建集群命令： kubectl config set-cluster y2312 --server=https://192.168.18.100:6443 --certificate-authority=/etc/kubernetes/pki/ca.crt --embed-certs
 查看是否添加成功： vi ~/.kube/config===加密证书是一样的
 
```

#### 3、创建用户：

```
首先生成私钥----通过秘钥----生成证书申请
openssl genrsa -out mr5.key 2048
openssl req -new -key mr5.key -out mr5.csr -subj "/CN=mr5/O=y2312"
openssl x509 -req -in mr5.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out mr5.crt -days 500
创建用户（写在配置文件）：
NAME：名称
[--client-certificate=path/to/certfile]：证书的地址
[--client-key=path/to/keyfile] ：证书的钥匙
--client-key=：证书的私钥
 --embed-certs：将证书隐藏
[root@www ~]# kubectl config set-credentials luoxiao --client-certificate=./mr5.crt --client-key=./mr5.key --embed-certs
获取用户：
[root@www ~]# kubectl config get-users
NAME
kubernetes-admin
luoxiao


```

#### 4、配置上下文：将集群与用户绑定在一起

```
[NAME | --current] [--cluster=cluster_nickname] [--user=user_nickname]
[--namespace=namespace] [options]
例：
[root@www ~]# kubectl config set-context luoxiao@y2312 --cluster=y2312 --user=luoxiao                              （自己取的名字）
查询当前会话：
[root@www ~]# kubectl config get-contexts
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   
          luoxiao@y232                  y2312        luoxiao   

使用那个上下文：use-context
kubectl config use-context luoxiao@y2312
此时luoxiao@y2312没有任何权限的

```

#### 5、授权：rbac==role base access controll

```
查询角色：
[root@www ~]# kubectl get role
NAME                                    CREATED AT
leader-locking-nfs-client-provisioner   2024-03-19T15:45:08Z
role:角色-----名称空间授权role
clusterrole:集群角色（用户空间资源）
apiGroups：资源组
resources：资源
verbs：对这些资源可以进行那些操作（增删改查list列出）
['get', 'list', 'watch', 'create', 'update', 'patch', 'delete']
                                             打补丁
apiGroups: ['', 'apps']-----''代表核心资源组
查看有哪些资源组：kubectl api-versions

1、创建角色：
[root@www ~]# mkdir y2312mainfeast/rbac
[root@www ~]# cd y2312mainfeast/rbac/
vi role1.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: test-role
  namespace: prod====只在prod里面生效
rules:
  - apiGroups: ['',]核心资源组
    resources: ['pods']==针对pod，有以下权限
    verbs: ['get', 'list', 'watch', 'create', 'update', 'patch', 'delete'] 

2、将角色分配给用户：rolebanding
vi rolebinding1.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cnych-rolebinding
  namespace: prod
subjects:===绑定那个用户
  - kind: User
    name: mr5   ====创建的luoxiao
    apiGroup: ''
roleRef:===关联到个角色里面
  kind: Role
  name: test-role
  apiGroup: rbac.authorization.k8s.io
  切换用户到mr5：
  kubectl config use-context luoxiao@y2312
  [root@www rbac]# kubectl get pod -n prod===只在prod当中生效
- node是集群资源，要进行授权才能使用，授权一个clusterole

3、授权ClusterRole，node是集群资源，要进行授权才能使用，授权一个clusterole
vi clusterrole1.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: test-clusterrole
rules:
  - apiGroups: ['',]
    resources: ['nodes']
    verbs: ['get', 'list', 'watch', 'create', 'update', 'patch', 'delete']

clusterroledinding：绑定
vi clusterrolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cnych-clusterrolebinding
  namespace: prod
subjects:
  - kind: User
    name: mr5
    apiGroup: ''
roleRef:
  kind: ClusterRole
  name: test-clusterrole
  apiGroup: rbac.authorization.k8s.io
切换账号：[root@www rbac]# kubectl config use-context luoxiao@y232
Switched to context "luoxiao@y232".
查看node：[root@www rbac]# kubectl get node
查看ns：不行，没有授权
查看role所有的名称空间：[root@www rbac]# kubectl get role -A
查看clusterrole所有的名称空间：[root@www rbac]# kubectl get clusterrole -A
cluster-admin：集群管理员

将cluster-admin绑定在mr5上面，那么mr5就成为管理员：
[root@www rbac]# kubectl create clusterrolebinding mr5-admin --clusterrole=cluster-admin --user=mr5

总结：
rolebinding：可以绑定role和clusterrole
clusterrolebinding：可以绑定clusterrole

```

>查看有哪些资源组：kubectl api-versions
>
>kubectl api-resources同样也可以查看

#### 6、创建serviceaccount用户

```
1、创建serviceaccount用户：
[root@www rbac]# kubectl create serviceaccount joyi
2、GitHub.com上面找插件：dashboard
地址：https://github.com/kubernetes/dashboard/releases?page=7
修改配置文件：Service：===type: NodePort
3、k9s-ns-svc:(找到端口)---浏览器中输入
4、需要输入token：在k9s中选个podshell进入：
t8jp:/# ls /var/run/secrets/kubernetes.io/serviceaccount/
创建token：  kubectl create token joyi
5、使joyi成为管理员：
kubectl create clusterrolebinding joyiadmin --clusterrole=cluster-admin --serviceaccount=dafault:joyi（ns_name:sa_name）
```



