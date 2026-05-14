## prometheus-operator

#### 1、prometheus-operator定义

​	prometheus-operator：自动化，属于有状态的指标（不能通过deployment来部署，而是使用statfulset来部署，使用一些逻辑来编写更符合规范）

```
prometheus-operator
	crd-----定义其他资源的资源
在crd中部署prometheus，要运行起来就要有控制器（controller），并把资源运行起来
在官网上prometheus-operator中快速安装:github上搜索

[root@www ~]# git clone https://github.com/prometheus-operator/kube-prometheus.git
1、创建crd和ns
[root@www ]# cd kube-prometheus
[root@www kube-prometheus]# kubectl create -f manifests/setup
2、#查看crd资源：
[root@www kube-prometheus]# kubectl get crd
3、创建controller
[root@www kube-prometheus]# kubectl create -f manifests/
[root@www kube-prometheus]# kubectl get prometheus -A
NAMESPACE    NAME   VERSION   DESIRED   READY   RECONCILED   AVAILABLE   AGE
monitoring   k8s    2.50.1    2                                          37s
====>：
#修改prometheus、alertmanager等副本为1，svc的type为NodePort类型。
```

#### 2、prometheus-operator原理：

```
通过crd创建了一个prometheus资源并交给apiserver----operator或者controller时刻监视（whatch）prometheus资源----》根据逻辑创建statefulset（k8s内置的资源）和service---又交给apiserver的controller---创建pod、pvc等----创建servermonitor被operatoe或者controller看到---就创建了一些配置规则（servermonitor）
												job_name：
												kubernetes_sd_configs
												- role: endpoints|node|service...
												relabels:
```



![image-20250312150856333](pict\image-20250312150856333.png)

#### 3、prometheus-operator实战之创建api-server的servicemonitor配置规则

```
servicemonitor关注service下的pod，并将它添加到prometheus里面，被operatorwatch到添加到prometheus server的配置规则。
cd kube-prometheus/manifests/
mkdir servicemonitor
cp *Monitor* servicemonitor/
cd servicemonitor/
kubectl delete -f ./====先删除，在自己写
创建servermonitor：变成配置规则
#首先先查看集群内部自带的api-server：kubectl get pod -n kube-system(通过endpoints关联)
cp kubernetesControlPlane-serviceMonitorApiserver.yaml apiserver-servicemonitor.yaml

vi apiserver-servermonitor.yaml（api-server、kuberlet、contoller、tsdb、node-exporter、coredns、scheduler、controller-manager等规则,先删除relabels等标签。）
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    app.kubernetes.io/name: apiserver
    app.kubernetes.io/part-of: kube-prometheus
  name: kube-apiserver
  namespace: monitoring
spec:
  endpoints:
  - bearerTokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
    interval: 15s    
    port: https
    scheme: https
    tlsConfig:
      caFile: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      serverName: kubernetes
  jobLabel: component
  namespaceSelector: =====》以下是在那个名称空间下找service
    matchNames:
    - default
  selector:
    matchLabels:
      component: apiserver
      provider: kubernetes
#运行：kubectl apply -f servicemonitor-apiserver.yaml
#查看servicemonitor的字段：
kubectl explain servicemonitor.spec.endpoints | less

#查看内置的api-server的标签等
kubectl get svc/kubernetes -o yaml

```

![Snipaste_2026-05-11_16-07-02](pict\Snipaste_2026-05-11_16-07-02.png)



#### 4、prometheus-operator实战之创建kubelet的servicemonitor配置规则

```
创建kuberlet的servicemonitor
[root@www servermonitor]# cp kubernetesControlPlane-serviceMonitorKubelet.yaml kubeletsservicemonitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    app.kubernetes.io/name: kubelet
    app.kubernetes.io/part-of: kube-prometheus
  name: kubelet
  namespace: monitoring
spec:
  endpoints:
  #基于https的
  - bearerTokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
    honorLabels: true
    interval: 15s
    port: https-metrics
    scheme: https
    tlsConfig:
      insecureSkipVerify: true
  #基于cAdvisor的
  - bearerTokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
    honorLabels: true
    honorTimestamps: false
    interval: 15s
    path: /metrics/cadvisor
    port: https-metrics
    scheme: https
    tlsConfig:
      insecureSkipVerify: true
  #基于http的
  - bearerTokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
    honorLabels: true
    interval: 15s
	path: /metrics/probes
    port: https-metrics
    scheme: https
    tlsConfig:
      insecureSkipVerify: true
  jobLabel: app.kubernetes.io/name
  namespaceSelector:  ==指定那个名称空间找那个svc
    matchNames:
    - kube-system
  selector:
    matchLabels:
      app.kubernetes.io/name: kubelet


#内置的kubelet的svc：kubectl get svc/kubelet -n kube-system -o yaml
```

![Snipaste_2026-05-11_16-14-53](pict\Snipaste_2026-05-11_16-14-53.png)

```
node-export的servicemonitor
coredns的servicemonitor
```

> 总结：配置coredns、scheduler、controller-manager等同上（配置前先查看原生k8s中的各个组件如何配置，再查看）