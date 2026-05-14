## helm：部署模板工具

#### 1、前提：

```
普通文件部署：deployment-----service-----ingress----rbac（认证）=====针对本机使用

helm===repo（仓库）----chart包-----安装过后就是release
```

#### 2、安装helm：

```
helm官网搜索
tar -xf helm....
[root@www ~]# mv linux-amd64/helm /usr/bin/  ==移动helm到/usr/bin/

添加仓库：stable
helm repo add stable（自己取得名字） http://mirror.azure.cn/kubernetes/charts/
helm repo add brigade https://brigadecore.github.io/charts
1、查看有几个仓库：helm repo list
2、查看仓库里面的chart包：
[root@www ~]# helm search repo stable
3、安装仓库里面的stable（chart包）的release，并取名为y2312mysql
安装仓库stable里面的msyql并将模板写入y2312mysql中
[root@www ~]# helm install y2312mysql stable/mysql
4、查看release： helm list
5、卸载：[root@www ~]# helm uninstall y2312mysql
chart拉取下来：helm pull stable/mysql
tar xf msyql-..gz
cd msyql
里面有：Chart.yaml  README.md  templates  values.yaml四个文件
[root@www mysql]# helm install y2312mysql ./ --namespace prod

安装更新时，--set优于values.yaml文件修改的
[root@www mysql]# helm upgrade y2312mysql ./ --set imageTag=latest --image=mysql --namespace prod

```

helm常用命令：

![Snipaste_2025-11-12_16-21-38](pict\Snipaste_2025-11-12_16-21-38.png)

#### 3、helm内容：

>Chart.yaml ：描述chart的版本和信息的
>
>templates  ：自己部署放置地
>
>values.yaml：默认值

```
创建自己的chart：
[root@www ~]# helm create mychart
[root@www ~]# cd mychart/
[root@www mychart]# ls
charts  Chart.yaml  templates  values.yaml
删除一些template所有，values.yaml（:1,$d）里面内容,删除charts，并在Charts.yaml修改v3和版本

写模板：
vi templates/configmap.yaml
[root@www mychart]# helm install test-myconfig(自己取的名字) ./ --namespace y2312helm --create-namespace
[root@www mychart]# kubectl get cm -n y2312helm
NAME                DATA   AGE
kube-root-ca.crt    1      2m56s
mychart-configmap   1      2m56s
[root@www mychart]# kubectl get cm/mychart-configmap -n y2312helm -o yaml

写values.yaml
vi values.yaml
y2312data: "weixinyue"
favoriteDrink: coffee

引用：
vi templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}
data:
  myvalue: {{  .Values.y2312data  }}
  drink: {{ .Values.favoriteDrink }}
  
helm upgrade [RELEASE] [CHART] [flags]  
[root@www mychart]# helm upgrade test-myconfig（指定release） ./（指定chart在哪里） -n y2312helm

调试：
干跑：helm install --generate-name --dry-run --debug ./
     helm install y2312config --dry-run --debug ./
                  可以自己改名
```

#### 4、helm模板函数列表：

```
函数：helm官网===模板函数列表
1、quote：引号
vi templates/config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name}}
data:
  myvalue: {{  .Values.y2312data  }}
  drink: {{ quote .Values.favorite.drink }}====加上括号
  food: {{ .Values.favorite.food | quote }} ===|管道
helm install y2312config --dry-run --debug ./干跑

2、repeat：重复次数

3、default：默认值，如果values.yaml文件中没有就是用默认值
vi templates/config.yaml
game: {{ .Values.game | default "football" | quote }}===默认为football

vi values.yaml
game： ""===是空
干跑：helm install y2312config --dry-run --debug ./

4、index：获取列表当中的第几个函数
vi templates/config.yaml
 drink: {{ index .Values.favorite.drink 0 | quote }}

vi values.yaml
drink:
  - coffee
  - tea
```

#### 5、helm流控制：

```
流控制：
1、（if等循环）
vi templates/config.yaml
{{- if  index .Values.favorite.drink 0 | eq "coffee" }}
mug: "true"
{{- end }}====》-删除空白行
{{- }}:删除左边空白行
{{ -}}:删除右边空白行


2、range：循环
vi values.yaml
pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions

vi templates/config.yaml
toppings: |-
    {{- range .Values.pizzaToppings }}
    - {{ . | title（首字母大写） | quote }}
    {{- end }}
yaml文件的格式： 
|-：删除最后一行的换行符
| ：保留文中所有的换行符
> ：文本块中的换行替换为空格


3、with：改变作用域
vi templates/config.yaml
{{- with .Values.favorite }}====改变根了，下面可以不用写
drink: {{ index .drink 1 | quote }}
food: {{ .food | repeat 3 | quote }}
{{- end }}

4、变量：
name: {{ $relname := .Release.Name -}}=====》变量要写在外面
{{- with .Values.favorite }}
drink: {{ index .drink 1 | quote }}
food: {{ .food | repeat 3 | quote }}
name: {{ $relname }}
{{- end }}


遍历：即遍历k又遍历v
vi values.yaml
labels:
  app: mychart
  dev: test

vi templates/configmap.yaml
labels:
  {{- range $key,$val := .Values.labels }}
  {{ $key }} : {{ $val -}}
  {{  end }}
helm install y2312config --dry-run --debug ./
```

#### 6、helm部署应用myapp实战：

##### 6.1、部署myapp

```
部署myapp：

helm create myapp
[root@www myapp]# rm -rf templates/hpa.yaml templates/NOTES.txt templates/_helpers.tpl templates/serviceaccount.yaml 
[root@www templates]# ls
deployment.yaml  ingress.yaml  service.yaml===部署只需要这三个
```

##### 6.2、部署myapp之deployment.yaml

```
vi deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
  namespace: {{ .Release.Namespace }}
  labels:
    {{- range $key,$val := .Values.labels}}
    {{ $key }} : {{ $val }}
    {{- end }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- range $key,$val := .Values.labels}}
      {{ $key }} : {{ $val }}
      {{- end }}
  template:
    metadata:
      labels:
        {{- range $key,$val := .Values.labels}}
        {{ $key }} : {{ $val }}
        {{- end }}
    spec:
      containers:
      - name: {{ .Release.Name }}
        image: {{ .Values.image }}:{{ .Values.tag | default "latest" }}
        imagePullPolicy: {{ .Values.imagepullPolicy }}
        ports:
        - name: http
          containerPort: {{ .Values.service.port }}
          protocol: TCP

vi values.yaml
replicaCount: 1
image: nginx
imagepullPolicy: IfNotPresent
tag: ""
labels:
  app: book
  env: dev
service:
  type: ClusterIP
  port: 80
ingress:
  enabled: false
  className: ""
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local


[root@www myapp]# mv templates/service.yaml templates/service.yaml.bak 
[root@www myapp]# mv templates/ingress.yaml templates/ingress.yaml.bak 
[root@www myapp]# helm install --generate-name --dry-run --debug ./===窜染出deployment
```

##### 6.3、部署myapp之service.yaml

```
渲染service
vi templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}
  namespace: {{ .Release.Namespace }}
  labels:
    {{- range $key,$val := .Values.labels }}
    {{ $key }} : {{ $val -}}
    {{ end }}
spec:
  type: {{ .Values.service.type }}
  ports:
  - port: {{ .Values.service.port }}
    targetPort: http
    protocol: TCP
    name: http
  selector:
    {{- range $key,$val := .Values.labels }}
    {{ $key }} : {{ $val -}}
    {{ end }}
[root@www myapp]# helm install --generate-name --dry-run --debug ./===渲染出service
```

##### 6.4、部署myapp之ingress.yaml

```
渲染ingress===可以暴露也可以不暴露，可以写个if如果为false不暴露true则暴露
vi templates/ingress.yaml
{{- if .Values.ingress.enabled -}}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}
  namespace: {{ .Release.Namespace }}
  labels:
    {{- range $key,$val := .Values.labels}}
    {{ $key }}: {{ $val }}
    {{- end }}
spec:
  ingressClassName: {{ .Values.ingress.className }}
  rules:
    {{- $svcname := .Release.Name }}
    {{- $svcport := .Values.service.port }}
    {{- range .Values.ingress.hosts }}
    - host: {{ .host | quote }}
      http:
        paths:
          {{- range .paths }}
          - path: {{ .path }}
            pathType: {{ .pathType }}
            backend:
              service:
                name: {{ $svcname }}
                port:
                  number: {{ $svcport }}
          {{- end }}
    {{- end }}
{{- end }}

[root@www myapp]# helm install --generate-name --set ingress.enabled=true --dry-run --debug ./=====ingress暴露

# 此时没有完全安装
# 完全安装： helm install myapp --set ingress.enabled=true -n y2312helm ./
[root@master myapp]# helm list
NAME     NAMESPACE   REVISION    UPDATED      STATUS  CHART  APP VERSION
chart-1765679265 default  1 2025-12-14 10:27:45.859841187 +0800 CST deployed        myapp-0.0.1  0.0.1
```

##### 6.5、helm其他命令

```
[root@www myapp]# helm list -n y2312helm===查看名称空间
NAME 	NAMESPACE	REVISION	UPDATED       	STATUS	CHART      	APP VERSION
myapp	y2312helm	1  2024-03-23 05:34:29.333898298 +0800 CST	deployed	myapp-0.1.0	1.16.0 
 helm uninstall RELEASE_NAME [...] [flags]
[root@www myapp]# helm uninstall myapp -n y2312helm ./ ====》卸载
helm install myapp --set ingress.enabled=true --namespace y2312helm ./===>运行

[root@www myapp]# helm history myapp -n y2312helm===查看有几个版本
[root@www myapp]# helm rollback myapp -n y2312helm===回滚
```

>注意安装myapphelm包时（真正安装时）：不要添加参数--dry-run