## k8s 存储

​	emptydir： pod内部容器数据共享

​	hostpath：使用本地宿主机的存储挂载到pod内部

### 1、定义存储：

```
 volumes:
 - name：
   emptyDir
   hostPath
pv： 定义存储

storagclass：存储类

PVC：存储申请

kubectl explain pv | less中得到
https://kubernetes.io/docs/concepts/storage/persistent-volumes#access-modes网站
accessModes：存储模式
	ReadWriteOnce
	ReadOnlyMany
	ReadWriteMany
	ReadWriteOncePod
capacity：提供存储的大小
```

### 2、实例：

```
======
100：
[root@www y2312mainfeast]# mkdir pv
[root@www y2312mainfeast]# cd pv/
vi pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
  labels:
    app: pv1
spec:
  capacity: 存储大小
    storage: 10Gi
  accessModes:===支持访问的类型
    - ReadWriteOnce
  hostPath:
    type: Directory
    path: /data/pv1
[root@www pv]# kubectl apply -f pv.yaml 
persistentvolume/pv1 created
[root@www pv]# kubectl get pv

vi pv2.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv2
  labels:
    app: pv2
spec:
  capacity:
    storage: 5Gi===存储5个G
  accessModes:
    - ReadWriteOnce
  hostPath:
    type: Directory
    path: /data/pv2

要使用这两个pv就要创建pvc，pvc有名称空间，pv没有，pv是集群资源
accessModes=要指定
resources=资源的大小
vi pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
  namespace: prod
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
kubectl apply -f pvc.yaml
kubectl get pvc -n prod
此时pvc1和pv1处于绑定状态
pvc1   Bound    pv1      10Gi       RWO             

使用pod与pvc1关联：
vi pod-pvc.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-pvc1
  namespace: prod
spec:
  volumes:
  - name: webdir
    persistentVolumeClaim:==存储媒介
      claimName: pvc1===资源类型
  containers:
  - name: web
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: webdir
      mountPath: "/usr/share/nginx/html"====要加引号
k9s中prod==pod-pvc1中ip为：10.244.2.197节点在node1上面
在101：echo "this is pv1" > /data/pv1/index.html
在100：curl 10.244.2.197===》得到this is pv1====》存储成功





101和102：
mkdir -pv /data/pv1
mkdir -pv /data/pv2
```

### 3、storage:将pv和pvc进行分组（分类）

```
一个存储类：
vi sc1.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: golden
  namespace: dev
provisioner: kubernetes.io/no-provisioner====提供者

第二个存储类：
vi sc2.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: sliver
  namespace: dev
provisioner: kubernetes.io/no-provisioner

[root@www pv]# kubectl get sc
NAME     PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
golden   kubernetes.io/no-provisioner   Delete          Immediate           false                  25s
sliver   kubernetes.io/no-provisioner   Delete          Immediate           false                  20s

Immediate：绑定策略，只要有pvc来就立即绑定，将pv话定在那个资源组里面，有两个资源组：golden和sliver
vi pv3|4.yaml===创建pv
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv3|4
  labels:
    app: pv3|4
spec:
  storageClassName: golden| sliver
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    type: Directory
    path: /data/pv3|4

vi pvc2.yaml====创建pvc并在golden组
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc2
  namespace: prod
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi  | 5Gi===此时就可以创建成功
  storageClassName: golden

此时pvc2没有绑定，golden里面是5个G，pvc里面是8个G
[root@www pv]# kubectl get pvc -n prod
NAME   STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc1   Bound     pv1      10Gi       RWO                           65m
pvc2   Pending                                      golden         12s

vi pod-pvc2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-pvc2
  namespace: prod
spec:
  nodeSelector:===节点选择node2
    noderole: web==标签===》kubectl get node --show-labels==得知node2标签为web
 
  containers:
    - name: webdir
      image: nginx:latest
      imagePullPolicy: IfNotPresent
      volumeMounts:
      - mountPath: "/usr/share/nginx/html"
        name: webdir
  volumes:
    - name: webdir
      persistentVolumeClaim:
        claimName: pvc2

101： mkdir -pv /data/pv3和4
```

> pod---pvc---bound--pv--hostpath：pv通过hostpath绑定pvc调度pod，但pod可能呢个会在其他节点上，会导致绑定不了，程序运行不了

### 4、指定pv选择节点

​	在那个节点pod就在那个节点上===》localpv

```
nodeAffinity：节点亲和性，使节点运行在那个节点上
	Reqired：硬限制，必须
创建pv----创建storageclass---创建pvc---pod调用pvc
1、创建pv
vi local-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-local
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:====访问模式
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete===回收模式
  storageClassName: local-storage====属于哪个存储类，要自己创建
  local:
    path: /data/k8s/localpv # node2节点上的目录
  nodeAffinity:==节点亲和性
    required:====硬核性
      nodeSelectorTerms:===节点选择器
      - matchExpressions:
        - key: kubernetes.io/hostname==标签
          operator: In
          values:
          - node2.y2312.com===标签值

2、创建storageclass存储类
创建local-storage===kubectl get pv
vi local-storage.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
  namespace: dev
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer====挂载绑定模式

3、创建pvc
vi local-pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-local====名称要一致
  namespace: dev
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: local-storage
[root@www pv]# kubectl get pvc -n dev===处于Pending

4、pod调用pvc：
vi pod-pvc.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-pvc3
  namespace: dev
spec:
  nodeSelector:
    noderole: web
  containers:
    - name: web
      image: nginx:latest
      imagePullPolicy: IfNotPresent
      volumeMounts:
      - name: webdir
        mountPath: /usr/share/nginx/html
  volumes:
    - name: webdir
      persistentVolumeClaim:
        claimName: pvc-local====名称要一致

k9s中进入dev===pod-pvc3==shell==echo "this is pvc3" > /usr/share/nginx/html
102：
创建pv：[root@www ~]# mkdir -pv /data/k8s/localpv
cat /data/k8s/localpv/index.html
====>this is pvc3
```

### 5、外部存储引入集群内部

nfs：小型

分布式存储：ceph

```
需要100和11服务器：
11：关闭防火墙
[root@www ~]# yum install rpcbind nfs-utils -y
[root@www ~]# systemctl start nfs
[root@www ~]# systemctl status nfs
vi /etc/exports
/data/k8s/ 192.168.18.0/24(rw,no_root_squash)
重启nfs：systemctl restart nfs

101：需要先安装nfs
查看是否挂载
[root@www ~]# mount -t nfs 192.168.18.11:/data/k8s /data
df -h 查看是否挂载
[root@www ~]# umount 192.168.18.11:/data/k8s
浏览器：https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner==查看
==deploy==deployment.yaml

100：
1、安装驱动：
vi nfs-provisioner.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  labels:
    app: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner===pod以哪个身份来运行
      containers:
        - name: nfs-client-provisioner
          image: registry.cn-chengdu.aliyuncs.com/mr5/nfs-provisioner:v4.0.2
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME===驱动名称
              value: k8s-sigs.io/nfs-subdir-external-provisioner
            - name: NFS_SERVER
              value: 192.168.18.11 ====地址
            - name: NFS_PATH
              value: /data/k8s
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.18.11
            path: /data/k8s
[root@www pv]# kubectl apply -f nfs-provisor.yaml 
k9s中default中有nfs-client-provisioner


2、授权：
 vi nfs-rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
kubectl apply -f nfs-rbac.yaml

3、使用驱动引入外部存储：
vi nfs-class.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-client
provisioner: k8s-sigs.io/nfs-subdir-external-provisioner # or choose another name, must match deployment's env PROVISIONER_NAME'
parameters:
  archiveOnDelete: "false"====回收策略，false删除的时候不归档，true时删除就归档 
  或者：
  onDelete： "retain"===删除并保留原来目录
查看策略：
[root@www pv]# kubectl get storageclass===出现nfs-client

4、创建pvc
vi nfs-pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-claim
spec:
  storageClassName: nfs-client==存储类
  accessModes:
    - ReadWriteMany===存储模式
  resources:
    requests:
      storage: 1Mi
[root@www pv]# kubectl apply -f nfs-pvc.yaml 
kubectl get pvc
test-claim处于绑定状态

4、创建pod
vi nfs-testpod.yaml
kind: Pod
apiVersion: v1
metadata:
  name: test-pod
spec:
  containers:
  - name: test-pod
    image: busybox:stable
    command:
      - "/bin/sh"
    args:
      - "-c"
      - "touch /mnt/SUCCESS && exit 0 || exit 1"===在/mnt下面写上/SUCCESS并退出
    volumeMounts:
      - name: nfs-pvc===挂载到/mnt
        mountPath: "/mnt"
  restartPolicy: "Never"
  volumes:
    - name: nfs-pvc
      persistentVolumeClaim:
        claimName: test-claim==使用的pvc
kubectl apply -f nfs-testpod.yaml
k9s中default中test-pod的删除11服务器中依然可以看到数据

local-pv： 先通过节点亲和性选择pv在那个节点上，当创建pod去绑定pv的时候，pv在那个节点上面pod就在那个节点上面
storageclass:引入外部存储，创建pvc就行了，pvc与pv进行绑定
```

>创建sc时回收策略：
>
>parameters:
>  archiveOnDelete: "false"====回收策略，false删除的时候不归档，true时删除就归档 
>  或者：
>  onDelete： "retain"===删除并保留原来目录

**nfs动态供给：已经创建nfs，需要动态供给，以后只需要填写pvc和pod申请即可（deployment状况下）**