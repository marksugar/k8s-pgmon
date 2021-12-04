### Kubernetes 兼容性矩阵

当我们在各自的分支中针对这些版本进行测试时，支持并运行以下版本。但请注意，其他版本可能有效！

| kube-prometheus 堆栈                                         | Kubernetes 1.18 | Kubernetes 1.19 | Kubernetes 1.20 | Kubernetes 1.21 | Kubernetes 1.22 |
| ------------------------------------------------------------ | --------------- | --------------- | --------------- | --------------- | --------------- |
| [`release-0.6`](https://github.com/prometheus-operator/kube-prometheus/tree/release-0.6) | ✗               | ✔               | ✗               | ✗               | ✗               |
| [`release-0.7`](https://github.com/prometheus-operator/kube-prometheus/tree/release-0.7) | ✗               | ✔               | ✔               | ✗               | ✗               |
| [`release-0.8`](https://github.com/prometheus-operator/kube-prometheus/tree/release-0.8) | ✗               | ✗               | ✔               | ✔               | ✗               |
| [`release-0.9`](https://github.com/prometheus-operator/kube-prometheus/tree/release-0.9) | ✗               | ✗               | ✗               | ✔               | ✔               |
| [`main`](https://github.com/prometheus-operator/kube-prometheus/tree/main) | ✗               | ✗               | ✗               | ✔               | ✔               |

## 探讨

这只是一个笔记的整理，如果有错误之处请指正。如果你有兴趣一起来讨论，欢迎加入[dingtalk](https://www.dingtalk.com/)群号: 32040586

- [1.clone代码](## 1.clone)
- [2.修改副本](## 2.修改副本)
- [3.修改端口配置](## 3.修改端口配置)
- [4.部署](## 4.部署)
  - [4.1 集群监控添加](### 4.1 集群监控添加)
  - [4.2 etcd](### 4.2 etcd)
  - [4.3 kube-proxy](### 4.3 kube-proxy)
  - [4.4 ingress-nginx](### 4.4 ingress-nginx)
- [5.修改存储位置](## 5.修改存储位置)
  - [5.1 prometheus](### 5.1 prometheus)
  - [5.2 grafana](### 5.2 grafana)
- [6.修改时区](## 6.修改时区)
- [7.webhook](## 7.webhook)
- [8.监控node](## 8.监控node)
  - [8.1 安装docker](### 8.1 安装docker)
- [9.监控mysql](## 9.监控mysql)
  - [9.1 mysql 监控参数](### 9.1 mysql 监控参数)
  - [9.2 node_exporter监控其他](### 9.2 node_exporter监控其他)
- [10.监控 rabbitmq](## 10.监控 rabbitmq)
- [11.监控mongodb](## 11.监控mongodb)
- [12.监控kafka](## 12.监控kafka)
- [13.监控windows](## 13.监控windows)
- [14.监控nginx](## 14.监控nginx)
- [15.监控NFS](## 15.监控NFS)
- [16.jvm](## 16.jvm)
  - [16.1 redis](### 16.1 redis)
- [16.2 添加警报项](## 16.2 添加警报项)
- [16.dingtalk1](## 16.dingtalk1)
- [17.dingtalk2](## 17.dingtalk2)
  - [17.0 config](### 17.0 config)
    - [17.0.1 关闭](#### 17.0.1 关闭)
    - [17.1 集群外主机rule](##### 17.1 集群外主机rule)
      - [linux](##### linux)
      - [磁盘使用率](##### 磁盘使用率)
      - [nginx](##### nginx)
      - [redis](##### redis)
      - [rabbitmq](##### rabbitmq)
      - [mysql](##### mysql)
      - [kafka](##### kafka)
      - [mongodb](##### mongodb)
      - [obd](##### obd)
      - [windows](##### windows)
      - [etcd](##### etcd)
    - [17.2 告警分组](#### 17.2 告警分组)
      - [17.2.1 8060.yaml](##### 17.2.1 8060.yaml)
      - [17.2.2 8061.yaml](##### 17.2.2 8061.yaml)
- [18.修改原有的规则](## 18.修改原有的规则)
- [19.外部监控k8s](## 19.外部监控k8s)
  - [19.1 授权](### 19.1 授权)
  - [19.2 compose安装prom](### 19.2 compose安装prom)
  - [19.3 alertmanager](### 19.3 alertmanager)
    - [19.3.1 报警模板](#### 19.3.1 报警模板)
  - [19.4 alert](### 19.4 alert)
  - [19.5 配置钉钉alertmanager](### 19.5 配置钉钉alertmanager)
    - [19.5.1 grafana模板](#### 19.5.1 grafana模板)
- [20.ingress](## 20.ingress)
  - [20.1 打标签](### 20.1 打标签)
  - [20.2 alertmanager](### 20.2 alertmanager)	
  - [20.3 grafana](### 20.3 grafana)
  - [20.4 prometheus](### 20.4 prometheus)
  - [20.5 nginx代理](### 20.5 nginx代理)
- [21.模板](## 21.模板)
- [22.钉钉分组模板](## 22.钉钉分组)

时间同步： 

```
yum -y install ntp ntpdate
ntpdate 0.asia.pool.ntp.org
#系统时间同步到硬件，防止系统重启后时间呗还原
ntpdate ntp.aliyun.com
hwclock --systohc

2、检测的是时间同步服务，安装chrony并设置成启动即可
yum install -y chrony
systemctl restart chronyd ; systemctl enable chronyd
```

- hwclock  --systohc

  hwclock  --show 查看

- ntpdate ntp.aliyun.com

如果服务器和访问prometneus的时间不一致会提示: `Warning: Error fetching server time: Detected 48:181518 seconds time difference between your browser and the server.`此时需要将客户端和服务器同步一致即可

## 1.clone

k8s集群版本1.20.2

```
https://github.com.cnpmjs.org/prometheus-operator/kube-prometheus.git
```

切换到v0.8.0 `git checkout v0.8.0`

```
[root@master1 ~]# cd kube-prometheus/manifests/ 
[root@master1 kube-prometheus]# git checkout v0.8.0
Note: checking out 'v0.8.0'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b new_branch_name

HEAD is now at d2f8152... Merge pull request #1101 from paulfantom/cut-0.8
```

查找镜像

```

[root@master1 manifests]#  find ./ -type f |xargs grep 'image: '|sort|uniq|awk '{print $3}'|grep ^[a-zA-Z]|grep -Evw 'error|kubeRbacProxy'|sort -rn|uniq
quay.io/prometheus/prometheus:v2.26.0
quay.io/prometheus-operator/prometheus-operator:v0.47.0
quay.io/prometheus/node-exporter:v1.1.2
quay.io/prometheus/blackbox-exporter:v0.18.0
quay.io/prometheus/alertmanager:v0.21.0
quay.io/brancz/kube-rbac-proxy:v0.8.0
k8s.gcr.io/kube-state-metrics/kube-state-metrics:v2.0.0
jimmidyson/configmap-reload:v0.5.0
grafana/grafana:7.5.4
directxman12/k8s-prometheus-adapter:v0.8.4
```

准备这些镜像

```
quay.io/prometheus-operator/prometheus-operator:v0.47.0
quay.io/brancz/kube-rbac-proxy:v0.8.0
quay.io/prometheus/alertmanager:v0.21.0
quay.io/prometheus/blackbox-exporter:v0.18.0
k8s.gcr.io/kube-state-metrics/kube-state-metrics:v2.0.0
quay.io/prometheus/node-exporter:v1.1.2
quay.io/prometheus/prometheus:v2.26.0

jimmidyson/configmap-reload:v0.5.0 
grafana/grafana:7.5.4
directxman12/k8s-prometheus-adapter:v0.8.4
```

镜像已经搬运到dockerhub

```
marksugar/k8s-prometheus:prometheus-operator-v0.47.0
marksugar/k8s-prometheus:kube-rbac-proxy-v0.8.0
marksugar/k8s-prometheus:alertmanager-v0.21.0
marksugar/k8s-prometheus:blackbox-exporter-v0.18.0
marksugar/k8s-prometheus:kube-state-metrics-v2.0.0
marksugar/k8s-prometheus:node-exporter-v1.1.2
marksugar/k8s-prometheus:prometheus-v2.26.0
marksugar/k8s-prometheus:configmap-reload-v0.5.0
marksugar/k8s-prometheus:grafana-7.5.4
marksugar/k8s-prometheus:k8s-prometheus-adapter-v0.8.4
```

我这里使用本地装载`docker load -i IMAGE.TAG`到所有的节点，确保所有的节点都有镜像

```
for i in echo $(echo `ls `);do docker load -i $i; done
```

阿里云

```
registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/prometheus:v2.26.0
registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/prometheus-operator:v0.47.0
registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/kube-rbac-proxy:v0.8.0
registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/alertmanager:v0.21.0
registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/blackbox-exporter:v0.18.0
registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/kube-state-metrics:v2.0.0
registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/node-exporter:v1.1.2
registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/configmap-reload:v0.5.0
registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/grafana:7.5.4
registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/k8s-prometheus-adapter:0.8.4
```

替换阿里云

```
sed -i 's@quay.io/prometheus/prometheus:v2.26.0@registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/prometheus:v2.26.0@g' *.yaml
sed -i 's@quay.io/prometheus-operator/prometheus-operator:v0.47.0@registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/prometheus-operator:v0.47.0@g' *.yaml
sed -i 's@quay.io/brancz/kube-rbac-proxy:v0.8.0@registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/kube-rbac-proxy:v0.8.0@g' *.yaml
sed -i 's@quay.io/prometheus/alertmanager:v0.21.0@registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/alertmanager:v0.21.0@g' *.yaml
sed -i 's@quay.io/prometheus/blackbox-exporter:v0.18.0@registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/blackbox-exporter:v0.18.0@g' *.yaml
sed -i 's@k8s.gcr.io/kube-state-metrics/kube-state-metrics:v2.0.0@registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/kube-state-metrics:v2.0.0@g' *.yaml
sed -i 's@quay.io/prometheus/node-exporter:v1.1.2@registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/node-exporter:v1.1.2@g' *.yaml
sed -i 's@jimmidyson/configmap-reload:v0.5.0@registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/configmap-reload:v0.5.0@g' *.yaml
sed -i 's@grafana/grafana:7.5.4@registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/grafana:7.5.4@g' *.yaml
sed -i 's@directxman12/k8s-prometheus-adapter:v0.8.4@registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/k8s-prometheus-adapter:0.8.4@g' *.yaml
```

## 2.修改副本

我们修改配置文件，每个应用只启动一个`replicas: 1`。替换他们`sed -i 's/replicas: .*/replicas: 1/g'`

```
sed -i 's/replicas: .*/replicas: 1/g' prometheus-adapter-deployment.yaml
sed -i 's/replicas: .*/replicas: 1/g' alertmanager-alertmanager.yaml 
sed -i 's/replicas: .*/replicas: 1/g' prometheus-prometheus.yaml
```

> 1.16.3
>
> ```
> git checkout ee8077db04965d6b4d9e6a328d5283dd6ba71d33
> 或者
> git checkout v0.4.0
> ```
>
> 官方建议v0.3.0
>
> ```
> git checkout 0.3
> ```
>
> image v0.3.0:
>
> - 用源镜像下载
>
> ```
> quay.io/prometheus/node-exporter:v0.18.1
> quay.io/coreos/prometheus-operator:v0.34.0
> quay.io/coreos/kube-state-metrics:v1.8.0
> quay.io/coreos/kube-rbac-proxy:v0.4.1
> quay.io/coreos/k8s-prometheus-adapter-amd64:v0.5.0
> gcr.io/google_containers/metrics-server-amd64:v0.2.0
> quay.io/prometheus/prometheus:v2.11.0
> grafana/grafana:6.4.3
> quay.io/prometheus/alertmanager:v0.18.0
> ```
>
> ```
> docker pull  quay.io/prometheus/node-exporter:v0.18.1             
> docker pull  quay.io/coreos/prometheus-operator:v0.34.0	         
> docker pull  quay.io/coreos/kube-state-metrics:v1.8.0             
> docker pull  quay.io/coreos/kube-rbac-proxy:v0.4.1                
> docker pull  quay.io/coreos/k8s-prometheus-adapter-amd64:v0.5.0   
> docker pull  gcr.io/google_containers/metrics-server-amd64:v0.2.0 
> docker pull  quay.io/prometheus/prometheus:v2.11.0                
> docker pull  grafana/grafana:6.4.3                                
> docker pull  quay.io/prometheus/alertmanager:v0.18.0         
> ```
>
> - 用阿里云的镜像
>
> 镜像加速.而这些镜像已经被我修改
>
> ```
> sudo mkdir -p /etc/docker
> sudo tee /etc/docker/daemon.json <<-'EOF'
> {
>   "registry-mirrors": ["https://9ykgc5q2.mirror.aliyuncs.com"]
> }
> EOF
> sudo systemctl daemon-reload
> sudo systemctl restart docker
> ```
>
> 而后修改配置文件,替换镜像的地址后直接拉取,或者在本地推送
>
> ```
> sed -i 's@quay.io/prometheus/node-exporter:v0.18.1@registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/node-exporter:v0.18.1@g' *
> sed -i 's@quay.io/coreos/prometheus-operator:v0.34.0@registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/prometheus-operator:v0.34.0@g' *
> sed -i 's@quay.io/coreos/kube-state-metrics:v1.8.0@registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/kube-state-metrics:v1.8.0@g' *
> sed -i 's@quay.io/coreos/kube-rbac-proxy:v0.4.1@registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/kube-rbac-proxy:v0.4.1@g' *
> sed -i 's@quay.io/coreos/k8s-prometheus-adapter-amd64:v0.5.0@registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/k8s-prometheus-adapter-amd64:v0.5.0@g' *
> sed -i 's@gcr.io/google_containers/metrics-server-amd64:v0.2.0@registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/metrics-server-amd64:v0.2.0@g' *
> sed -i 's@quay.io/prometheus/prometheus:v2.11.0@registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/prometheus:v2.11.0@g' *
> sed -i 's@grafana/grafana:6.4.3@registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/grafana:6.4.3@g' *
> sed -i 's@quay.io/prometheus/alertmanager:v0.18.0@registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/alertmanager:v0.18.0@g' *
> ```
>
> > 本地镜像
>
> 1.修改推送到阿里云(已经推送)
>
> ```
> docker tag  quay.io/prometheus/node-exporter:v0.18.1                 registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/node-exporter:v0.18.1
> docker tag  quay.io/coreos/prometheus-operator:v0.34.0	             registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/prometheus-operator:v0.34.0
> docker tag  quay.io/coreos/kube-state-metrics:v1.8.0                 registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/kube-state-metrics:v1.8.0
> docker tag  quay.io/coreos/kube-rbac-proxy:v0.4.1                    registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/kube-rbac-proxy:v0.4.1
> docker tag  quay.io/coreos/k8s-prometheus-adapter-amd64:v0.5.0       registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/k8s-prometheus-adapter-amd64:v0.5.0
> docker tag  gcr.io/google_containers/metrics-server-amd64:v0.2.0     registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/metrics-server-amd64:v0.2.0 
> docker tag  quay.io/prometheus/prometheus:v2.11.0                    registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/prometheus:v2.11.0
> docker tag  grafana/grafana:6.4.3                                    registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/grafana:6.4.3 
> docker tag  quay.io/prometheus/alertmanager:v0.18.0                  registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/alertmanager:v0.18.0
> ```
>
> 2.或者直接pull已经有的
>
> ```
> docker pull registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/node-exporter:v0.18.1
> docker pull registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/prometheus-operator:v0.34.0
> docker pull registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/kube-state-metrics:v1.8.0
> docker pull registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/kube-rbac-proxy:v0.4.1
> docker pull registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/k8s-prometheus-adapter-amd64:v0.5.0
> docker pull registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/metrics-server-amd64:v0.2.0 
> docker pull registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/prometheus:v2.11.0
> docker pull registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/grafana:6.4.3 
> docker pull registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/alertmanager:v0.18.0
> ```
>
> 保存在本地
>
> ```
> mkdikr 0.4.0 && cd 0.4.0
> docker save registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/node-exporter:v0.18.1 -o node-exporter.tar
> docker save registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/prometheus-operator:v0.34.0 -o prometheus-operator.tar
> docker save registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/kube-state-metrics:v1.8.0 -o kube-state-metrics.tar
> docker save registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/kube-rbac-proxy:v0.4.1 -o kube-rbac-proxy.tar
> docker save registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/k8s-prometheus-adapter-amd64:v0.5.0 -o k8s-prometheus-adapter-amd64.tar
> docker save registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/metrics-server-amd64:v0.2.0 -o metrics-server-amd64.tar
> docker save registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/prometheus:v2.11.0 -o prometheus.tar
> docker save registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/grafana:6.4.3 -o grafana.tar
> docker save registry.cn-hangzhou.aliyuncs.com/marksugar-k8s/alertmanager:v0.18.0 -o alertmanager.tar
> ```
>
> 3.在本地tar包的目录进行导入
>
> ```
> tar -zcf 0.4.0.tar.gz 0.4.0
> tar xf 0.4.0.tar.gz
> 解压导入
> for i in `ls *.tar`;do docker load -i  $i;done
> ```
>
> 

## 3.修改端口配置

资源

```
        resources:
          limits:
            memory: 512Mi
          requests:
            memory: 256Mi
```

vi prometheus-service.yaml

```
...
spec:
  ports:
  - name: web
    port: 9090
    targetPort: web
  selector:
...
改成
...
spec:
  ports:
  - name: web
    port: 9090
    targetPort: web
    nodePort: 30090
  type: NodePort
  selector:
...
```

vi grafana-service.yaml 

```
...
spec:
  ports:
  - name: http
    port: 3000
    targetPort: http
  selector:
...
改成
...
spec:
  ports:
  - name: http
    port: 3000
    nodePort: 30091
  type: NodePort    
    targetPort: http
  selector:
...
```

vi  alertmanager-service.yaml 

```
...
spec:
  ports:
  - name: web
    port: 9093
    targetPort: web
  selector:
...
改成
...
spec:
  ports:
  - name: web
    port: 9093
    targetPort: web
    nodePort: 30092
  type: NodePort      
  selector:
...
```

修改存储时间

vi prometheus-prometheus.yaml 

```
...
name: k8s
  namespace: monitoring
spec:
  alerting:
    alertmanagers:
...
添加
...
  name: k8s
  namespace: monitoring
spec:
  retention: 7d
  alerting:
    alertmanagers:
...    
```

## 4.部署

```
[root@master1 ~]# kubectl apply -f kube-prometheus/manifests/setup/
namespace/monitoring created
customresourcedefinition.apiextensions.k8s.io/alertmanagerconfigs.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/alertmanagers.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/podmonitors.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/probes.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/prometheuses.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/prometheusrules.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/servicemonitors.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/thanosrulers.monitoring.coreos.com created
clusterrole.rbac.authorization.k8s.io/prometheus-operator created
clusterrolebinding.rbac.authorization.k8s.io/prometheus-operator created
deployment.apps/prometheus-operator created
service/prometheus-operator created
serviceaccount/prometheus-operator created
```

等待就绪

```
[root@master1 ~]# kubectl -n monitoring get all
NAME                                       READY   STATUS    RESTARTS   AGE
pod/prometheus-operator-7775c66ccf-25976   2/2     Running   0          78s

NAME                          TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
service/prometheus-operator   ClusterIP   None         <none>        8443/TCP   79s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/prometheus-operator   1/1     1            1           80s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/prometheus-operator-7775c66ccf   1         1         1       79s
```

创建manifests下的yaml

```
kubectl apply -f kube-prometheus/manifests/
```

查看就绪状态

```
[root@master1 ~]# kubectl -n monitoring get all
NAME                                       READY   STATUS    RESTARTS   AGE
pod/alertmanager-main-0                    2/2     Running   0          50s
pod/blackbox-exporter-55c457d5fb-r4wwk     3/3     Running   0          50s
pod/grafana-9df57cdc4-g68v6                1/1     Running   0          48s
pod/kube-state-metrics-76f6cb7996-2ndp6    3/3     Running   0          48s
pod/node-exporter-g6d67                    2/2     Running   0          48s
pod/node-exporter-nlljk                    2/2     Running   0          48s
pod/node-exporter-pzxgh                    2/2     Running   0          48s
pod/prometheus-adapter-59df95d9f5-vzd8d    1/1     Running   0          48s
pod/prometheus-k8s-0                       2/2     Running   1          46s
pod/prometheus-operator-7775c66ccf-25976   2/2     Running   0          2m41s

NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/alertmanager-main       NodePort    10.68.111.60    <none>        9093:30092/TCP               51s
service/alertmanager-operated   ClusterIP   None            <none>        9093/TCP,9094/TCP,9094/UDP   52s
service/blackbox-exporter       ClusterIP   10.68.71.98     <none>        9115/TCP,19115/TCP           51s
service/grafana                 NodePort    10.68.181.96    <none>        3000:30091/TCP               50s
service/kube-state-metrics      ClusterIP   None            <none>        8443/TCP,9443/TCP            49s
service/node-exporter           ClusterIP   None            <none>        9100/TCP                     49s
service/prometheus-adapter      ClusterIP   10.68.10.196    <none>        443/TCP                      48s
service/prometheus-k8s          NodePort    10.68.189.126   <none>        9090:30090/TCP               47s
service/prometheus-operated     ClusterIP   None            <none>        9090/TCP                     47s
service/prometheus-operator     ClusterIP   None            <none>        8443/TCP                     2m42s

NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/node-exporter   3         3         3       3            3           kubernetes.io/os=linux   49s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/blackbox-exporter     1/1     1            1           51s
deployment.apps/grafana               1/1     1            1           50s
deployment.apps/kube-state-metrics    1/1     1            1           49s
deployment.apps/prometheus-adapter    1/1     1            1           48s
deployment.apps/prometheus-operator   1/1     1            1           2m43s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/blackbox-exporter-55c457d5fb     1         1         1       51s
replicaset.apps/grafana-9df57cdc4                1         1         1       50s
replicaset.apps/kube-state-metrics-76f6cb7996    1         1         1       49s
replicaset.apps/prometheus-adapter-59df95d9f5    1         1         1       48s
replicaset.apps/prometheus-operator-7775c66ccf   1         1         1       2m42s

NAME                                 READY   AGE
statefulset.apps/alertmanager-main   1/1     52s
statefulset.apps/prometheus-k8s      1/1     47s
```

通过30090，30091，30092访问

###  4.1 集群监控添加

- 二进制安装需要配置

```
serviceMonitor/monitoring/kube-controller-manager/0 (0/0 up) 
serviceMonitor/monitoring/kube-scheduler/0 (0/0 up) 
```

如果controller和scheduler端口监听在127.0.0.1则需要修改

```
sed -ri 's+127.0.0.1+0.0.0.0+g' /etc/systemd/system/kube-controller-manager.service 
#sed -ri 's+127.0.0.1+0.0.0.0+g' /etc/systemd/system/kube-scheduler.service
```

为了能让K8s上的prometheus能发现，我们还需要来创建相应的service和endpoints来将其关联起来

```
apiVersion: v1
kind: Service
metadata:
  namespace: kube-system
  name: kube-controller-manager
  labels:
  labels:
    app.kubernetes.io/name: kube-controller-manager
spec:
  type: ClusterIP
  clusterIP: None
  ports:
  - name: http-metrics
    port: 10252
    targetPort: 10252
    protocol: TCP

---
apiVersion: v1
kind: Endpoints
metadata:
  labels:
    app.kubernetes.io/name: kube-controller-manager
  name: kube-controller-manager
  namespace: kube-system
subsets:
- addresses:
  - ip: 172.16.1.79
  - ip: 172.16.1.34
  - ip: 172.16.1.242
  ports:
  - name: http-metrics
    port: 10252
    protocol: TCP

---

apiVersion: v1
kind: Service
metadata:
  namespace: kube-system
  name: kube-scheduler
  labels:
    app.kubernetes.io/name: kube-scheduler
spec:
  type: ClusterIP
  clusterIP: None
  ports:
  - name: http-metrics
    port: 10251
    targetPort: 10251
    protocol: TCP

---
apiVersion: v1
kind: Endpoints
metadata:
  labels:
    app.kubernetes.io/name: kube-scheduler
  name: kube-scheduler
  namespace: kube-system
subsets:
- addresses:
  - ip: 192.168.3.100
  - ip: 192.168.3.101
  ports:
  - name: http-metrics
    port: 10251
    protocol: TCP
```

修改`kubectl -n monitoring edit servicemonitors.monitoring.coreos.com kube-scheduler `

```
...
    port: https-metrics
    scheme: https
...
修改为http
...
    port: http-metrics
    scheme: http
...
```

修改` kubectl -n monitoring edit servicemonitors.monitoring.coreos.com kube-controller-manager`

```
...
    port: https-metrics
    scheme: https
...
改成
...
    port: http-metrics
    scheme: http
...
```



apply

```
[root@master1 base-prometheus]# kubectl apply -f controller-scheduler.yaml
service/kube-controller-manager created
endpoints/kube-controller-manager created
service/kube-scheduler created
endpoints/kube-scheduler created
```

```
[root@master1 base-prometheus]# kubectl -n kube-system get svc |egrep 'controller|scheduler'
kube-controller-manager   ClusterIP   None            <none>        10252/TCP                      6m7s
kube-scheduler            ClusterIP   None            <none>        10251/TCP                      6m6s
```

- kubeadm

```
apiVersion: v1
kind: Service
metadata:
  namespace: kube-system
  name: kube-controller-manager
  labels:
    k8s-app: kube-controller-manager
spec:
  ports:
  - name: https-metrics
    port: 10257
  selector:
    component: kube-controller-manager
---
apiVersion: v1
kind: Service
metadata:
  namespace: kube-system
  name: kube-scheduler
  labels:
    k8s-app: kube-scheduler
spec:
  ports:
  - name: https-metrics
    port: 10259
  selector:
    component: kube-scheduler
```

在 /etc/kubernetes/manifests修改kube-controller-manager.yaml和kube-scheduler.yaml的--bind-address是0.0.0.0

而后创建上面的Service

### 4.2 etcd

- pod

> ```csharp
> curl -k  --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/peer.crt --key /etc/kubernetes/pki/etcd/peer.key https://127.0.0.1:2379/metrics
> ```
> ## 1.secret
>
> ```
> kubectl -n monitoring create secret generic etcd-certs --from-file=/etc/kubernetes/pki/etcd/ca.crt --from-file=/etc/kubernetes/pki/etcd/peer.crt    --from-file=/etc/kubernetes/pki/etcd/peer.key
> ```
>
> ## 2.yaml
>
> etcd pod的密钥文件路径是容器内的路径
>
> ```
> apiVersion: v1
> kind: Service
> metadata:
> name: etcd-k8s
> namespace: monitoring
> labels:
> k8s-app: etcd
> spec:
> type: ClusterIP
> clusterIP: None
> ports:
>   - name: api
>     port: 2379
>     protocol: TCP
> ---
> apiVersion: v1
> kind: Endpoints
> metadata:
>   name: etcd-k8s
>   namespace: monitoring
>   labels:
>     k8s-app: etcd
> subsets:
> - addresses:
>   - ip: 10.59.8.151
>   ports:
>   - name: api
>     port: 2379
>     protocol: TCP
> ---
> apiVersion: monitoring.coreos.com/v1
> kind: ServiceMonitor
> metadata:
>   name: etcd-k8s
>   namespace: monitoring
>   labels:
>     k8s-app: etcd-k8s
> spec:
>   jobLabel: k8s-app
>   endpoints:
>   - port: api
>     interval: 30s
>     scheme: https
>     tlsConfig:
>       caFile: /etc/prometheus/secrets/etcd-certs/ca.crt
>       certFile: /etc/prometheus/secrets/etcd-certs/peer.crt
>       keyFile: /etc/prometheus/secrets/etcd-certs/peer.key
>       #use insecureSkipVerify only if you cannot use a Subject Alternative Name
>       insecureSkipVerify: true
>   selector:
>     matchLabels:
>       k8s-app: etcd
>   namespaceSelector:
>     matchNames:
>     - monitoring
> ```
>
> 1
>
> ```
> # kubectl -n monitoring get ServiceMonitor | grep  etcd-k8s
> etcd-k8s                  2m4s
> # kubectl -n monitoring get secret | grep etcd
> etcd-certs                        Opaque                                3      52s
> ```
> ## 3.引用secrets
>
> 接着在prometheus里面引用这个secrets
>
> `kubectl -n monitoring edit prometheus k8s `
>
> ```
> ...
> spec:
>   alerting:
>     alertmanagers:
> ...
> 改为
> ...
> spec:
> ...
> spec:
>   secrets:
>   - etcd-certs
>     alerting:
>     alertmanagers:
>     ...
> ```
>
> 验证
>
> ```
> [root@master1 base-prometheus]# kubectl -n monitoring exec -it prometheus-k8s-0 -c prometheus  -- sh
> /prometheus $  ls /etc/prometheus/secrets/etcd-certs/
> ca.pem        etcd-key.pem  etcd.pem
> ```
>
> 
>
> 

- 二进制

测试连通性

```
 curl --cacert /etc/kubernetes/ssl/ca.pem --cert /etc/kubernetes/ssl/etcd.pem  --key /etc/kubernetes/ssl/etcd-key.pem  https://192.168.201.96:2379/metrics
```

创建etcd为secret

```
kubectl -n monitoring create secret generic etcd-certs --from-file=/etc/kubernetes/ssl/etcd.pem   --from-file=/etc/kubernetes/ssl/etcd-key.pem    --from-file=/etc/kubernetes/ssl/ca.pem
```
接着在prometheus里面引用这个secrets

`kubectl -n monitoring edit prometheus k8s `

```
...
spec:
  alerting:
    alertmanagers:
...
改为
...
spec:
...
spec:
  secrets:
  - etcd-certs
    alerting:
    alertmanagers:
    ...
```

保存退出后，prometheus会自动重启服务pod以加载这个secret配置，过一会，我们进pod来查看下是不是已经加载到ETCD的证书了

```
[root@master1 base-prometheus]# kubectl -n monitoring exec -it prometheus-k8s-0 -c prometheus  -- sh
/prometheus $  ls /etc/prometheus/secrets/etcd-certs/
ca.pem        etcd-key.pem  etcd.pem
```

创建service、endpoints以及ServiceMonitor的yaml配置

```
apiVersion: v1
kind: Service
metadata:
  name: etcd-k8s
  namespace: monitoring
  labels:
    k8s-app: etcd
spec:
  type: ClusterIP
  clusterIP: None
  ports:
  - name: api
    port: 2379
    protocol: TCP
---
apiVersion: v1
kind: Endpoints
metadata:
  name: etcd-k8s
  namespace: monitoring
  labels:
    k8s-app: etcd
subsets:
- addresses:
  - ip: 172.16.1.79
  - ip: 172.16.1.34
  - ip: 172.16.1.242
  ports:
  - name: api
    port: 2379
    protocol: TCP
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: etcd-k8s
  namespace: monitoring
  labels:
    k8s-app: etcd-k8s
spec:
  jobLabel: k8s-app
  endpoints:
  - port: api
    interval: 30s
    scheme: https
    tlsConfig:
      caFile: /etc/prometheus/secrets/etcd-certs/ca.pem
      certFile: /etc/prometheus/secrets/etcd-certs/etcd.pem
      keyFile: /etc/prometheus/secrets/etcd-certs/etcd-key.pem
      #use insecureSkipVerify only if you cannot use a Subject Alternative Name
      insecureSkipVerify: true
    selector:
    matchLabels:
      k8s-app: etcd
    namespaceSelector:
    matchNames:
    - monitoring
```

### 4.3 kube-proxy

```
apiVersion: v1
kind: Service
metadata:
  name: external-kube-proxy
  namespace: monitoring
  labels:
    k8s-app: kube-proxy
spec:
  type: ClusterIP
  clusterIP: None
  ports:
  - name: external-kube-proxy
    port: 10249
    protocol: TCP
---
apiVersion: v1
kind: Endpoints
metadata:
  name: external-kube-proxy
  namespace: monitoring
  labels:
    k8s-app: kube-proxy
subsets:
- addresses:
  - ip: 17.168.0.165
  - ip: 17.168.0.166
  - ip: 17.168.0.167
  - ip: 17.168.0.168
  - ip: 17.168.0.169
  - ip: 17.168.0.170
  - ip: 17.168.0.171  
  ports:
  - name: external-kube-proxy
    port: 10249
    protocol: TCP
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: external-kube-proxy
  namespace: monitoring
  labels:
    k8s-app: kube-proxy
spec:
  jobLabel: k8s-app
  endpoints:
  - port: external-kube-proxy
    path: /metrics
    interval: 30s
    scheme: http
    selector:
    matchLabels:
      k8s-app: kube-proxy
    namespaceSelector:
    matchNames:
    - monitoring
```

### 4.4 ingress-nginx

```
curl 10.0.1.201:10254/metrics
```



```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    app: ingress-nginx
  name: nginx-ingress-scraping
  namespace: ingress-nginx
spec:
  endpoints:
  - interval: 30s
    path: /metrics
    port: metrics
    jobLabel: app
    namespaceSelector:
    matchNames:
    - ingress-nginx
    selector:
      matchLabels:
      app: ingress-nginx
```

查看是否有错误

```
kubectl -n monitoring logs prometheus-k8s-0 -c prometheus |grep error
```

需要修改prometheus的clusterrole

```
#   kubectl edit clusterrole prometheus-k8s
#------ 原始的rules -------
rules:
- apiGroups:
  - ""
  resources:
  - nodes/metrics
  verbs:
  - get
- nonResourceURLs:
  - /metrics
  verbs:
  - get
  #---------------------------

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus-k8s
rules:
- apiGroups:
  - ""
  resources:
  - nodes
  - services
  - endpoints
  - pods
  - nodes/proxy
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - configmaps
  - nodes/metrics
  verbs:
  - get
- nonResourceURLs:
  - /metrics
  verbs:
  - get
```



## 5.修改存储位置

### 5.1 prometheus

创建一个新的class.yaml

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-prom
provisioner: fuseim.pri/ifs # or choose another name, must match deployment's env PROVISIONER_NAME'
parameters:
  archiveOnDelete: "false"
# Supported policies: Delete、 Retain ， default is Delete
reclaimPolicy: Retain
```

查看下是否创建成功

```
[root@master1 base-prometheus]# kubectl get sc
NAME                  PROVISIONER      RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
managed-nfs-storage   fuseim.pri/ifs   Retain          Immediate           false                  47m
nfs-prom              fuseim.pri/ifs   Retain          Immediate           false                  5m41s
```

修改`kubectl -n monitoring edit prometheus k8s`

```
spec:
....
  storage:
    volumeClaimTemplate:
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 10Gi
        storageClassName: nfs-prom
```

查看pv,pvc

```
# pvc
[root@master1 base-prometheus]# kubectl get pvc -n monitoring
NAME                                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
prometheus-k8s-db-prometheus-k8s-0   Bound    pvc-5a72084e-a840-4a76-a8f8-a012ea6a8278   10Gi       RWO            nfs-prom       36m
# pv
[root@master1 base-prometheus]# kubectl get pv -A
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM        
pvc-5a72084e-a840-4a76-a8f8-a012ea6a8278   10Gi       RWO            Delete           Bound    monitoring/prometheus-k8s-db-prometheus-k8s-0   nfs-prom  
```

nfs查看

```
[root@Node-192_168_3_19 /data/nfs-k8s]# du -sh monitoring-prometheus-k8s-db-prometheus-k8s-0-pvc-5a72084e-a840-4a76-a8f8-a012ea6a8278/prometheus-db/
34M	monitoring-prometheus-k8s-db-prometheus-k8s-0-pvc-5a72084e-a840-4a76-a8f8-a012ea6a8278/prometheus-db/
```

### 5.2 grafana

创建grafana-pvc.yaml

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: grafana
  namespace: monitoring
spec:
  storageClassName: nfs-prom
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```

创建后查看

```
[root@master1 base-prometheus]# kubectl get pvc -n monitoring
NAME                                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
grafana                              Bound    pvc-49fe9113-b09a-45bf-89f5-a747abe96918   1Gi        RWX            nfs-prom       19s
prometheus-k8s-db-prometheus-k8s-0   Bound    pvc-5a72084e-a840-4a76-a8f8-a012ea6a8278   10Gi       RWO   
```

辑grafana的deployment资源配置`kubectl -n monitoring edit deployments.apps grafana `

```
# 旧的配置
      volumes:
      - emptyDir: {}
        name: grafana-storage
# 替换成新的配置
      volumes:
      - name: grafana-storage
        persistentVolumeClaim:
          claimName: grafana
```

同时修改账户和密码

```
...
    spec:
      containers:
      - env:
        - name: GF_SECURITY_ADMIN_USER
          value: mark
        - name: GF_SECURITY_ADMIN_PASSWORD
          value: linuxea.com
```

查看

```
[root@Node-192_168_3_19 /data/nfs-k8s]# du -sh monitoring-grafana-pvc-49fe9113-b09a-45bf-89f5-a747abe96918/
1.5M	monitoring-grafana-pvc-49fe9113-b09a-45bf-89f5-a747abe96918/
```

>修改grafana密码
>
>进入nfs中 sqlite3 grafana.db
>
>```
>[root@nfs-sto /data/nfs-share/monitoring-grafana-pvc-27d32bf1-7b68-4b11-92de-6dd95b74c8ee]# sqlite3 grafana.db
>SQLite version 3.7.17 2013-05-20 00:56:22
>Enter ".help" for instructions
>Enter SQL statements terminated with a ";"
>sqlite> .tables
>alert                     dashboard_tag             server_lock
>alert_notification        dashboard_version         session
>alert_notification_state  data_source               short_url
>alert_rule_tag            login_attempt             star
>annotation                migration_log             tag
>annotation_tag            org                       team
>api_key                   org_user                  team_member
>cache_data                playlist                  temp_user
>dashboard                 playlist_item             test_data
>dashboard_acl             plugin_setting            user
>dashboard_provisioning    preferences               user_auth
>dashboard_snapshot        quota                     user_auth_token
>sqlite> select * from user;
>1|0|admin|admin@localhost||660c7c730f3b449e99f6656fbaf362f2f312ea7bede29c0ac0833ebc1017bd44f9d4cc95e9f7e2117d5c0da119ecee8fc7da|xJQNbuc9OS|Bmc3w28tpt||1|1|0||2021-05-24 05:01:27|2021-05-24 05:11:08|0|2021-05-24 09:35:33|0
>sqlite> update user set password = '59acf18b94d7eb0694c61e60ce44c110c7a683ac6a8f09580d626f90f4a242000746579358d77dd9e570e83fa24faa88a8a6', salt = 'F3FAxVm33R' where login = 'admin';
>sqlite> 
>```

## 6.修改时区

prometheus-prometheus.yaml

alertmanager-alertmanager.yaml

```
spec:
...
  volumes:
    - name: localtime
      hostPath:
        path: /etc/localtime
    - name: zoneinfo
      hostPath:
        path: /usr/share/zoneinfo
  volumeMounts:
    - name: localtime
      mountPath: /etc/localtime
      readOnly: true
    - name: zoneinfo
      mountPath: /usr/share/zoneinfo
      readOnly: true
```

grafana-deployment.yaml

```
       volumeMounts:
        - mountPath: /etc/localtime
          name: localtime
          readOnly: true
        - mountPath: /usr/share/zoneinfo
          name: zoneinfo
          readOnly: true
```

```
      volumes:
      - name: localtime
        hostPath:
          path: /etc/localtime
      - name: zoneinfo
        hostPath:
          path: /usr/share/zoneinfo
```

## 7.webhook

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    name: kube-eventer
  name: kube-eventer
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kube-eventer
  template:
    metadata:
      labels:
        app: kube-eventer
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ''
    spec:
      dnsPolicy: ClusterFirstWithHostNet
      serviceAccount: kube-eventer
      containers:
        - image: registry.aliyuncs.com/acs/kube-eventer-amd64:v1.2.0-484d9cd-aliyun
          name: kube-eventer
          command:
            - "/kube-eventer"
            - "--source=kubernetes:https://kubernetes.default"
            - --sink=dingtalk:https://oapi.dingtalk.com/robot/send?access_token=bcef6588c33fdc3f43dfabf696a7e587925f503&label=测试&level=Warning
          env:
          # If TZ is assigned, set the TZ value as the time zone
          - name: TZ
            value: "Asia/Shanghai"
          volumeMounts:
            - name: localtime
              mountPath: /etc/localtime
              readOnly: true
            - name: zoneinfo
              mountPath: /usr/share/zoneinfo
              readOnly: true
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
            limits:
              cpu: 500m
              memory: 250Mi
      volumes:
        - name: localtime
          hostPath:
            path: /etc/localtime
        - name: zoneinfo
          hostPath:
            path: /usr/share/zoneinfo
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: kube-eventer
rules:
  - apiGroups:
      - ""
      resources:
      - configmaps
      - events
      verbs:
      - get
      - list
      - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kube-eventer
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kube-eventer
subjects:
  - kind: ServiceAccount
    name: kube-eventer
    namespace: monitoring
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kube-eventer
  namespace: monitoring
```

## 8.监控node

### 8.1 安装docker

```
yum install -y yum-utils
yum-config-manager \
    --add-repo \
    https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io -y    
systemctl start docker
systemctl enable docker
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
 chmod +x /usr/local/bin/docker-compose
 ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose --version
docker -v
```

mkdir /usr/local/prom_exporter -p









```
/etc/prometheus/prometheus.yml \
```

docker pull marksugar/k8s-prometheus:grafana-7.5.4

```
docker pull  prom/prometheus:2.27.1
```

docker pull  grafana/grafana:8.0.3

docker pull marksugar/k8s-prometheus:alertmanager-v0.21.0

docker pull marksugar/k8s-prometheus:prometheus-v2.26.0

docker pull marksugar/k8s-prometheus:node-exporter-v1.1.2

```
version: '3.7'
services:
  node_exporter:
    image: quay.io/prometheus/node-exporter:v1.1.2
    #image: marksugar/k8s-prometheus:node-exporter-v1.1.2
    container_name: node_exporter
    command:
      - '--path.rootfs=/host'
    network_mode: host
    pid: host
    restart: unless-stopped
    volumes:
      - '/:/host:ro,rslave'
      - /etc/localtime:/etc/localtime:ro
    logging:
      driver: "json-file"
      options:
        max-size: "50M"
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 256M
  kafka_exporter:
    container_name: kafka_exporter
    image: danielqsj/kafka-exporter:v1.3.1
    network_mode: host
    command:
      - --kafka.server=17.168.0.172:9092
      - --sasl.enabled
      - --sasl.username=markadmin
      - --sasl.password=MwMzA0MGIwZjMwMjg3MjY4NWE2ZGFmOG
      - --sasl.mechanism=scram-sha256
      - --web.listen-address=0.0.0.0:9308
      - --log.level=debug
    restart: always
    environment:
     - JAVA_OPTS=-Duser.timezone=Asia/Shanghai  # 时区1
    volumes:
     - /etc/localtime:/etc/localtime:ro  # 时区2
    logging:
      driver: "json-file"
      options:
        max-size: "50M"
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 256M
```

```
docker-compose --compatibility -f docker-compose.yaml up -d
```

监控集群外节点

```
apiVersion: v1
kind: Service
metadata:
  name: external-node-k8s
  namespace: monitoring
  labels:
    k8s-app: external-node
spec:
  type: ClusterIP
  clusterIP: None
  ports:
  - name: external-node-port
    port: 9100
    protocol: TCP
---
apiVersion: v1
kind: Endpoints
metadata:
  name: external-node-k8s
  namespace: monitoring
  labels:
    k8s-app: external-node
subsets:
- addresses:
  - ip: 192.168.63.7
  - ip: 192.168.63.8
  - ip: 192.168.63.9
  - ip: 192.168.63.10
  - ip: 192.168.63.11
  - ip: 192.168.63.12
  - ip: 192.168.63.17
  - ip: 192.168.63.18
  - ip: 192.168.63.14
  - ip: 192.168.63.15
  - ip: 192.168.63.16
  - ip: 192.168.63.13
  ports:
  - name: external-node-port
    port: 9100
    protocol: TCP
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: external-node-k8s
  namespace: monitoring
  labels:
    k8s-app: external-node
spec:
  jobLabel: k8s-app
  endpoints:
  - port: external-node-port
    path: /metrics
    interval: 30s
    scheme: http
    selector:
    matchLabels:
      k8s-app: external-node
    namespaceSelector:
    matchNames:
    - monitoring
```

```
https://grafana.com/grafana/dashboards/8919
```

```
firewall-cmd --permanent --zone=public   --add-rich-rule='rule family="ipv4" source address="192.168.201.0/24" port port="9100" protocol="tcp" accept'
firewall-cmd --permanent --zone=public   --add-rich-rule='rule family="ipv4" source address="192.168.201.0/24" port port="3306" protocol="tcp" accept'
firewall-cmd --reload
```

firewall-cmd --permanent --zone=public   --add-rich-rule='rule family="ipv4" source address="192.168.29.0/24" port port="9100" protocol="tcp" accept'

firewall-cmd --reload

- 监控windows

下载

`https://github.com/prometheus-community/windows_exporter`

开启防火墙

```
netsh advfirewall firewall add rule name="windows_exporter" protocol=TCP dir=in localport=9182 action=allow
netsh advfirewall firewall delete rule name="windows_exporter"
netsh advfirewall firewall add rule name="windows_exporter" protocol=TCP dir=in localport=9182 action=allow 
```

启动

```
"C:\Program Files\windows_exporter\windows_exporter.exe" --collectors.enabled "cpu,cs,net,service,memory,logon,process,logical_disk,os,tcp,time" --collector.service.services-where "Name='windows_exporter'"
```



```
apiVersion: v1
kind: Service
metadata:
  name: external-windows-node-k8s
  namespace: monitoring
  labels:
    k8s-app: external-windows-node
spec:
  type: ClusterIP
  clusterIP: None
  ports:
  - name: external-node-port
    port: 9182
    protocol: TCP
---
apiVersion: v1
kind: Endpoints
metadata:
  name:  external-windows-node-k8s
  namespace: monitoring
  labels:
    k8s-app: external-windows-node
subsets:
- addresses:
  - ip: 192.168.63.19
  ports:
  - name: external-node-port
    port: 9182
    protocol: TCP
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name:  external-windows-node-k8s
  namespace: monitoring
  labels:
    k8s-app: external-windows-node
spec:
  jobLabel: k8s-app
  endpoints:
  - port: external-node-port
    path: /metrics
    interval: 30s
    scheme: http
    selector:
    matchLabels:
      k8s-app: external-windows-node
    namespaceSelector:
    matchNames:
    - monitoring
```



## 9.监控mysql

```
apiVersion: v1
kind: Service
metadata:
  name: external-mysql-k8s
  namespace: monitoring
  labels:
    k8s-app: external-mysql
spec:
  type: ClusterIP
  clusterIP: None
  ports:
  - name: external-mysql-port
    port: 9104
    protocol: TCP
---
apiVersion: v1
kind: Endpoints
metadata:
  name: external-mysql-k8s
  namespace: monitoring
  labels:
    k8s-app: external-mysql
subsets:
- addresses:
  - ip: 192.168.63.17
  - ip: 192.168.63.18
  ports:
  - name: external-mysql-port
    port: 9104
    protocol: TCP
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: external-mysql-k8s
  namespace: monitoring
  labels:
    k8s-app: external-mysql
spec:
  jobLabel: k8s-app
  endpoints:
  - port: external-mysql-port
    path: /metrics
    interval: 30s
    scheme: http
    selector:
    matchLabels:
      k8s-app: external-mysql
    namespaceSelector:
    matchNames:
    - monitoring
```



```
CREATE USER 'monitoring'@'localhost' IDENTIFIED BY 'kZDA5MjM1MTNiZmQxNGNiM2EyODRhN';
GRANT PROCESS, REPLICATION CLIENT ON *.* TO 'monitoring'@'localhost';
GRANT SELECT ON performance_schema.* TO 'monitoring'@'localhost';

CREATE USER 'monitoring'@'127.0.0.1' IDENTIFIED BY 'kZDA5MjM1MTNiZmQxNGNiM2EyODRhN';
GRANT PROCESS, REPLICATION CLIENT ON *.* TO 'monitoring'@'127.0.0.1';
GRANT SELECT ON performance_schema.* TO 'monitoring'@'127.0.0.1';
```



```
version: '3.8'
services:
  mysql:
    container_name: mysql_exporter
    image: prom/mysqld-exporter:v0.13.0
    network_mode: host
    environment:
    - DATA_SOURCE_NAME=monitoring:kZDA5MjM1MTNiZmQxNGNiM2EyODRhN@(127.0.0.1:3016)/    
    restart: always
    volumes:
     - /etc/localtime:/etc/localtime:ro  # 时区2
    logging:
      driver: "json-file"
      options:
        max-size: "50M"       
```



```
firewall-cmd --permanent --zone=public   --add-rich-rule='rule family="ipv4" source address="192.168.201.0/24" port port="9104" protocol="tcp" accept'
firewall-cmd --reload
```

`dashboard:https://grafana.com/grafana/dashboards/11329 `



### 9.1 mysql 监控参数

```
version: '3.8'
services:
  mysql:
    container_name: mysql_exporter
    image: prom/mysqld-exporter:v0.12.1
    network_mode: host
    environment:
    - DATA_SOURCE_NAME=monitoring:kZDA5MjM1MTNiZmQxNGNiM2EyODRhN@(127.0.0.1:3016)/
    command: --collect.info_schema.processlist --collect.info_schema.processlist.processes_by_user --collect.info_schema.processlist.processes_by_host  --collect.perf_schema.tablelocks --collect.perf_schema.tableiowaits --collect.perf_schema.indexiowaits --collect.info_schema.query_response_time --collect.perf_schema.eventsstatements --collect.perf_schema.eventsstatementssum --collect.perf_schema.eventswaits --collect.engine_innodb_status --collect.global_status --collect.global_variables --collect.info_schema.clientstats --collect.info_schema.innodb_metrics --collect.info_schema.innodb_tablespaces --collect.info_schema.innodb_cmp --collect.info_schema.innodb_cmpmem
```

nohub

```
export DATA_SOURCE_NAME='monitoring:kZDA5MjM1MTNiZmQxNGNiM2EyODRhN@(127.0.0.1:3016)/'
nohup ./mysqld_exporter --collect.info_schema.processlist --collect.info_schema.processlist.processes_by_user --collect.info_schema.processlist.processes_by_host  --collect.perf_schema.tablelocks --collect.perf_schema.tableiowaits --collect.perf_schema.indexiowaits --collect.info_schema.query_response_time --collect.perf_schema.eventsstatements --collect.perf_schema.eventsstatementssum --collect.perf_schema.eventswaits --collect.engine_innodb_status --collect.global_status --collect.global_variables --collect.info_schema.clientstats --collect.info_schema.innodb_metrics --collect.info_schema.innodb_tablespaces --collect.info_schema.innodb_cmp --collect.info_schema.innodb_cmpmem  2>&1 &
```

```
firewall-cmd --permanent --zone=public   --add-rich-rule='rule family="ipv4" source address="10.10.160.0/24" port port="3306" protocol="tcp" accept'
firewall-cmd --permanent --zone=public   --add-rich-rule='rule family="ipv4" source address="10.10.160.0/24" port port="9104" protocol="tcp" accept'
firewall-cmd --permanent --zone=public   --add-rich-rule='rule family="ipv4" source address="10.10.160.0/24" port port="9100" protocol="tcp" accept'
firewall-cmd --reload
```



如下

```
--collect.info_schema.processlist --collect.info_schema.processlist.processes_by_user --collect.info_schema.processlist.processes_by_host  --collect.perf_schema.tablelocks --collect.perf_schema.tableiowaits --collect.perf_schema.indexiowaits --collect.info_schema.query_response_time --collect.perf_schema.eventsstatements --collect.perf_schema.eventsstatementssum --collect.perf_schema.eventswaits --collect.engine_innodb_status --collect.global_status --collect.global_variables --collect.info_schema.clientstats --collect.info_schema.innodb_metrics --collect.info_schema.innodb_tablespaces --collect.info_schema.innodb_cmp --collect.info_schema.innodb_cmpmem
```

- innodb

```
--collect.engine_innodb_status 
--collect.global_status 
--collect.global_variables 
--collect.info_schema.clientstats 
--collect.info_schema.innodb_metrics 
--collect.info_schema.innodb_tablespaces 
--collect.info_schema.innodb_cmp 
--collect.info_schema.innodb_cmpmem 
```

```
mysql_info_schema_innodb_cmp_compress_ops_ok_total
mysql_info_schema_innodb_cmp_compress_ops_total
mysql_info_schema_innodb_cmp_compress_time_seconds_total
mysql_info_schema_innodb_cmpmem_pages_free_total
mysql_info_schema_innodb_cmpmem_pages_used_total
mysql_info_schema_innodb_cmpmem_relocation_ops_total
mysql_info_schema_innodb_cmpmem_relocation_time_seconds_total
mysql_info_schema_innodb_cmp_uncompress_ops_total
mysql_info_schema_innodb_cmp_uncompress_time_seconds_total
mysql_info_schema_innodb_metrics_adaptive_hash_index_adaptive_hash_searches_btree_total 6.9549685e+09
mysql_info_schema_innodb_metrics_adaptive_hash_index_adaptive_hash_searches_total 8.2132691511e+10
mysql_info_schema_innodb_metrics_buffer_buffer_data_reads_total 2.7507347968e+10
mysql_info_schema_innodb_metrics_buffer_buffer_data_written_total 1.28752345088e+12
mysql_info_schema_innodb_metrics_buffer_buffer_pages_created_total 390292
mysql_info_schema_innodb_metrics_buffer_buffer_pages_read_total 1.678911e+06
mysql_info_schema_innodb_metrics_buffer_buffer_pages_written_total 3.7520226e+07
mysql_info_schema_innodb_metrics_buffer_buffer_pool_bytes_data 8.112832512e+09
mysql_info_schema_innodb_metrics_buffer_buffer_pool_bytes_dirty 9.6878592e+07
mysql_info_schema_innodb_metrics_buffer_buffer_pool_read_ahead_evicted_total 0
mysql_info_schema_innodb_metrics_buffer_buffer_pool_read_ahead_total 67668
mysql_info_schema_innodb_metrics_buffer_buffer_pool_read_requests_total 3.20166852455e+11
mysql_info_schema_innodb_metrics_buffer_buffer_pool_reads_total 895091
mysql_info_schema_innodb_metrics_buffer_buffer_pool_wait_free_total 0
mysql_info_schema_innodb_metrics_buffer_buffer_pool_write_requests_total 1.923061381e+09
mysql_info_schema_innodb_metrics_buffer_pool_dirty_pages 5913
mysql_info_schema_innodb_metrics_buffer_pool_pages
mysql_info_schema_innodb_metrics_change_buffer_ibuf_merges_delete_mark_total 3.023999e+06
mysql_info_schema_innodb_metrics_change_buffer_ibuf_merges_delete_total 143429
mysql_info_schema_innodb_metrics_change_buffer_ibuf_merges_discard_delete_mark_total 0
mysql_info_schema_innodb_metrics_change_buffer_ibuf_merges_discard_delete_total 0
mysql_info_schema_innodb_metrics_change_buffer_ibuf_merges_discard_insert_total 0
mysql_info_schema_innodb_metrics_change_buffer_ibuf_merges_insert_total 4.783398e+06
mysql_info_schema_innodb_metrics_change_buffer_ibuf_merges_total 745714
mysql_info_schema_innodb_metrics_change_buffer_ibuf_size_total 1
mysql_info_schema_innodb_metrics_dml_dml_deletes_total 8.5972214e+07
mysql_info_schema_innodb_metrics_dml_dml_inserts_total 2.04191046e+08
mysql_info_schema_innodb_metrics_dml_dml_updates_total 2.45199e+06
mysql_info_schema_innodb_metrics_file_system_file_num_open_files 240
mysql_info_schema_innodb_metrics_lock_lock_deadlocks_total 0
mysql_info_schema_innodb_metrics_lock_lock_row_lock_current_waits_total 0
mysql_info_schema_innodb_metrics_lock_lock_row_lock_time_avg 10947
mysql_info_schema_innodb_metrics_lock_lock_row_lock_time_max 51904
mysql_info_schema_innodb_metrics_lock_lock_row_lock_time_total 2.5475783e+07
mysql_info_schema_innodb_metrics_lock_lock_row_lock_waits_total 2327
mysql_info_schema_innodb_metrics_lock_lock_timeouts_total 433
mysql_info_schema_innodb_metrics_os_os_data_fsyncs_total 1.0957013e+07
mysql_info_schema_innodb_metrics_os_os_data_reads_total 1.679161e+06
mysql_info_schema_innodb_metrics_os_os_data_writes_total 4.3312276e+07
mysql_info_schema_innodb_metrics_os_os_log_bytes_written_total 6.6076711424e+10
mysql_info_schema_innodb_metrics_os_os_log_fsyncs_total 895814
mysql_info_schema_innodb_metrics_os_os_log_pending_fsyncs_total 0
mysql_info_schema_innodb_metrics_os_os_log_pending_writes_total 0
mysql_info_schema_innodb_metrics_recovery_log_padded_total 1.780294144e+09
mysql_info_schema_innodb_metrics_recovery_log_waits_total 0
mysql_info_schema_innodb_metrics_recovery_log_write_requests_total 1.55176313e+08
mysql_info_schema_innodb_metrics_recovery_log_writes_total 1.905225e+06
mysql_info_schema_innodb_metrics_server_buffer_pool_size 8.589934592e+09
mysql_info_schema_innodb_metrics_server_innodb_activity_count_total 1.1361926e+07
mysql_info_schema_innodb_metrics_server_innodb_dblwr_pages_written_total 3.7024178e+07
mysql_info_schema_innodb_metrics_server_innodb_dblwr_writes_total 3.588419e+06
mysql_info_schema_innodb_metrics_server_innodb_page_size 16384
mysql_info_schema_innodb_metrics_server_innodb_rwlock_s_os_waits_total 2.409811e+06
mysql_info_schema_innodb_metrics_server_innodb_rwlock_s_spin_rounds_total 8.5863931e+07
mysql_info_schema_innodb_metrics_server_innodb_rwlock_s_spin_waits_total 0
mysql_info_schema_innodb_metrics_server_innodb_rwlock_sx_os_waits_total 837950
mysql_info_schema_innodb_metrics_server_innodb_rwlock_sx_spin_rounds_total 7.927366e+07
mysql_info_schema_innodb_metrics_server_innodb_rwlock_sx_spin_waits_total 5.158882e+06
mysql_info_schema_innodb_metrics_server_innodb_rwlock_x_os_waits_total 2.397412e+06
mysql_info_schema_innodb_metrics_server_innodb_rwlock_x_spin_rounds_total 1.4835944e+08
mysql_info_schema_innodb_metrics_server_innodb_rwlock_x_spin_waits_total 0
mysql_info_schema_innodb_metrics_transaction_trx_rseg_history_len 8727
mysql_info_schema_innodb_tablespace_allocated_size_bytes
mysql_info_schema_innodb_tablespace_file_size_bytes
mysql_info_schema_innodb_tablespace_space_info
```

- 事件

```
--collect.perf_schema.eventsstatements
--collect.perf_schema.eventsstatementssum
--collect.perf_schema.eventswaits 
```

```
mysql_global_variables_performance_schema_events_statements_history_long_size 10000
mysql_global_variables_performance_schema_events_statements_history_size 10
mysql_perf_schema_events_statements_errors_total
mysql_perf_schema_events_statements_no_index_used_total
mysql_perf_schema_events_statements_rows_affected_total
mysql_perf_schema_events_statements_rows_examined_total
mysql_perf_schema_events_statements_rows_sent_total
mysql_perf_schema_events_statements_seconds_total
mysql_perf_schema_events_statements_sort_merge_passes_total
mysql_perf_schema_events_statements_sort_rows_total
mysql_perf_schema_events_statements_sum_created_tmp_disk_tables 86139
mysql_perf_schema_events_statements_sum_created_tmp_tables
mysql_perf_schema_events_statements_sum_errors 469
mysql_perf_schema_events_statements_sum_lock_time 1.5995296691e+16
mysql_perf_schema_events_statements_sum_no_good_index_used 0
mysql_perf_schema_events_statements_sum_no_index_used 7.777306e+06
mysql_perf_schema_events_statements_sum_rows_affected 2.25370482e+08
mysql_perf_schema_events_statements_sum_rows_examined 2.13377027868e+11
mysql_perf_schema_events_statements_sum_rows_sent 7.34432893e+08
mysql_perf_schema_events_statements_sum_select_full_join 571844
mysql_perf_schema_events_statements_sum_select_full_range_join 0
mysql_perf_schema_events_statements_sum_select_range 886935
mysql_perf_schema_events_statements_sum_select_range_check 0
mysql_perf_schema_events_statements_sum_select_scan 8.11732e+06
mysql_perf_schema_events_statements_sum_sort_merge_passes 45313
mysql_perf_schema_events_statements_sum_sort_range 722937
mysql_perf_schema_events_statements_sum_sort_rows 1.96587251e+08
mysql_perf_schema_events_statements_sum_sort_scan 988366
mysql_perf_schema_events_statements_sum_timer_wait 885586.6921017469
mysql_perf_schema_events_statements_sum_total 2.60999411e+08
mysql_perf_schema_events_statements_sum_warnings 4.75710422e+08
mysql_perf_schema_events_statements_tmp_disk_tables_total
mysql_perf_schema_events_statements_tmp_tables_total
mysql_perf_schema_events_statements_total
mysql_perf_schema_events_statements_warnings_total
```

- 表

```
 --collect.perf_schema.tableiowaits
```

```
mysql_perf_schema_table_io_waits_seconds_total
mysql_perf_schema_table_io_waits_total
```

- 索引

```
--collect.perf_schema.indexiowaits
```

```
索引 io 总共等待秒数
mysql_perf_schema_index_io_waits_seconds_total
索引 io 总数
mysql_perf_schema_index_io_waits_total
```

- 用户连接数
  主机连接情况

```
mysql_info_schema_processes_by_user 
mysql_info_schema_processes_by_host
```

- 表锁

```
 --collect.perf_schema.tablelocks
```

```
mysql perf schema sql lock 等待总数
mysql perf schema sql lock 总共等待秒数
mysql_perf_schema_external 外部锁等待总数
mysql_perf_schema_external 锁等待总秒数
mysql_perf_schema_sql_lock_waits_total
mysql_perf_schema_sql_lock_waits_seconds_total
mysql_perf_schema_external_lock_waits_seconds_total
mysql_perf_schema_external_lock_waits_total
```

### 9.2 node_exporter监控其他

下载node_exporter

```
wget https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-amd64.tar.gz
```

或者

```
wget https://github.91chifun.workers.dev/https://github.com//prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-amd64.tar.gz
```

示例1：

创建/usr/local/prom/mysql目录

```
mkdir -p /usr/local/prom/mysql
./node_exporter --path.rootfs=/host --collector.textfile.directory=/usr/local/prom/mysql
```

制造一个简单的数据，文件以.prom结尾

```
#!/bin/bash
if [ `ss -tlnp | egrep "32573|32574|32575" | wc -l` -ne 3 ]; then
   echo "Mysql_32573_32574_32575_is_runing 0" > /usr/local/prom/mysql/mysql.prom
else
   echo "Mysql_32573_32574_32575_is_runing 1" > /usr/local/prom/mysql/mysql.prom
fi
```

示例2：

/usr/local/node_exporter/

```
#!/bin/bash
echo  "http_metric_cannal_status_code `curl  -I  -m  10  -o  /dev/null  -s  -w  %{http_code}    127.0.0.1:8089`" > /usr/local/node_exporter/canal/run.prom
```

计划任务

```
*/1 * * * * /bin/bash /usr/local/node_exporter/runing.sh
```



```
# mkdir  /usr/local/node_exporter/canal -p
# cat /usr/local/node_exporter/canal/run.prom
http_metric_cannal_status_code 200
```

- start 

```
nohup ./node_exporter --path.rootfs=/host --collector.textfile.directory=/usr/local/node_exporter/canal &
```

```
[root@cz-redis-192 redis]# curl -s  127.0.0.1:9100/metrics | grep http_metric_cannal_status_code
# HELP http_metric_cannal_status_code Metric read from /usr/local/node_exporter/canal/run.prom
# TYPE http_metric_cannal_status_code untyped
http_metric_cannal_status_code 200
```



```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    prometheus: k8s
    role: alert-rules
  name: cannal-rules
  namespace: monitoring
spec:
  groups:
  - name: external_cannal_alarm
    rules:
    - alert: cannal_http_status_code_not200
      expr: http_metric_cannal_status_code != 200
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: cannal is Down (instance {{ $labels.instance }})
        description: "cannal value http_metric_cannal_status_code is not 200,服务已关闭\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
```



## 10.监控 rabbitmq

```
version: '3.8'
services:
  rabbitmq_exporter:
    container_name: rabbitmq_exporter
    image: kbudde/rabbitmq-exporter:v1.0.0-RC8
    network_mode: host
    environment:
    - RABBIT_USER=admin
    - RABBIT_PASSWORD=admin
    - PUBLISH_PORT=9419
    - LOG_LEVEL=info
    - RABBIT_TIMEOUT=30
    - SKIP_QUEUES="RPC_.*"
    - MAX_QUEUES=5000
    restart: always
    environment:
     - JAVA_OPTS=-Duser.timezone=Asia/Shanghai  # 时区1
    volumes:
     - /etc/localtime:/etc/localtime:ro  # 时区2
    logging:
      driver: "json-file"
      options:
        max-size: "50M"    
```



```
firewall-cmd --permanent --zone=public   --add-rich-rule='rule family="ipv4" source address="192.168.63.0/24" port port="9104" protocol="tcp" accept'
firewall-cmd --permanent --zone=public   --add-rich-rule='rule family="ipv4" source address="192.168.63.0/24" port port="9419" protocol="tcp" accept'
firewall-cmd --reload
```



```
apiVersion: v1
kind: Service
metadata:
  name: external-rabbitmq-k8s
  namespace: monitoring
  labels:
    k8s-app: external-rabbitmq
spec:
  type: ClusterIP
  clusterIP: None
  ports:
  - name: external-rabbitmq-port
    port: 9419
    protocol: TCP
---
apiVersion: v1
kind: Endpoints
metadata:
  name: external-rabbitmq-k8s
  namespace: monitoring
  labels:
    k8s-app: external-rabbitmq
subsets:
- addresses:
  - ip: 192.168.63.18
  ports:
  - name: external-rabbitmq-port
    port: 9419
    protocol: TCP
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: external-rabbitmq-k8s
  namespace: monitoring
  labels:
    k8s-app: external-rabbitmq
spec:
  jobLabel: k8s-app
  endpoints:
  - port: external-rabbitmq-port
    path: /metrics
    interval: 30s
    scheme: http
  selector:
    matchLabels:
      k8s-app: external-rabbitmq
  namespaceSelector:
    matchNames:
    - monitoring
```

`dashboard: https://grafana.com/grafana/dashboards/4279`

- 二进制
- config.json

```
{
    "rabbit_url": "http://127.0.0.1:15672",
    "rabbit_user": "admin",
    "rabbit_pass": "admin",
    "publish_port": "9419",
    "output_format": "TTY",
    "enabled_exporters": [
            "exchange",
            "node",
            "overview",
            "queue"
    ],
    "timeout": 30,
    "max_queues": 5000
}
```



```
nohup ./rabbitmq_exporter -config-file config.json > rabbitmq_exporter.log 2>&1 &
```



## 11.监控mongodb

mongodb配置

```
systemLog:
  destination: file
  logAppend: true
  path: /u01/data/mongodb/logs/mongod.log

storage:
  dbPath: /u01/data/mongodb/data
  journal:
    enabled: true
  directoryPerDB: true
  wiredTiger:
     engineConfig:
        cacheSizeGB: 8
        directoryForIndexes: true
processManagement:
  fork: true
  pidFilePath: /u01/data/mongodb/pid/mongod.pid

net:
  port: 27020
  bindIp: 192.168.63.10,localhost,mongodb1
  maxIncomingConnections: 5000

#security:
  #keyFile: /data/mongodb/conf/keyfile
  #authorization: enabled
replication:
#   oplogSizeMB: 1024
   replSetName: rs02
```

mongodb需要配置授权

```
db.grantRolesToUser("root", [{role:"__system", db:"admin"}])
db.grantRolesToUser("root", [{role:"dbAdminAnyDatabase", db:"admin"}]);
```

- docker-compose.yaml

```
version: '3.8'
services:
  mongodb_exporter:
    container_name: mongodb_exporter
    image: lovullo/percona-mongodb-exporter:v0.11.2
    network_mode: host
    command:
      - --mongodb.uri=mongodb://root:xyt#123.com@127.0.0.1:27020/admin?ssl=false
      - --web.listen-address=0.0.0.0:9216
      - --web.telemetry-path=/metrics
      - --log.level=error
    restart: always
    environment:
     - JAVA_OPTS=-Duser.timezone=Asia/Shanghai  # 时区1
    volumes:
     - /etc/localtime:/etc/localtime:ro  # 时区2
    logging:
      driver: "json-file"
      options:
        max-size: "50M"         
```



```
apiVersion: v1
kind: Service
metadata:
  name: external-mongodb-k8s
  namespace: monitoring
  labels:
    k8s-app: external-mongodb
spec:
  type: ClusterIP
  clusterIP: None
  ports:
  - name: external-mongodb-port
    port: 9216
    protocol: TCP
---
apiVersion: v1
kind: Endpoints
metadata:
  name: external-mongodb-k8s
  namespace: monitoring
  labels:
    k8s-app: external-mongodb
subsets:
- addresses:
  - ip: 192.168.63.10
  - ip: 192.168.63.11
  - ip: 192.168.63.12
  ports:
  - name: external-mongodb-port
    port: 9216
    protocol: TCP
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: external-mongodb-k8s
  namespace: monitoring
  labels:
    k8s-app: external-mongodb
spec:
  jobLabel: k8s-app
  endpoints:
  - port: external-mongodb-port
    path: /metrics
    interval: 30s
    scheme: http
  selector:
    matchLabels:
      k8s-app: external-mongodb
  namespaceSelector:
    matchNames:
    - monitoring
```

`dashboard: https://grafana.com/grafana/dashboards/2583`

## 12.监控kafka



```
version: '3.8'
services:
  kafka_exporter:
    container_name: kafka_exporter
    image: danielqsj/kafka-exporter:v1.3.1
    network_mode: host
    command:
      - --kafka.server=10.110.112.26:9092
      - --sasl.enabled
      - --sasl.username=admin
      - --sasl.password=YxthnSiu!302#*
      - --sasl.mechanism=scram-sha256
      - --web.listen-address=0.0.0.0:9308
      - --log.level=debug
    restart: always
    environment:
     - JAVA_OPTS=-Duser.timezone=Asia/Shanghai  # 时区1
    volumes:
     - /etc/localtime:/etc/localtime:ro  # 时区2
    logging:
      driver: "json-file"
      options:
        max-size: "50M"         
```



```
apiVersion: v1
kind: Service
metadata:
  name: external-kafka-k8s
  namespace: monitoring
  labels:
    k8s-app: external-kafka
spec:
  type: ClusterIP
  clusterIP: None
  ports:
  - name: external-kafka-port
    port: 9308
    protocol: TCP
---
apiVersion: v1
kind: Endpoints
metadata:
  name: external-kafka-k8s
  namespace: monitoring
  labels:
    k8s-app: external-kafka
subsets:
- addresses:
  - ip: 192.168.63.7
  - ip: 192.168.63.8
  - ip: 192.168.63.9
  ports:
  - name: external-kafka-port
    port: 9308
    protocol: TCP
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: external-kafka-k8s
  namespace: monitoring
  labels:
    k8s-app: external-kafka
spec:
  jobLabel: k8s-app
  endpoints:
  - port: external-kafka-port
    path: /metrics
    interval: 30s
    scheme: http
    selector:
    matchLabels:
      k8s-app: external-kafka
    namespaceSelector:
    matchNames:
    - monitoring
```

```
dashboard:
https://grafana.com/grafana/dashboards/7589
https://grafana.com/grafana/dashboards/14281
```

| 名称                                 | 暴露信息                            |
| ------------------------------------ | ----------------------------------- |
| `kafka_consumergroup_current_offset` | 消费者组在主题/分区的当前偏移量     |
| `kafka_consumergroup_lag`            | 当前消费者组在主题/分区上的近似滞后 |

## 13.监控windows

`download : https://github.com/prometheus-community/windows_exporter/releases`

```
apiVersion: v1
kind: Service
metadata:
  name: external-windows-node-k8s
  namespace: monitoring
  labels:
    k8s-app: external-windows-node
spec:
  type: ClusterIP
  clusterIP: None
  ports:
  - name: external-node-port
    port: 9182
    protocol: TCP
---
apiVersion: v1
kind: Endpoints
metadata:
  name:  external-windows-node-k8s
  namespace: monitoring
  labels:
    k8s-app: external-windows-node
subsets:
- addresses:
  - ip: 192.168.63.19
  ports:
  - name: external-node-port
    port: 9182
    protocol: TCP
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name:  external-windows-node-k8s
  namespace: monitoring
  labels:
    k8s-app: external-windows-node
spec:
  jobLabel: k8s-app
  endpoints:
  - port: external-node-port
    path: /metrics
    interval: 30s
    scheme: http
    selector:
    matchLabels:
      k8s-app: external-windows-node
    namespaceSelector:
    matchNames:
    - monitoring
```

`dashboard: https://grafana.com/grafana/dashboards/10467`

## 14.监控nginx

开启

```
server {
        listen       40080;
        server_name  localhost;
        location /nginx_status
        {
                stub_status on;
                access_log off;
        }
        location /php-fpm_status
        {
                fastcgi_pass 127.0.0.1:9000;
                fastcgi_param SCRIPT_FILENAME $fastcgi_script_name;
                include fastcgi_params;
        }
}
```



```
version: '3.8'
services:
  nginx_prometheus_exporter:
    container_name: nginx_prometheus_exporter
    image: nginx/nginx-prometheus-exporter:0.9.0
    network_mode: host
    command:
      - -nginx.scrape-uri=http://127.0.0.1:40080/nginx_status
    restart: always
    environment:
     - JAVA_OPTS=-Duser.timezone=Asia/Shanghai  # 时区1
    volumes:
     - /etc/localtime:/etc/localtime:ro  # 时区2
    logging:
      driver: "json-file"
      options:
        max-size: "50M"         
```

`dashboard: https://grafana.com/grafana/dashboards/12708`

yaml

```
apiVersion: v1
kind: Service
metadata:
  name: external-nginx
  namespace: monitoring
  labels:
    k8s-app: external-nginx
spec:
  type: ClusterIP
  clusterIP: None
  ports:
  - name: external-nginx
    port: 9113
    protocol: TCP
---
apiVersion: v1
kind: Endpoints
metadata:
  name:  external-nginx
  namespace: monitoring
  labels:
    k8s-app: external-nginx
subsets:
- addresses:
  - ip: 192.168.63.14
  ports:
  - name: external-nginx
    port: 9113
    protocol: TCP
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name:  external-nginx
  namespace: monitoring
  labels:
    k8s-app: external-nginx
spec:
  jobLabel: k8s-app
  endpoints:
  - port: external-nginx
    path: /metrics
    interval: 30s
    scheme: http
    selector:
    matchLabels:
      k8s-app: external-nginx
    namespaceSelector:
    matchNames:
    - monitoring
```



## 15.监控NFS

NFS集成在node-exporter中

`dashboard: https://grafana.com/grafana/dashboards/14614`

## 16.jvm

https://micrometer.io/

## 16.1 redis

docker

https://hub.docker.com/r/oliver006/redis_exporter/tags?page=1&ordering=last_updated

```
docker pull oliver006/redis_exporter:v1.26.0-alpine
docker run -d --name redis_exporter -p 9121:9121 oliver006/redis_exporter:v1.26.0-alpine
```

二进制

https://github.com/oliver006/redis_exporter/releases

https://github.com/oliver006/redis_exporter/releases/download/v1.27.0/redis_exporter-v1.27.0.linux-amd64.tar.gz

```
wget https://github.com/oliver006/redis_exporter/releases/download/v1.27.0/redis_exporter-v1.27.0.linux-amd64.tar.gz
tar xf redis_exporter-v1.27.0.linux-amd64.tar.gz
mv redis_exporter-v1.27.0.linux-amd64 /usr/local/redis_exporter-v1.27.0.linux-amd64
ln -s /usr/local/redis_exporter-v1.27.0.linux-amd64 /usr/local/redis_exporter
nohup /usr/local/redis_exporter/redis_exporter -redis.password r2nKSaQ4nixE77E3  start.file 2>&1 &
```

```
curl 127.0.0.1:9121/metrics
```

```
firewall-cmd --permanent --zone=public   --add-rich-rule='rule family="ipv4" source address="192.168.201.0/24" port port="9121" protocol="tcp" accept'

firewall-cmd --reload
```



yaml

```
apiVersion: v1
kind: Service
metadata:
  name: external-redis
  namespace: monitoring
  labels:
    k8s-app: external-redis
spec:
  type: ClusterIP
  clusterIP: None
  ports:
  - name: external-redis
    port: 9121
    protocol: TCP
---
apiVersion: v1
kind: Endpoints
metadata:
  name:  external-redis
  namespace: monitoring
  labels:
    k8s-app: external-redis
subsets:
- addresses:
  - ip: 192.168.63.13
  ports:
  - name: external-redis
    port: 9121
    protocol: TCP
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name:  external-redis
  namespace: monitoring
  labels:
    k8s-app: external-redis
spec:
  jobLabel: k8s-app
  endpoints:
  - port: external-redis
    path: /metrics
    interval: 30s
    scheme: http
    selector:
    matchLabels:
      k8s-app: external-redis
    namespaceSelector:
    matchNames:
    - monitoring
```

模板：https://github.com/oliver006/redis_exporter/blob/master/contrib/grafana_prometheus_redis_dashboard.json

## 16.2添加磁盘警报项

https://awesome-prometheus-alerts.grep.to/

disk.yaml

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    prometheus: k8s
    role: alert-rules
  name: disk-free-rules
  namespace: monitoring
spec:
  groups:
  - name: disk
    rules:
    - alert: diskFree
      labels:
        severity: warning
      annotations:
        summary: "{{ $labels.job }}  项目实例 {{ $labels.instance }} 磁盘使用率大于 80%"
        description: "{{ $labels.instance }}  {{ $labels.mountpoint }}  磁盘使用率大于80%  (当前的值: {{ $value }}%),请及时处理"
      expr: |
        (1-(node_filesystem_free_bytes{fstype=~"ext4|xfs",mountpoint!="/boot"} / node_filesystem_size_bytes{fstype=~"ext4|xfs",mountpoint!="/boot"}) )*100 > 85
      for: 3m
```



## 16.dingtalk1

部署即可

- alertmanager-secret.yaml

```
apiVersion: v1
data: {}
kind: Secret
metadata:
  name: alertmanager-main
  namespace: monitoring
stringData:
  alertmanager.yaml: |-
    global:
      resolve_timeout: 5m
    route: 
      group_by: ['alertname'] //根据label标签进行匹配，如果是多个，就要多个都匹配
      group_wait: 10s   //等待匹配的组报警，10秒内有报警即可发送人，如果有多个都发
      group_interval: 10s //下一次报警的时间
      repeat_interval: 5m  // 为解决的警报，重复报警时间      
      receiver: 'webhook'
      routes:  //条件匹配，如果报警级别为critical，匹配(^(warning|critical)$)则发现有些给pager组
      - match: 
        severity:  critical
        receiver: webhook
    receivers:
    - name: 'webhook'
      webhook_configs:
      - send_resolved: true
        url: 'http://webhook-dingtalk:8060/dingtalk/webhook/send'
    inhibit_rules:
      - source_match:
          severity: 'critical'
        target_match:
          severity: 'warning'
        equal: ['alertname', 'dev', 'instance']
```

- dingtalk-configmap.yaml

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: dingtalk-config
  namespace: monitoring
  labels:
    app: dingtalk-config
data:
  config.yml: |
    templates:
      - /etc/prometheus-webhook-dingtalk/templates/default.tmpl
    targets:
      webhook:
        url: https://oapi.dingtalk.com/robot/send?access_token=cdb647c5c1200899d5a7d1349ec0d798547e
        secret: SECb0039317816c114b69dc4b3ad995093f0
        message:
          title: '{{ template "ding.link.title" . }}'
          text: '{{ template "ding.link.content" . }}'
```

- dingtalk-template.yaml

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: dingtalk-template
  namespace: monitoring
  labels:
    app: dingtalk-template
data:
  default.tmpl: |
    {{ define "__subject" }}[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}] {{ .GroupLabels.SortedPairs.Values | join " " }} {{ if gt (len .CommonLabels) (len .GroupLabels) }}({{ with .CommonLabels.Remove .GroupLabels.Names }}{{ .Values | join " " }}{{ end }}){{ end }}{{ end }}
    {{ define "__alertmanagerURL" }}{{ .ExternalURL }}/#/alerts?receiver={{ .Receiver }}{{ end }}

    {{ define "__text_alert_list" }}{{ range . }}
    **Labels**
    {{ range .Labels.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}
    {{ end }}
    **Annotations**
    {{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}
    {{ end }}
    **Source:** [{{ .GeneratorURL }}]({{ .GeneratorURL }})
    {{ end }}{{ end }}
    
    {{ define "default.__text_alert_list" }}{{ range . }}
    ---
    **alert level:** {{ .Labels.severity | upper }}
    
    **trigger time:** {{ dateInZone "2006.01.02 15:04:05" (.StartsAt) "Asia/Shanghai" }}
    
    **event info:**
    {{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}


    {{ end }}
    
    **event label:**
    {{ range .Labels.SortedPairs }}{{ if and (ne (.Name) "severity") (ne (.Name) "summary") (ne (.Name) "team") }}> - {{ .Name }}: {{ .Value | markdown | html }}
    {{ end }}{{ end }}
    {{ end }}
    {{ end }}
    {{ define "default.__text_alertresovle_list" }}{{ range . }}
    ---
    **alert level:** {{ .Labels.severity | upper }}
    
    **trigger time:** {{ dateInZone "2006.01.02 15:04:05" (.StartsAt) "Asia/Shanghai" }}
    
    **end time:** {{ dateInZone "2006.01.02 15:04:05" (.EndsAt) "Asia/Shanghai" }}
    
    **event info:**
    {{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}


    {{ end }}
    
    **event label:**
    {{ range .Labels.SortedPairs }}{{ if and (ne (.Name) "severity") (ne (.Name) "summary") (ne (.Name) "team") }}> - {{ .Name }}: {{ .Value | markdown | html }}
    {{ end }}{{ end }}
    {{ end }}
    {{ end }}
    
    {{/* Default */}}
    {{ define "default.title" }}{{ template "__subject" . }}{{ end }}
    {{ define "default.content" }}#### \[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}\] **[{{ index .GroupLabels "alertname" }}]({{ template "__alertmanagerURL" . }})**
    {{ if gt (len .Alerts.Firing) 0 -}}
    
    ***====⚠️⚠️⚠️trigger alarm====***
    {{ template "default.__text_alert_list" .Alerts.Firing }}


    {{- end }}
    
    {{ if gt (len .Alerts.Resolved) 0 -}}
    **====[烟花]recover alarm====**
    {{ template "default.__text_alertresovle_list" .Alerts.Resolved }}


    {{- end }}
    {{- end }}
    
    {{/* Legacy */}}
    {{ define "legacy.title" }}{{ template "__subject" . }}{{ end }}
    {{ define "legacy.content" }}#### \[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}\] **[{{ index .GroupLabels "alertname" }}]({{ template "__alertmanagerURL" . }})**
    {{ template "__text_alert_list" .Alerts.Firing }}
    {{- end }}
    
    {{/* Following names for compatibility */}}
    {{ define "ding.link.title" }}{{ template "default.title" . }}{{ end }}
    {{ define "ding.link.content" }}{{ template "default.content" . }}{{ end }}
```

告警模板2

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: dingtalk-template
  namespace: monitoring
  labels:
    app: dingtalk-template
data:
  default.tmpl: |
    {{ define "__subject" }}[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}] {{ .GroupLabels.SortedPairs.Values | join " " }} {{ if gt (len .CommonLabels) (len .GroupLabels) }}({{ with .CommonLabels.Remove .GroupLabels.Names }}{{ .Values | join " " }}{{ end }}){{ end }}{{ end }}
    {{ define "__alertmanagerURL" }}{{ .ExternalURL }}/#/alerts?receiver={{ .Receiver }}{{ end }}

    {{ define "__text_alert_list" }}{{ range . }}
    **Labels**
    {{ range .Labels.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}
    {{ end }}
    **Annotations**
    {{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}
    {{ end }}
    **Source:** [{{ .GeneratorURL }}]({{ .GeneratorURL }})
    {{ end }}{{ end }}
    
    {{ define "default.__text_alert_list" }}{{ range . }}
    ---
    **alert level:** {{ .Labels.severity | upper }}
    
    **trigger time:** {{ dateInZone "2006.01.02 15:04:05" (.StartsAt) "Asia/Shanghai" }}
    
    **event info:**
    {{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}


    {{ end }}
    
    **event label:**
    {{ range .Labels.SortedPairs }}{{ if and (ne (.Name) "severity") (ne (.Name) "summary") (ne (.Name) "team") }}> - {{ .Name }}: {{ .Value | markdown | html }}
    {{ end }}{{ end }}
    {{ end }}
    {{ end }}
    {{ define "default.__text_alertresovle_list" }}{{ range . }}
    ---
    **alert level:** {{ .Labels.severity | upper }}
    
    **trigger time:** {{ dateInZone "2006.01.02 15:04:05" (.StartsAt) "Asia/Shanghai" }}
    
    **end time:** {{ dateInZone "2006.01.02 15:04:05" (.EndsAt) "Asia/Shanghai" }}
    
    **event info:**
    {{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}


    {{ end }}
    
    **event label:**
    {{ range .Labels.SortedPairs }}{{ if and (ne (.Name) "severity") (ne (.Name) "summary") (ne (.Name) "team") }}> - {{ .Name }}: {{ .Value | markdown | html }}
    {{ end }}{{ end }}
    {{ end }}
    {{ end }}
    
    {{/* Default */}}
    {{ define "default.title" }}{{ template "__subject" . }}{{ end }}
    {{ define "default.content" }}#### \[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}\] **[{{ index .GroupLabels "alertname" }}]({{ template "__alertmanagerURL" . }})**
    {{ if gt (len .Alerts.Firing) 0 -}}
    
    **====⚠️⚠️⚠️trigger alarm====**
    {{ template "default.__text_alert_list" .Alerts.Firing }}


    {{- end }}
    
    {{ if gt (len .Alerts.Resolved) 0 -}}
    **====[烟花]recover alarm====**
    {{ template "default.__text_alertresovle_list" .Alerts.Resolved }}


    {{- end }}
    {{- end }}
    
    {{/* Legacy */}}
    {{ define "legacy.title" }}{{ template "__subject" . }}{{ end }}
    {{ define "legacy.content" }}#### \[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}\] **[{{ index .GroupLabels "alertname" }}]({{ template "__alertmanagerURL" . }})**
    {{ template "__text_alert_list" .Alerts.Firing }}
    {{- end }}
    
    {{/* Following names for compatibility */}}
    {{ define "ding.link.title" }}{{ template "default.title" . }}{{ end }}
    {{ define "ding.link.content" }}{{ template "default.content" . }}{{ end }}
```

模板2告警截图

```
[FIRING:1] KubeMemOvercommit
====⚠️⚠️⚠️trigger alarm====

alert level: WARNING

trigger time: 2021.07.15 09:10:17

event info:

message: Cluster has overcommitted memory resource requests for Pods and cannot tolerate node failure.
runbook_url: https://github.com/kubernetes-monitoring/kubernetes-mixin/tree/master/runbook.md#alert-name-kubememovercommit
event label:

alertname: KubeMemOvercommit
prometheus: monitoring/k8s
```

告警模板3。当前使用

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: dingtalk-template
  namespace: monitoring
  labels:
    app: dingtalk-template
data:
  default.tmpl: |
    {{ define "__subject" }}[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}] {{ .GroupLabels.SortedPairs.Values | join " " }} {{ if gt (len .CommonLabels) (len .GroupLabels) }}({{ with .CommonLabels.Remove .GroupLabels.Names }}{{ .Values | join " " }}{{ end }}){{ end }}{{ end }}
    {{ define "__alertmanagerURL" }}{{ .ExternalURL }}/#/alerts?receiver={{ .Receiver }}{{ end }}

    {{ define "__text_alert_list" }}{{ range . }}
    **Labels**
    {{ range .Labels.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}
    {{ end }}
    **Annotations**
    {{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}
    {{ end }}
    **Source:** [{{ .GeneratorURL }}]({{ .GeneratorURL }})
    {{ end }}{{ end }}
    
    {{ define "default.__text_alert_list" }}{{ range . }}
    ---
    **告警级别:** {{ .Labels.severity | upper }}
    
    **触发时间:** {{ dateInZone "2006.01.02 15:04:05" (.StartsAt) "Asia/Shanghai" }}
    
    **事件信息:**
    {{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}


    {{ end }}
    
    **事件标签:**
    {{ range .Labels.SortedPairs }}{{ if and (ne (.Name) "severity") (ne (.Name) "summary") (ne (.Name) "team") }}> - {{ .Name }}: {{ .Value | markdown | html }}
    {{ end }}{{ end }}
    {{ end }}
    {{ end }}
    {{ define "default.__text_alertresovle_list" }}{{ range . }}
    ---
    **告警级别:** {{ .Labels.severity | upper }}
    
    **触发时间:** {{ dateInZone "2006.01.02 15:04:05" (.StartsAt) "Asia/Shanghai" }}
    
    **结束时间:** {{ dateInZone "2006.01.02 15:04:05" (.EndsAt) "Asia/Shanghai" }}
    
    **事件信息:**
    {{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}


    {{ end }}
    
    **事件标签:**
    {{ range .Labels.SortedPairs }}{{ if and (ne (.Name) "severity") (ne (.Name) "summary") (ne (.Name) "team") }}> - {{ .Name }}: {{ .Value | markdown | html }}
    {{ end }}{{ end }}
    {{ end }}
    {{ end }}
    
    {{/* Default */}}
    {{ define "default.title" }}{{ template "__subject" . }}{{ end }}
    {{ define "default.content" }}#### \[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}\] **[{{ index .GroupLabels "alertname" }}]({{ template "__alertmanagerURL" . }})**
    {{ if gt (len .Alerts.Firing) 0 -}}
    
    ![20210716143611.png](https://pic5.58cdn.com.cn/nowater/webim/big/n_v27ab5f3b496a2419ba8ae8b3cf27960f0.png)
    
    **====⚠️⚠️⚠️trigger alarm====**
    
    {{ template "default.__text_alert_list" .Alerts.Firing }}


    {{- end }}
    
    {{ if gt (len .Alerts.Resolved) 0 -}}
    **====[烟花]recover alarm====**
    {{ template "default.__text_alertresovle_list" .Alerts.Resolved }}


    {{- end }}
    {{- end }}
    
    {{/* Legacy */}}
    {{ define "legacy.title" }}{{ template "__subject" . }}{{ end }}
    {{ define "legacy.content" }}#### \[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}\] **[{{ index .GroupLabels "alertname" }}]({{ template "__alertmanagerURL" . }})**
    {{ template "__text_alert_list" .Alerts.Firing }}
    {{- end }}
    
    {{/* Following names for compatibility */}}
    {{ define "ding.link.title" }}{{ template "default.title" . }}{{ end }}
    {{ define "ding.link.content" }}{{ template "default.content" . }}{{ end }}
```



- webhook-dingtalk.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: webhook-dingtalk
  name: webhook-dingtalk
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webhook-dingtalk
  template:
    metadata:
      labels:
        app: webhook-dingtalk
    spec:
      containers:
      - image: marksugar/k8s-prometheus:prometheus_dingtalk_v0.2
        name: webhook-dingtalk
        args:
        - --web.listen-address=:8060
        - --config.file=/etc/prometheus-webhook-dingtalk/config.yml
        volumeMounts:
        - name: webdingtalk-configmap
          mountPath: /etc/prometheus-webhook-dingtalk/
        - name: webdingtalk-template
          mountPath: /etc/prometheus-webhook-dingtalk/templates/
        ports:
        - containerPort: 8060
          protocol: TCP
      imagePullSecrets:
        - name: IfNotPresent
      volumes:
        - name: webdingtalk-configmap
          configMap:
            name: dingtalk-config
        - name: webdingtalk-template
          configMap:
            name: dingtalk-template
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: webhook-dingtalk
  name: webhook-dingtalk
  namespace: monitoring
spec:
  ports:
  - name: http
    port: 8060
    protocol: TCP
    targetPort: 8060
    selector:
    app: webhook-dingtalk
    type: ClusterIP
```

- 启动

```
kubectl apply -f dingtalk-configmap.yamll
kubectl apply -f  dingtalk-template.yaml
kubectl apply -f webhook-dingtalk.yaml
kubectl apply -f alertmanager-secret.yaml
```



## 17.dingtalk2

可部署dingtalk1

参考项目：https://github.com/timonwong/prometheus-webhook-dingtalk 其中的k8s部分： https://github.com/timonwong/prometheus-webhook-dingtalk/tree/master/contrib/k8s

- deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: alertmanager-webhook-dingtalk
spec:
  template:
    spec:
      volumes:
        - name: config
          configMap:
            name: alertmanager-webhook-dingtalk
      containers:
        - name: alertmanager-webhook-dingtalk
          image: timonwong/prometheus-webhook-dingtalk
          args:
            - --web.listen-address=:8060
            - --config.file=/config/config.yaml
          volumeMounts:
            - name: config
              mountPath: /config
          resources:
            limits:
              cpu: 100m
              memory: 100Mi
          ports:
            - name: http
              containerPort: 8060
```

- kustomization.yaml

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml

namespace: monitoring

commonLabels:
  app: alertmanager-webhook-dingtalk

replicas:
  - name: alertmanager-webhook-dingtalk
    count: 1

images:
  - name: timonwong/prometheus-webhook-dingtalk
    newTag: v1.4.0

configMapGenerator:
  - name: alertmanager-webhook-dingtalk
    files:
      - config/config.yaml
      - config/template.tmpl
```

- service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: alertmanager-webhook-dingtalk
spec:
  ports:
  - name: http
    port: 8060
    protocol: TCP
    targetPort: 8060
```

1.yaml

```
apiVersion: v1
data: {}
kind: Secret
metadata:
  name: alertmanager-main
  namespace: monitoring
stringData:
  alertmanager.yaml: |-
    global:
      resolve_timeout: 5m
    route:
      group_by: ['alertname']
      group_wait: 10s
      group_interval: 10s
      repeat_interval: 5m
      receiver: 'webhook'
    receivers:
    - name: 'webhook'
      webhook_configs:
      - send_resolved: true
        url: 'http://alertmanager-webhook-dingtalk:8060/dingtalk/webhook1/send'
    inhibit_rules:
      - source_match:
          severity: 'critical'
        target_match:
          severity: 'warning'
        equal: ['alertname', 'dev', 'instance']
```

### 17.0 config

config下有两个文件

- config.yaml

```
##
# This config is for prometheus-webhook-dingtalk instead of Kubernetes!
##

## Request timeout
# timeout: 5s

## Customizable templates path
templates:
  - /config/template.tmpl

## You can also override default template using `default_message`
## The following example to use the 'legacy' template from v0.3.0
# default_message:
#   title: '{{ template "legacy.title" . }}'
#   text: '{{ template "legacy.content" . }}'
targets:
  webhook1:
    url: https://oapi.dingtalk.com/robot/send?access_token=6018ed67a539dd5e9409cda7cd7076dbb6578beab78d5
    # secret for signature
    secret: SECcecf593013365d46ff419324cc8c2da1d66e2f49
```

- template.tmpl

```
{{ define "__subject" }}[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}] {{ .GroupLabels.SortedPairs.Values | join " " }} {{ if gt (len .CommonLabels) (len .GroupLabels) }}({{ with .CommonLabels.Remove .GroupLabels.Names }}{{ .Values | join " " }}{{ end }}){{ end }}{{ end }}
{{ define "__alertmanagerURL" }}{{ .ExternalURL }}/#/alerts?receiver={{ .Receiver }}{{ end }}

{{ define "__text_alert_list" }}{{ range . }}
**Labels**
{{ range .Labels.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}
{{ end }}
**Annotations**
{{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}
{{ end }}
**Source:** [{{ .GeneratorURL }}]({{ .GeneratorURL }})
{{ end }}{{ end }}

{{ define "default.__text_alert_list" }}{{ range . }}
#### \[{{ .Labels.severity | upper }}\] {{ .Annotations.summary }}

**Description:** {{ .Annotations.description }}

**Graph:** [📈]({{ .GeneratorURL }})

**Details:**
{{ range .Labels.SortedPairs }}{{ if and (ne (.Name) "severity") (ne (.Name) "summary") }}> - {{ .Name }}: {{ .Value | markdown | html }}
{{ end }}{{ end }}
{{ end }}{{ end }}

{{/* Default */}}
{{ define "default.title" }}{{ template "__subject" . }}{{ end }}
{{ define "default.content" }}#### \[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}\] **[{{ index .GroupLabels "alertname" }}]({{ template "__alertmanagerURL" . }})**
{{ if gt (len .Alerts.Firing) 0 -}}
**Alerts Firing**
{{ template "default.__text_alert_list" .Alerts.Firing }}
{{- end }}
{{ if gt (len .Alerts.Resolved) 0 -}}
**Alerts Resolved**
{{ template "default.__text_alert_list" .Alerts.Resolved }}
{{- end }}
{{- end }}

{{/* Legacy */}}
{{ define "legacy.title" }}{{ template "__subject" . }}{{ end }}
{{ define "legacy.content" }}#### \[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}\] **[{{ index .GroupLabels "alertname" }}]({{ template "__alertmanagerURL". }})**
{{ template "__text_alert_list" .Alerts.Firing }}
{{- end }}

{{/* Following names for compatibility */}}
{{ define "ding.link.title" }}{{ template "default.title" . }}{{ end }}
{{ define "ding.link.content" }}{{ template "default.content" . }}{{ end }}
```

目录结构如下

```
[root@yxtcg13 prometheus-webhook-dingtalk]# tree ./
./
├── 1.yaml
├── config
│   ├── config.yaml
│   └── template.tmpl
├── deployment.yaml
├── kustomization.yaml
└── service.yaml

1 directory, 6 files
```

启动

```
[root@yxtcg13 prometheus-webhook-dingtalk]# kustomize build ./|kubectl apply -f -
configmap/alertmanager-webhook-dingtalk-fk8h4d5kkf unchanged
service/alertmanager-webhook-dingtalk unchanged
deployment.apps/alertmanager-webhook-dingtalk unchanged
```

### 17.0.1 关闭

```
...
  #暂时关闭此警报，直到上游修复                         
  - match :                                                                               
       alertname : etcdHighNumberOfFailedGRPCRequests                                     
    receiver : " null
```



### 17.1 集群外主机rule

https://docs.signalfx.com/en/latest/integrations/agent/monitors/collectd-cpufreq.html

https://awesome-prometheus-alerts.grep.to/rules.html

https://awesome-prometheus-alerts.grep.to/rules#etcd

#### linux

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    prometheus: k8s
    role: alert-rules
  name: node-rules
  namespace: monitoring
spec:
  groups:
    - name: external_node_alarm
      rules:
      - alert: node_host_lost
        expr: up{job="external-node"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "{{$labels.instance}}:service is down"
          description: "{{$labels.instance}}: lost contact for 1 minutes"
      - alert: HostOutOfMemory
        expr: node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100 < 5
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: (instance {{ $labels.instance }})实例内存不足
          description: "节点内存已用完 (剩余小于5%)已持续2分钟\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      - alert: HostOutOfMemory
        expr: node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100 < 2
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: (instance {{ $labels.instance }})实例内存不足
          description: "节点内存已用完 (剩余小于2%)已持续2分钟\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"          
      - alert: HostMemoryUnderMemoryPressure
        expr: rate(node_vmstat_pgmajfault[1m]) > 1000
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: (instance {{ $labels.instance }})实例内存压力很大
          description: "节点内存压力很大major page错误率高已持续2分钟\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      - alert: HostUnusualNetworkThroughputIn
        expr: sum by (instance) (rate(node_network_receive_bytes_total[2m])) / 1024 / 1024 > 100
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: (instance {{ $labels.instance }})实例出现异常的input网络吞吐量
          description: "{{ $labels.instance }}网络接口接收超过100MB/s的数据已持续5分钟 \n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"          
      - alert: HostUnusualNetworkThroughputOut
        expr: sum by (instance) (rate(node_network_transmit_bytes_total[2m])) / 1024 / 1024 > 100
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: (instance {{ $labels.instance }})实例出现异常的output网络吞吐量
          description: "{{ $labels.instance }}实例网络发送了超过100MB/s的数据已持续5分钟 (> 100 MB/s)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      - alert: HostUnusualDiskReadRate
        expr: sum by (instance) (rate(node_disk_read_bytes_total[2m])) / 1024 / 1024 > 100
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: (instance {{ $labels.instance }})实例磁盘读取临近峰值
          description: "{{ $labels.instance }}实例磁盘读取超过>100MB/s已持续5分钟 \n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      - alert: HostUnusualDiskWriteRate
        expr: sum by (instance) (rate(node_disk_written_bytes_total[2m])) / 1024 / 1024 > 60
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: (instance {{ $labels.instance }})实例磁盘写入临近峰值
          description: "{{ $labels.instance }}实例磁盘写入超过60MB/s已持续5分钟 \n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      # Please add ignored mountpoints in node_exporter parameters like
      # "--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|run)($|/)".
      # Same rule using "node_filesystem_free_bytes" will fire when disk fills for non-root users.
      - alert: HostOutOfDiskSpace
        expr: (node_filesystem_avail_bytes * 100) / node_filesystem_size_bytes < 10 and ON (instance, device, mountpoint) node_filesystem_readonly == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: (instance {{ $labels.instance }})实例磁盘空间不足
          description: "{{ $labels.instance }}磁盘快满了,剩余不足10%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      # Please add ignored mountpoints in node_exporter parameters like
      # "--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|run)($|/)".
      # Same rule using "node_filesystem_free_bytes" will fire when disk fills for non-root users.
      - alert: HostOutOfDiskSpace
        expr: (node_filesystem_avail_bytes * 100) / node_filesystem_size_bytes < 5 and ON (instance, device, mountpoint) node_filesystem_readonly == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: (instance {{ $labels.instance }})实例磁盘空间不足
          description: "{{ $labels.instance }}磁盘快满了,剩余不足5%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"      
      - alert: HostDiskWillFillIn24Hours
        expr: (node_filesystem_avail_bytes * 100) / node_filesystem_size_bytes < 10 and ON (instance, device, mountpoint) predict_linear(node_filesystem_avail_bytes{fstype!~"tmpfs"}[1h], 24 * 3600) < 0 and ON (instance, device, mountpoint) node_filesystem_readonly == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: (instance {{ $labels.instance }})在24小时内就会写满
          description: "{{ $labels.instance }}在当前的写入速率下,预计文件系统将会在24小时内耗尽磁盘空间\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      - alert: HostOutOfInodes
        expr: node_filesystem_files_free{mountpoint ="/rootfs"} / node_filesystem_files{mountpoint="/rootfs"} * 100 < 10 and ON (instance, device, mountpoint) node_filesystem_readonly{mountpoint="/rootfs"} == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: (instance {{ $labels.instance }})实例inodes不足
          description: "{{ $labels.instance }}磁盘几乎用完了inodes,剩余不足10%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      - alert: HostInodesWillFillIn24Hours
        expr: node_filesystem_files_free{mountpoint ="/rootfs"} / node_filesystem_files{mountpoint="/rootfs"} * 100 < 10 and predict_linear(node_filesystem_files_free{mountpoint="/rootfs"}[1h], 24 *3600) < 0 and ON (instance, device, mountpoint) node_filesystem_readonly{mountpoint="/rootfs"} == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: Host inodes will fill in 24 hours (instance {{ $labels.instance }})
          description: "{{ $labels.instance }}以当前写入速率,文件系统预计将在未来 24小时内耗尽inode\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      - alert: HostUnusualDiskReadLatency
        expr: rate(node_disk_read_time_seconds_total[1m]) / rate(node_disk_reads_completed_total[1m]) > 0.2 and rate(node_disk_reads_completed_total[1m]) > 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: (instance {{ $labels.instance }})实例磁盘读取延迟异常
          description: "磁盘读取延迟正在增长,当下大于200ms已持续1分钟\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"		  
      - alert: HostUnusualDiskWriteLatency
        expr: rate(node_disk_write_time_seconds_total[1m]) / rate(node_disk_writes_completed_total[1m]) > 0.1 and rate(node_disk_writes_completed_total[1m]) > 0
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: (instance {{ $labels.instance }})实例磁盘写入延迟异常
          description: "磁盘写入延迟正在增长,当下大于100ms已持续1分钟\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"      	  
      - alert: HostHighCpuLoad
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100) > 95
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: (instance {{ $labels.instance }})实例CPU负载过高
          description: "当前CPU load高于95%持续1分钟\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      - alert: HostCpuStealNoisyNeighbor
        expr: avg by(instance) (rate(node_cpu_seconds_total{mode="steal"}[5m])) * 100 > 10
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: (instance {{ $labels.instance }})CPU steal time开始增长
          description: "CPU steal time大于10%已持续5分钟. 这意味着CPU资源不足或vm资源竞争。当前实例计算能力将开始下降.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      # 1000 context switches is an arbitrary number.
      # Alert threshold depends on nature of application.
      # Please read: https://github.com/samber/awesome-prometheus-alerts/issues/58
      - alert: HostContextSwitching
        expr: (rate(node_context_switches_total[5m])) / (count without(cpu, mode) (node_cpu_seconds_total{mode="idle"})) > 10000
        for: 0m
        labels:
          severity: warning
        annotations:
          summary: (instance {{ $labels.instance }})主机上下文切换异常
          description: "实例上下文切换正在增长,当下超过10000/s)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      - alert: HostSwapIsFillingUp
        expr: (1 - (node_memory_SwapFree_bytes / node_memory_SwapTotal_bytes)) * 100 > 80
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: (instance {{ $labels.instance }})swap内存即将用完
          description: "Swap使用超过80% \n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      - alert: HostSystemdServiceCrashed
        expr: node_systemd_unit_state{state="failed"} == 1
        for: 0m
        labels:
          severity: critical
        annotations:
          summary: (instance {{ $labels.instance }})当前实例systemd service崩溃
          description: "当前实例systemd service崩溃\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      - alert: HostKernelVersionDeviations
        expr: count(sum(label_replace(node_uname_info, "kernel", "$1", "release", "([0-9]+.[0-9]+.[0-9]+).*")) by (kernel)) > 1
        for: 6h
        labels:
          severity: warning
        annotations:
          summary: (instance {{ $labels.instance }})实例内核版本偏差
          description: "正在运行的内核版本:\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      - alert: HostOomKillDetected
        expr: increase(node_vmstat_oom_kill[1m]) > 0
        for: 0m
        labels:
          severity: critical
        annotations:
          summary: (instance {{ $labels.instance }})实例发生OOM kill事件
          description: "检测到OOM kill\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      - alert: HostNetworkReceiveErrors
        expr: rate(node_network_receive_errs_total[2m]) / rate(node_network_receive_packets_total[2m]) > 0.01
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: (instance {{ $labels.instance }})主机网络接收错误
          description: "主机 {{ $labels.instance }} interface {{ $labels.device }} 在过去五分钟内遇到了 {{ printf \"%.0f\" $value }} 接收错误。\n VALUE = { { $value }}\n 标签 = {{ $labels }}"
      - alert: HostNetworkTransmitErrors
        expr: rate(node_network_transmit_errs_total[2m]) / rate(node_network_transmit_packets_total[2m]) > 0.01
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: (instance {{ $labels.instance }})主机网络传输错误
          description: "主机 {{ $labels.instance }} interface {{ $labels.device }} 在过去五分钟内遇到了 {{ printf \"%.0f\" $value }} 传输错误。\n VALUE = { { $value }}\n 标签 = {{ $labels }}"
      - alert: HostNetworkInterfaceSaturated
        expr: (rate(node_network_receive_bytes_total{device!~"^tap.*"}[1m]) + rate(node_network_transmit_bytes_total{device!~"^tap.*"}[1m])) / node_network_speed_bytes{device!~"^tap.*"} > 0.8
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: (instance {{ $labels.instance }})网络接口接近饱和
          description: "\"{{ $labels.instance }}\" 上的网络接口 \"{{ $labels.interface }}\" 正在过载。\n VALUE = {{ $value }}\n LABELS = { { $labels }}"
      - alert: HostConntrackLimit
        expr: node_nf_conntrack_entries / node_nf_conntrack_entries_limit > 0.8
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: 主机 conntrack 限制(instance {{ $labels.instance }})
          description: "conntrack 数量接近限制\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
      - alert: HostClockSkew
        expr: (node_timex_offset_seconds > 0.05 and deriv(node_timex_offset_seconds[5m]) >= 0) or (node_timex_offset_seconds < -0.05 and deriv(node_timex_offset_seconds[5m]) <= 0)
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: 主机时钟偏差(instance {{ $labels.instance }})
          description: "检测到时钟偏差。时钟不同步。\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
      - alert: HostClockNotSynchronising
        expr: min_over_time(node_timex_sync_status[1m]) == 0 and node_timex_maxerror_seconds >= 16
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: 主机时钟不同步(instance {{ $labels.instance }})
          description: "时钟不同步.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      - alert: HostEdacCorrectableErrorsDetected
        expr: increase(node_edac_correctable_errors_total[1m]) > 0
        for: 0m
        labels:
          severity: info
        annotations:
          summary: Host EDAC Correctable Errors detected (instance {{ $labels.instance }})
          description: "Host {{ $labels.instance }} has had {{ printf \"%.0f\" $value }} correctable memory errors reported by EDAC in the last 5 minutes.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      - alert: HostEdacUncorrectableErrorsDetected
        expr: node_edac_uncorrectable_errors_total > 0
        for: 0m
        labels:
          severity: warning
        annotations:
          summary: Host EDAC Uncorrectable Errors detected (instance {{ $labels.instance }})
          description: "Host {{ $labels.instance }} has had {{ printf \"%.0f\" $value }} uncorrectable memory errors reported by EDAC in the last 5 minutes.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      - alert: HostPhysicalComponentTooHot
        expr: node_hwmon_temp_celsius > 75
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: (instance {{ $labels.instance }})宿主机组件过热
          description: "物理硬件过热\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      - alert: HostNodeOvertemperatureAlarm
        expr: node_hwmon_temp_crit_alarm_celsius == 1
        for: 0m
        labels:
          severity: critical
        annotations:
          summary: 物理节点超温告警(instance {{ $labels.instance }})
          description: "物理节点超温告警\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      - alert: HostRaidArrayGotInactive
        expr: node_md_state{state="inactive"} > 0
        for: 0m
        labels:
          severity: critical
        annotations:
          summary:  RAID阵列处于非活动状态(instance {{ $labels.instance }})
          description: "RAID array {{ $labels.device }} is in degraded state due to one or more disks failures. Number of spare drives is insufficient to fix issue automatically.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      - alert: HostRaidDiskFailure
        expr: node_md_disks{state="failed"} > 0
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: RAID磁盘故障(instance {{ $labels.instance }})
          description: "At least one device in RAID array on {{ $labels.instance }} failed. Array {{ $labels.md_device }} needs attention and possibly a disk swap\n  VALUE = {{ $value }}\n  LABELS ={{ $labels }}"
      - alert: node_disk_Utilization_high
        expr: rate(node_disk_io_time_seconds_total[5m]) * 100 > 90
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "{{$labels.instance}}: disk Utilization > {{ $value }} % "
          description: "{{$labels.instance}}: disk Utilization > {{ $value }}. this should attract attention . disk performance seems to be problematic .for 1 minutes"  
```

##### 磁盘使用率

```
 rate(node_disk_io_time_seconds_total[5m]) * 100 
```

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    prometheus: k8s
    role: alert-rules
  name: node-rules
  namespace: monitoring
spec:
  groups:
    - name: external_node_alarm
      rules:
      - alert: node_disk_Utilization_high
        expr: rate(node_disk_io_time_seconds_total[5m]) * 100 > 90
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "{{$labels.instance}}: disk Utilization > {{ $value }} % "
          description: "{{$labels.instance}}: disk Utilization > {{ $value }}. this should attract attention . disk performance seems to be problematic .for 1 minutes"
```



#### nginx

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    prometheus: k8s
    role: alert-rules
  name: nginx-rules
  namespace: monitoring
spec:
  groups:
  - name: NginxStatsAlert
    rules:
    - alert: nginx is down
      expr: up{job="external-nginx"} == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "{{ $labels.instance }} Nginx is down"
        description: "nginx is down. This requires immediate action! {{ $labels.instance }} "
```

#### redis

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    prometheus: k8s
    role: alert-rules
  name: redis-rules
  namespace: monitoring
spec:
  groups:
  - name: redisStatsAlert
    rules:
    - alert: redis is down
      expr: up{job="external-redis"} == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "{{ $labels.instance }} redis is down"
        description: "{{ $labels.instance }} redis is down "
```

#### rabbitmq

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    prometheus: k8s
    role: alert-rules
  name: rabbitmq-rules
  namespace: monitoring
spec:
  groups:
  - name: rabbitmqStatsAlert
    rules:
    - alert: rabbitmq is down
      expr: up{job="external-rabbitmq"} == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "{{ $labels.instance }} rabbitmq is down"
        description: "{{ $labels.instance }} rabbitmq is down "
```



#### mysql

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    prometheus: k8s
    role: alert-rules
  name: mysql-rules
  namespace: monitoring
spec:
  groups:
  - name: MySQLStatsAlert
    rules:
    - alert: MySQL is down
      expr: mysql_up == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "{{ $labels.instance }} MySQL is down"
        description: "MySQL database is down. This requires immediate action!"
    - alert: MysqlTooManyConnections(>80%)
      expr: avg by (instance) (rate(mysql_global_status_threads_connected[1m])) / avg by (instance) (mysql_global_variables_max_connections) * 100 > 80
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: MySQL too many connections (> 80%) (instance {{ $labels.instance }})
        description: "超过 80% 的 MySQL 连接在 {{ $labels.instance }} 上使用\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
    - alert: MysqlHighThreadsRunning
      expr: avg by (instance) (rate(mysql_global_status_threads_running[1m])) / avg by (instance) (mysql_global_variables_max_connections) * 100 > 60
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: MySQL high threads running (instance {{ $labels.instance }})
        description: "超过 60% 的 MySQL 连接在 {{ $labels.instance }} 上处于运行状态\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
    - alert: MysqlRestarted
      expr: mysql_global_status_uptime < 60
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: MySQL重新启动 (instance {{ $labels.instance }})
        description: "MySQL 刚刚重新启动，不到一分钟前在 {{ $labels.instance }} 上。\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
  ##  MySQL Slave IO 线程未运行
  #- alert: MysqlSlaveIoThreadNotRunning
  #  expr: mysql_slave_status_master_server_id > 0 and ON (instance) mysql_slave_status_slave_io_running == 0
  #  for: 0m
  #  labels:
  #    severity: critical
  #  annotations:
  #    summary: MySQL Slave IO thread not running (instance {{ $labels.instance }})
  #    description: "MySQL Slave IO thread not running on {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"	  
  ##  MySQL Slave SQL 线程未运行
  #- alert: MysqlSlaveSqlThreadNotRunning
  #  expr: mysql_slave_status_master_server_id > 0 and ON (instance) mysql_slave_status_slave_sql_running == 0
  #  for: 0m
  #  labels:
  #    severity: critical
  #  annotations:
  #    summary: MySQL Slave SQL thread not running (instance {{ $labels.instance }})
  #    description: "MySQL Slave SQL thread not running on {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
  ## MySQL Slave 复制滞后
  #- alert: MysqlSlaveReplicationLag
  #  expr: mysql_slave_status_master_server_id > 0 and ON (instance) (mysql_slave_status_seconds_behind_master - mysql_slave_status_sql_delay) > 30
  #  for: 1m
  #  labels:
  #    severity: critical
  #  annotations:
  #    summary: MySQL Slave replication lag (instance {{ $labels.instance }})
  #    description: "MySQL replication lag on {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
  ## MySQL 慢查询
  #- alert: MysqlSlowQueries
  #  expr: increase(mysql_global_status_slow_queries[1m]) > 0
  #  for: 2m
  #  labels:
  #    severity: warning
  #  annotations:
  #    summary: MySQL slow queries (instance {{ $labels.instance }})
  #    description: "MySQL server mysql has some new slow query.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"  
  ## MySQL innodb 日志写入停滞 MySQL InnoDB 日志等待
  #- alert: MysqlInnodbLogWaits
  #  expr: rate(mysql_global_status_innodb_log_waits[15m]) > 10
  #  for: 0m
  #  labels:
  #    severity: warning
  #  annotations:
  #    summary: MySQL InnoDB log waits (instance {{ $labels.instance }})
  #    description: "MySQL innodb log writes stalling\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

#### kafka



```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    prometheus: k8s
    role: alert-rules
  name: kafka-rules
  namespace: monitoring
spec:
  groups:
  - name: external_kafka_alarm
    rules:
    - alert: KafkaTopicsReplicas
      expr: sum(kafka_topic_partition_in_sync_replica) by (topic) < 3
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Kafka topics replicas (instance {{ $labels.instance }})
        description: "Kafka 副本分区\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
    - alert: kafka_is_down
      expr: up{job="external-kafka"} == 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Kafka server is down (instance {{ $labels.instance }})
        description: " {{ $labels.instance }} Kafka server is down now!"
    - alert: KafkaConsumersGroup
      expr: sum(kafka_consumergroup_lag) by (consumergroup) > 8888
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: Kafka consumers group (instance {{ $labels.instance }})
        description: "Kafka 消费者组\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"		
    - alert: KafkaTopicOffsetDecreased
      expr: delta(kafka_burrow_partition_current_offset[1m]) < 0
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Kafka topic offset decreased (instance {{ $labels.instance }})
        description: "Kafka 主题偏移量已减少\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
    - alert: KafkaConsumerLag
      expr: 'kafka_burrow_topic_partition_offset - on(partition, cluster, topic) group_right() kafka_burrow_partition_current_offset >= (kafka_burrow_topic_partition_offset offset 15m - on(partition, cluster, topic) group_right() kafka_burrow_partition_current_offset offset 15m)
      AND kafka_burrow_topic_partition_offset - on(partition, cluster, topic) group_right() kafka_burrow_partition_current_offset > 0'
      for: 15m
      labels:
        severity: critical
      annotations:
        summary: Kafka consumer lag (instance {{ $labels.instance }})
        description: "卡夫卡消费者有 30 分钟的延迟并且越来越多\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"	 
```

#### mongodb

grafana: https://github.com/percona/grafana-dashboards/tree/main/dashboards

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    prometheus: k8s
    role: alert-rules
  name: mongodb-rules
  namespace: monitoring
spec:
  groups:
  - name: external_mongodb_alarm
    rules:
    - alert: MongodbDown
      expr: mongodb_up == 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: MongoDB Down (instance {{ $labels.instance }})
        description: "MongoDB 实例已关闭\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
    - alert: MongodbReplicationLag
      #expr: mongodb_mongod_replset_member_optime_date{state="PRIMARY"} - ON (set) mongodb_mongod_replset_member_optime_date{state="SECONDARY"} > 10
      expr: avg(mongodb_mongod_replset_member_optime_date{state="PRIMARY"})-avg(mongodb_mongod_replset_member_optime_date{state="SECONDARY"}) > 10
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: MongoDB replication lag (instance {{ $labels.instance }})
        description: "Mongodb 复制延迟超过 10 秒\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
    - alert: MongodbNumberCursorsOpen
      expr: mongodb_mongod_metrics_cursor_open{state="total"} > 10 * 1000
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: MongoDB number cursors open (instance {{ $labels.instance }})
        description: "MongoDB 为客户端打开了太多cursors （> 10k）\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
    - alert: MongodbCursorsTimeouts
      expr: increase(mongodb_mongod_metrics_cursor_timed_out_total[1m]) > 100
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: MongoDB cursors timeouts (instance {{ $labels.instance }})
        description: "太多的cursors 超时\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: MongodbTooManyConnections
      expr: avg by(instance) (rate(mongodb_connections{state="current"}[1m])) / avg by(instance) (sum (mongodb_connections) by (instance)) * 100 > 80
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: MongoDB too many connections (instance {{ $labels.instance }})
        description: "连接过多 (> 80%)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

#### obd

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    prometheus: k8s
    role: alert-rules
  name: gisObd-rules
  namespace: monitoring
spec:
  groups:
  - name: external_gisObd_alarm
    rules:
    - alert: ObdshutDown
      expr: gis_uptime_second == 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: obd Down (instance {{ $labels.instance }})
        description: "obd 服务已关闭\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
```

#### windows

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    prometheus: k8s
    role: alert-rules
  name: windows-rules
  namespace: monitoring
spec:
  groups:
  - name: external_windows_alarm
    rules:
    - alert: windows-server down
      expr: up{job="external-windows-node"} == 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: windows Down (instance {{ $labels.instance }})
        description: "{{ $labels.instance }} windows已关闭\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
```

#### etcd

etcdHighNumberOfFailedGRPCRequests : https://github.com/rancher/rancher/issues/29939

https://monitoring.mixins.dev/etcd/

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    prometheus: k8s
    role: alert-rules
  name: etcd-rules
  namespace: monitoring
spec:
  groups:
  - name: external_etcd_alarm
    rules:
    - alert: EtcdInsufficientMembers
      expr: count(etcd_server_id) % 2 == 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: etcd cluster 已崩溃 (instance {{ $labels.instance }})
        description: "Etcd 集群应该有奇数个成员, \n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
    - alert: EtcdNoLeader
      expr: etcd_server_has_leader == 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Etcd no Leader (instance {{ $labels.instance }})
        description: "Etcd cluster have no leader\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: EtcdHighNumberOfLeaderChanges
      expr: increase(etcd_server_leader_changes_seen_total[10m]) > 2
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Etcd leader 切换异常 (instance {{ $labels.instance }})
        description: "10分钟内leader切换了两次\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"        
    - alert: EtcdHighNumberOfFailedGrpcRequests
      expr: 100 * sum by(job, instance, grpc_service, grpc_method) (rate(grpc_server_handled_total{grpc_code!="OK",job=~".*etcd.*"}[5m])) / sum by(job, instance, grpc_service, grpc_method) (rate(grpc_server_handled_total{job=~".*etcd.*"}[5m]))> 1
      for: 10m
      labels:
        severity: warning
      annotations:
        summary: Etcd 大量失败的 GRPC 请求 (instance {{ $labels.instance }})
        description: "在 Etcd 中检测到超过 1% 的 GRPC 请求失败\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"        
    - alert: EtcdHighNumberOfFailedGrpcRequests     
      expr: 100 * sum(rate(grpc_server_handled_total{job=~".*etcd.*", grpc_code!="OK"}[5m])) without (grpc_type, grpc_code) / sum(rate(grpc_server_handled_total{job=~".*etcd.*"}[5m])) without (grpc_type, grpc_code) > 5
      for: 5m
      labels:
        severity: critical
      annotations:
        summary:  Etcd 大量失败的 GRPC 请求  (instance {{ $labels.instance }})
        description: "在 Etcd 中检测到超过 5% 的 GRPC 请求失败\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"                       
    - alert: EtcdGrpcRequestsSlow
      expr: histogram_quantile(0.99, sum(rate(grpc_server_handling_seconds_bucket{grpc_type="unary"}[1m])) by (grpc_service, grpc_method, le)) > 0.15
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Etcd GRPC 请求缓慢(instance {{ $labels.instance }})
        description: "GRPC 请求变慢，99% 超过 0.15s\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"	  
    - alert: EtcdHighNumberOfFailedHttpRequests
      expr: sum(rate(etcd_http_failed_total[1m])) BY (method) / sum(rate(etcd_http_received_total[1m])) BY (method) > 0.01
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Etcd 大量失败的 HTTP 请求 (instance {{ $labels.instance }})
        description: "在 Etcd 中检测到超过 1% 的 HTTP 失败\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"	
    - alert: EtcdHighNumberOfFailedHttpRequests
      expr: sum(rate(etcd_http_failed_total[1m])) BY (method) / sum(rate(etcd_http_received_total[1m])) BY (method) > 0.05
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: Etcd 大量失败的 HTTP 请求  (instance {{ $labels.instance }})
        description: "在 Etcd 中检测到超过 5% 的 HTTP 失败\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"	 
    - alert: EtcdHttpRequestsSlow
      expr: histogram_quantile(0.99, rate(etcd_http_successful_duration_seconds_bucket[1m])) > 0.15
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Etcd HTTP 请求缓慢 (instance {{ $labels.instance }})
        description: "Etcd HTTP 请求变慢，99% 超过 0.15s\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: EtcdMemberCommunicationSlow
      expr: histogram_quantile(0.99, rate(etcd_network_peer_round_trip_time_seconds_bucket[1m])) > 0.15
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Etcd成员通讯缓慢 (instance {{ $labels.instance }})
        description: "Etcd 成员通信变慢，99% 超过 0.15s\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: EtcdHighNumberOfFailedProposals
      expr: increase(etcd_server_proposals_failed_total[1h]) > 5
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Etcd 大量失败的proposals  (instance {{ $labels.instance }})
        description: "Etcd 服务器在过去一小时收到了超过 5 个失败的proposals\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: EtcdHighFsyncDurations
      expr: histogram_quantile(0.99, rate(etcd_disk_wal_fsync_duration_seconds_bucket[1m])) > 0.5
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Etcd fsync 持续时间变高 (instance {{ $labels.instance }})
        description: "Etcd WAL fsync 持续时间增加，99% 超过 0.5s\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"	
    - alert: EtcdHighCommitDurations
      expr: histogram_quantile(0.99, rate(etcd_disk_backend_commit_duration_seconds_bucket[1m])) > 0.25
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Etcd 提交持续时间较高 (instance {{ $labels.instance }})
        description: "Etcd 提交持续时间增加，99% 超过 0.25s\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"	 
```

### 17.2 告警分组

钉钉测试

```
curl "https://oapi.dingtalk.com/robot/send?access_token=TOKEN" -H "Content-Type: application/json" -d "{'msgtype': 'text','text': {'content': 'label'}}"
```

我们通过两个分组来区分告警，分别是所有和critical，并创建两个群，在两个群分别创建两个机器人，通过webhook分别发送到两个群。

- webhook2作为全局的告警
- webhookcritical发送critical的日志

必要条件： 

创建钉钉机器人

cat alertmanager-main.yaml

```
apiVersion: v1
data: {}
kind: Secret
metadata:
  name: alertmanager-main
  namespace: monitoring
stringData:
  alertmanager.yaml: |-
    global:
      resolve_timeout: 5m
    route:
      group_by: ['alertname']
      group_wait: 10s
      group_interval: 10s
      repeat_interval: 45m
      receiver: 'webhook2'
      routes:
      - receiver: 'webhookcritical'
        match:
          severity: 'critical'
      - receiver: 'webhook2'
        match:
          severity: '~(warning)$'
    receivers:
    - name: 'webhook2'
      webhook_configs:
      - send_resolved: true
        url: 'http://webhook-dingtalk:8060/dingtalk/webhook/send'
    - name: 'webhookcritical'
      webhook_configs:
      - send_resolved: true
        url: 'http://webhook-dingtalk1:8061/dingtalk/webhook/send'
    inhibit_rules:
      - source_match:
          severity: 'critical'
        target_match:
          severity: 'warning'
        equal: ['alertname', 'dev', 'instance']
```

#### 17.2.1 8060.yaml

发送到8060的配置 

- warning

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: webhook-dingtalk
  name: webhook-dingtalk
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webhook-dingtalk
  template:
    metadata:
      labels:
        app: webhook-dingtalk
    spec:
      containers:
      - image: marksugar/k8s-prometheus:prometheus_dingtalk_v0.2
        name: webhook-dingtalk
        args:
        - --web.listen-address=:8060
        - --config.file=/etc/prometheus-webhook-dingtalk/config.yml
        volumeMounts:
        - name: webdingtalk-configmap
          mountPath: /etc/prometheus-webhook-dingtalk/
        - name: webdingtalk-template
          mountPath: /etc/prometheus-webhook-dingtalk/templates/
        ports:
        - containerPort: 8060
          protocol: TCP
      imagePullSecrets:
        - name: IfNotPresent
      volumes:
        - name: webdingtalk-configmap
          configMap:
            name: dingtalk-config
        - name: webdingtalk-template
          configMap:
            name: dingtalk-template       
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: webhook-dingtalk
  name: webhook-dingtalk
  namespace: monitoring
spec:
  ports:
  - name: http
    port: 8060
    protocol: TCP
    targetPort: 8060
    selector:
    app: webhook-dingtalk
    type: ClusterIP
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: dingtalk-config
  namespace: monitoring
  labels:
    app: dingtalk-config
data:
  config.yml: |
    templates:
      - /etc/prometheus-webhook-dingtalk/templates/default.tmpl
    targets:
      webhook:
        url: https://oapi.dingtalk.com/robot/send?access_token=a452bf5a83ef2f036ec94e84e299a824f92106b9f74fff
        secret: SEC3111192e1af62bde446eb9855c324d38038
        message:
          title: '{{ template "ding.link.title" . }}'
          text: '{{ template "ding.link.content" . }}'
```



```
apiVersion: v1
kind: ConfigMap
metadata:
  name: dingtalk-template
  namespace: monitoring
  labels:
    app: dingtalk-template
data:
  default.tmpl: |
    {{ define "__subject" }}[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}] {{ .GroupLabels.SortedPairs.Values | join " " }} {{ if gt (len .CommonLabels) (len .GroupLabels) }}({{ with .CommonLabels.Remove .GroupLabels.Names }}{{ .Values | join " " }}{{ end }}){{ end }}{{ end }}
    {{ define "__alertmanagerURL" }}{{ .ExternalURL }}/#/alerts?receiver={{ .Receiver }}{{ end }}

    {{ define "__text_alert_list" }}{{ range . }}
    **Labels**
    {{ range .Labels.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}
    {{ end }}
    **Annotations**
    {{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}
    {{ end }}
    **Source:** [{{ .GeneratorURL }}]({{ .GeneratorURL }})
    {{ end }}{{ end }}
    
    {{ define "default.__text_alert_list" }}{{ range . }}
    ---
    **alert level:** {{ .Labels.severity | upper }}
    
    **trigger time:** {{ dateInZone "2006.01.02 15:04:05" (.StartsAt) "Asia/Shanghai" }}
    
    **event info:**
    {{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}


    {{ end }}
    
    **event label:**
    {{ range .Labels.SortedPairs }}{{ if and (ne (.Name) "severity") (ne (.Name) "summary") (ne (.Name) "team") }}> - {{ .Name }}: {{ .Value | markdown | html }}
    {{ end }}{{ end }}
    {{ end }}
    {{ end }}
    {{ define "default.__text_alertresovle_list" }}{{ range . }}
    ---
    **alert level:** {{ .Labels.severity | upper }}
    
    **trigger time:** {{ dateInZone "2006.01.02 15:04:05" (.StartsAt) "Asia/Shanghai" }}
    
    **end time:** {{ dateInZone "2006.01.02 15:04:05" (.EndsAt) "Asia/Shanghai" }}
    
    **event info:**
    {{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}


    {{ end }}
    
    **event label:**
    {{ range .Labels.SortedPairs }}{{ if and (ne (.Name) "severity") (ne (.Name) "summary") (ne (.Name) "team") }}> - {{ .Name }}: {{ .Value | markdown | html }}
    {{ end }}{{ end }}
    {{ end }}
    {{ end }}
    
    {{/* Default */}}
    {{ define "default.title" }}{{ template "__subject" . }}{{ end }}
    {{ define "default.content" }}#### \[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}\] **[{{ index .GroupLabels "alertname" }}]({{ template "__alertmanagerURL" . }})**
    {{ if gt (len .Alerts.Firing) 0 -}}
    
    ![20210716143611.png](https://pic5.58cdn.com.cn/nowater/webim/big/n_v27ab5f3b496a2419ba8ae8b3cf27960f0.png)
    
    **====⚠️⚠️⚠️trigger alarm====**
    {{ template "default.__text_alert_list" .Alerts.Firing }}


    {{- end }}
    
    {{ if gt (len .Alerts.Resolved) 0 -}}
    **====[烟花]recover alarm====**	
    {{ template "default.__text_alertresovle_list" .Alerts.Resolved }}


    {{- end }}
    {{- end }}
    
    {{/* Legacy */}}
    {{ define "legacy.title" }}{{ template "__subject" . }}{{ end }}
    {{ define "legacy.content" }}#### \[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}\] **[{{ index .GroupLabels "alertname" }}]({{ template "__alertmanagerURL" . }})**
    {{ template "__text_alert_list" .Alerts.Firing }}
    {{- end }}
    
    {{/* Following names for compatibility */}}
    {{ define "ding.link.title" }}{{ template "default.title" . }}{{ end }}
    {{ define "ding.link.content" }}{{ template "default.content" . }}{{ end }}
```



#### 17.2.2 8061.yaml

发送到8061的配置

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: webhook-dingtalk1
  name: webhook-dingtalk1
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webhook-dingtalk1
  template:
    metadata:
      labels:
        app: webhook-dingtalk1
    spec:
      containers:
      - image: marksugar/k8s-prometheus:prometheus_dingtalk_v0.2
        name: webhook-dingtalk1
        args:
        - --web.listen-address=:8061
        - --config.file=/etc/prometheus-webhook-dingtalk/config.yml
        volumeMounts:
        - name: webdingtalk-configmap1
          mountPath: /etc/prometheus-webhook-dingtalk/
        - name: webdingtalk-template1
          mountPath: /etc/prometheus-webhook-dingtalk/templates/
        ports:
        - containerPort: 8061
          protocol: TCP
      imagePullSecrets:
        - name: IfNotPresent
      volumes:
        - name: webdingtalk-configmap1
          configMap:
            name: dingtalk-config1
        - name: webdingtalk-template1
          configMap:
            name: dingtalk-template1
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: webhook-dingtalk1
  name: webhook-dingtalk1
  namespace: monitoring
spec:
  ports:
  - name: http
    port: 8061
    protocol: TCP
    targetPort: 8061
    selector:
    app: webhook-dingtalk1
    type: ClusterIP
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: dingtalk-config1
  namespace: monitoring
  labels:
    app: dingtalk-config1
data:
  config.yml: |
    templates:
      - /etc/prometheus-webhook-dingtalk/templates/default.tmpl
    targets:
      webhook:
        url: https://oapi.dingtalk.com/robot/send?access_token=cdb633bc3b82398e90899d5a7d1349ec0d798547e
        secret: SECb0039317816c114b6f5d995093f0
        message:
          title: '{{ template "ding.link.title" . }}'
          text: '{{ template "ding.link.content" . }}'
```

dingtalk-template1

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: dingtalk-template1
  namespace: monitoring
  labels:
    app: dingtalk-template1
data:
  default.tmpl: |
    {{ define "__subject" }}[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}] {{ .GroupLabels.SortedPairs.Values | join " " }} {{ if gt (len .CommonLabels) (len .GroupLabels) }}({{ with .CommonLabels.Remove .GroupLabels.Names }}{{ .Values | join " " }}{{ end }}){{ end }}{{ end }}
    {{ define "__alertmanagerURL" }}{{ .ExternalURL }}/#/alerts?receiver={{ .Receiver }}{{ end }}

    {{ define "__text_alert_list" }}{{ range . }}
    **Labels**
    {{ range .Labels.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}
    {{ end }}
    **Annotations**
    {{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}
    {{ end }}
    **Source:** [{{ .GeneratorURL }}]({{ .GeneratorURL }})
    {{ end }}{{ end }}
    
    {{ define "default.__text_alert_list" }}{{ range . }}
    ---
    **alert level:** {{ .Labels.severity | upper }}
    
    **trigger time:** {{ dateInZone "2006.01.02 15:04:05" (.StartsAt) "Asia/Shanghai" }}
    
    **event info:**
    {{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}


    {{ end }}
    
    **event label:**
    {{ range .Labels.SortedPairs }}{{ if and (ne (.Name) "severity") (ne (.Name) "summary") (ne (.Name) "team") }}> - {{ .Name }}: {{ .Value | markdown | html }}
    {{ end }}{{ end }}
    {{ end }}
    {{ end }}
    {{ define "default.__text_alertresovle_list" }}{{ range . }}
    ---
    **alert level:** {{ .Labels.severity | upper }}
    
    **trigger time:** {{ dateInZone "2006.01.02 15:04:05" (.StartsAt) "Asia/Shanghai" }}
    
    **end time:** {{ dateInZone "2006.01.02 15:04:05" (.EndsAt) "Asia/Shanghai" }}
    
    **event info:**
    {{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}


    {{ end }}
    
    **event label:**
    {{ range .Labels.SortedPairs }}{{ if and (ne (.Name) "severity") (ne (.Name) "summary") (ne (.Name) "team") }}> - {{ .Name }}: {{ .Value | markdown | html }}
    {{ end }}{{ end }}
    {{ end }}
    {{ end }}
    
    {{/* Default */}}
    {{ define "default.title" }}{{ template "__subject" . }}{{ end }}
    {{ define "default.content" }}#### \[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}\] **[{{ index .GroupLabels "alertname" }}]({{ template "__alertmanagerURL" . }})**
    {{ if gt (len .Alerts.Firing) 0 -}}
    
    ![20210716143611.png](https://pic5.58cdn.com.cn/nowater/webim/big/n_v27ab5f3b496a2419ba8ae8b3cf27960f0.png)
    
    **====⚠️⚠️⚠️trigger alarm====**
    {{ template "default.__text_alert_list" .Alerts.Firing }}


    {{- end }}
    
    {{ if gt (len .Alerts.Resolved) 0 -}}
    **====[烟花]recover alarm====**	
    {{ template "default.__text_alertresovle_list" .Alerts.Resolved }}


    {{- end }}
    {{- end }}
    
    {{/* Legacy */}}
    {{ define "legacy.title" }}{{ template "__subject" . }}{{ end }}
    {{ define "legacy.content" }}#### \[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}\] **[{{ index .GroupLabels "alertname" }}]({{ template "__alertmanagerURL" . }})**
    {{ template "__text_alert_list" .Alerts.Firing }}
    {{- end }}
    
    {{/* Following names for compatibility */}}
    {{ define "ding.link.title" }}{{ template "default.title" . }}{{ end }}
    {{ define "ding.link.content" }}{{ template "default.content" . }}{{ end }}
```



## 18.修改原有的规则

查看

```
kubectl -n monitoring get PrometheusRule
NAME                              AGE
alertmanager-main-rules           53d
gis-rules                         28m
kafka-rules                       36m
kube-prometheus-rules             53d
kube-state-metrics-rules          53d
kubernetes-monitoring-rules       53d
mongodb-rules                     36m
mysql-rules                       36m
node-exporter-rules               53d
node-rules                        39m
prometheus-k8s-prometheus-rules   53d
prometheus-operator-rules         53d
```

修改node

```
 kubectl -n monitoring edit PrometheusRule  node-exporter-rules
```

找到要修改的规则进行修改即可

- 修改Watchdog时间为for: 10m

 kubectl -n monitoring edit PrometheusRule  kube-prometheus-rules

```
    - alert: Watchdog
      annotations:
        description: |
          This is an alert meant to ensure that the entire alerting pipeline is functional.
          This alert is always firing, therefore it should always be firing in Alertmanager
          and always fire against a receiver. There are integrations with various notification
          mechanisms that send a notification when this alert is not firing. For example the
          "DeadMansSnitch" integration in PagerDuty.
          (这是一条探测的警告信息，皆在确保整个警报工作正常。此警告消息始终在触发)
        runbook_url: https://github.com/prometheus-operator/kube-prometheus/wiki/watchdog
        summary: An alert that should always be firing to certify that Alertmanager
          is working properly.(当你收到这条警告消息，说明报警发送是正常的)
      expr: vector(1)
      for: 15m
      labels:
        severity: none
```



- 修改其他的

- CPUThrottlingHigh

  kubectl -n monitoring edit PrometheusRule  kubernetes-monitoring-rules

  CPU使用超过了pod给定的值
  
  https://blog.csdn.net/alex_yangchuansheng/article/details/109063996

## 19.外部监控k8s

### 19.1 授权

cat prom.rbac.yaml授权文件

```
# prom.rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus
  namespace: kube-mon
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups:
  - ""
  resources:
  - nodes
  - services
  - endpoints
  - pods
  - nodes/proxy
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - "extensions"
  resources:
    - ingresses
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - configmaps
  - nodes/metrics
  verbs:
  - get
- nonResourceURLs:
  - /metrics
  verbs:
  - get
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: kube-mon
```

获取token

```
 kubectl get secret  `kubectl get secret -n kube-mon|grep prometheus-token|awk '{print $1}'` -o jsonpath={.data.token} -n kube-mon |base64 -d
```
获取后复制到监控节点存放为/etc/token


> 2
>
> ```
> apiVersion: v1
> kind: Service
> metadata:
>   labels:
>     app.kubernetes.io/name: kube-state-metrics
>     app.kubernetes.io/version: 2.1.0
>   name: kube-state-metrics
>   namespace: kube-system
> spec:
>   clusterIP: None
>   ports:
>   - name: http-metrics
>     port: 8080
>     targetPort: http-metrics
>   - name: telemetry
>     port: 8081
>     targetPort: telemetry
>   selector:
>     app.kubernetes.io/name: kube-state-metrics
> ---
> 
> apiVersion: apps/v1
> kind: Deployment
> metadata:
>   labels:
>     app.kubernetes.io/name: kube-state-metrics
>     app.kubernetes.io/version: 2.1.0
>   name: kube-state-metrics
>   namespace: kube-system
> spec:
>   replicas: 1
>   selector:
>     matchLabels:
>       app.kubernetes.io/name: kube-state-metrics
>   template:
>     metadata:
>       labels:
>         app.kubernetes.io/name: kube-state-metrics
>         app.kubernetes.io/version: 2.1.0
>     spec:
>       containers:
>       - image:  marksugar/k8s-prometheus:kube-state-metrics-v2.0.0
>         livenessProbe:
>           httpGet:
>             path: /healthz
>             port: 8080
>           initialDelaySeconds: 5
>           timeoutSeconds: 5
>         name: kube-state-metrics
>         ports:
>         - containerPort: 8080
>           name: http-metrics
>         - containerPort: 8081
>           name: telemetry
>         readinessProbe:
>           httpGet:
>             path: /
>             port: 8081
>           initialDelaySeconds: 5
>           timeoutSeconds: 5
>         securityContext:
>           runAsUser: 65534
>       nodeSelector:
>         kubernetes.io/os: linux
>       serviceAccountName: kube-state-metrics
> ---
> apiVersion: rbac.authorization.k8s.io/v1
> kind: ClusterRoleBinding
> metadata:
>   labels:
>     app.kubernetes.io/name: kube-state-metrics
>     app.kubernetes.io/version: 2.1.0
>   name: kube-state-metrics
> roleRef:
>   apiGroup: rbac.authorization.k8s.io
>   kind: ClusterRole
>   name: kube-state-metrics
> subjects:
> - kind: ServiceAccount
>   name: kube-state-metrics
>   namespace: kube-system
> ---
> apiVersion: rbac.authorization.k8s.io/v1
> kind: ClusterRole
> metadata:
>   labels:
>     app.kubernetes.io/name: kube-state-metrics
>     app.kubernetes.io/version: 2.1.0
>   name: kube-state-metrics
> rules:
> - apiGroups:
>   - ""
>   resources:
>   - configmaps
>   - secrets
>   - nodes
>   - pods
>   - services
>   - resourcequotas
>   - replicationcontrollers
>   - limitranges
>   - persistentvolumeclaims
>   - persistentvolumes
>   - namespaces
>   - endpoints
>   verbs:
>   - list
>   - watch
> - apiGroups:
>   - apps
>   resources:
>   - statefulsets
>   - daemonsets
>   - deployments
>   - replicasets
>   verbs:
>   - list
>   - watch
> - apiGroups:
>   - batch
>   resources:
>   - cronjobs
>   - jobs
>   verbs:
>   - list
>   - watch
> - apiGroups:
>   - autoscaling
>   resources:
>   - horizontalpodautoscalers
>   verbs:
>   - list
>   - watch
> - apiGroups:
>   - authentication.k8s.io
>   resources:
>   - tokenreviews
>   verbs:
>   - create
> - apiGroups:
>   - authorization.k8s.io
>   resources:
>   - subjectaccessreviews
>   verbs:
>   - create
> - apiGroups:
>   - policy
>   resources:
>   - poddisruptionbudgets
>   verbs:
>   - list
>   - watch
> - apiGroups:
>   - certificates.k8s.io
>   resources:
>   - certificatesigningrequests
>   verbs:
>   - list
>   - watch
> - apiGroups:
>   - storage.k8s.io
>   resources:
>   - storageclasses
>   - volumeattachments
>   verbs:
>   - list
>   - watch
> - apiGroups:
>   - admissionregistration.k8s.io
>   resources:
>   - mutatingwebhookconfigurations
>   - validatingwebhookconfigurations
>   verbs:
>   - list
>   - watch
> - apiGroups:
>   - networking.k8s.io
>   resources:
>   - networkpolicies
>   - ingresses
>   verbs:
>   - list
>   - watch
> - apiGroups:
>   - coordination.k8s.io
>   resources:
>   - leases
>   verbs:
>   - list
>   - watch
> ---
> apiVersion: v1
> kind: ServiceAccount
> metadata:
>   labels:
>     app.kubernetes.io/name: kube-state-metrics
>     app.kubernetes.io/version: 2.1.0
>   name: kube-state-metrics
>   namespace: kube-system
> ```

在每个节点部署node

```
version: '2.2'
services:
  node_exporter:
    image: prom/node-exporter:v1.1.2
    container_name: node_exporter
    user: root
    privileged: true
    network_mode: "host"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|host|etc)($$|/)'
    restart: unless-stopped
    cpu_shares: 14
    mem_limit: 50m
    logging:
      driver: "json-file"
      options:
        max-size: "200M"
    labels:
      - "SERVICE_TAGS=prometheus"
```

### 19.2 compose安装prom

```
version: '2.2'
services:
  prometheus:
    image: prom/prometheus:v2.27.1
    container_name: prometheus
    restart: always
    privileged: true
    network_mode: "host"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention=45d'
      - '--web.enable-lifecycle'
      - '--web.enable-admin-api'
    volumes:
    - /data/prom/prometheus/data/:/prometheus:rw  # NOTE: chown 65534.65534 /data/prom/prometheus/
    - /data/prom/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
    - /data/prom/prometheus/alert:/etc/prometheus/alert
    - /data/prom/prometheus/ssl:/etc/prometheus/ssl
    - /data/prom/prometheus/k8s/:/etc/
    cpu_shares: 60
    mem_limit: 2048m
    logging:
      driver: "json-file"
      options:
        max-size: "200M"
    labels:
      SERVICE_TAGS: prometheus
  grafana:
    image: grafana/grafana:8.0.3
    container_name: grafana
    privileged: true
    network_mode: "host"
    volumes:
      - /data/prom/grafana/data:/var/lib/grafana #NOTE chown 472.472 /data/prom/grafana/
      - /data/prom/grafana/datasources:/etc/grafana/datasources
      - /data/prom/grafana/dashboards:/etc/grafana/dashboards
    environment:
      - GF_SECURITY_ADMIN_USER=${ADMIN_USER:-admin}
      - GF_SECURITY_ADMIN_PASSWORD=${ADMIN_PASSWORD:-admin}
      - GF_USERS_ALLOW_SIGN_UP=false
    restart: unless-stopped
    cpu_shares: 20
    mem_limit: 512m
    logging:
      driver: "json-file"
      options:
        max-size: "200M"
    depends_on:
    - prometheus
    labels:
      SERVICE_TAGS: prometheus

  alertmanager:
    image: prom/alertmanager:v0.21.0
    container_name: alertmanager
    network_mode: "host"
    volumes:
      - /data/prom/alertmanager/:/etc/alertmanager/
    command:
      - '--config.file=/etc/alertmanager/config.yml'
      - '--storage.path=/alertmanager'
    restart: unless-stopped
    expose:
      - 9093
    cpu_shares: 10
    mem_limit: 128m
    logging:
      driver: "json-file"
      options:
        max-size: "200M"
    labels:
      SERVICE_TAGS: prometheus

  node_exporter:
    image: prom/node-exporter:v1.1.2
    container_name: node_exporter
    user: root
    privileged: true
    network_mode: "host"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|host|etc)($$|/)'
    restart: unless-stopped
    cpu_shares: 14
    mem_limit: 50m
    logging:
      driver: "json-file"
      options:
        max-size: "200M"
    labels:
      - "SERVICE_TAGS=prometheus"
```

prometheus.yaml配置文件

配置文件存放在compose挂载的位置

- 10.110.112.29:6443 k8s地址
-  /etc/token  授权得到的token

```
global:
  scrape_interval:     15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
  - scheme: http
    static_configs:
    - targets:
      - '127.0.0.1:9093'
      rule_files:
  - "alert/host.alert.rules"
  - "alert/container.alert.rules"
  - "alert/targets.alert.rules"
scrape_configs:
  - job_name: prometheus
    scrape_interval: 30s
    static_configs:
    - targets: ['127.0.0.1:9090']
    - targets: ['127.0.0.1:9093']

  - job_name: hn_node_exporter
    scrape_interval: 30s
    static_configs:
    - targets: ['127.0.0.1:9100']
    - targets: ['10.110.112.35:9182']
  - job_name: hn_mongodb_exporter
    scrape_interval: 30s
    static_configs:
    - targets: ['10.110.112.22:9216']
    - targets: ['10.110.112.23:9216']
    - targets: ['10.110.112.24:9216']

  - job_name: hn_kafka_exporter
    scrape_interval: 30s
    static_configs:
    - targets: ['10.110.112.25:9308']
    - targets: ['10.110.112.26:9308']
    - targets: ['10.110.112.27:9308']

  - job_name: hn_mysql_exporter
    scrape_interval: 30s
    static_configs:
    - targets: ['10.110.112.20:9104']
    - targets: ['10.110.112.21:9104']

  - job_name: hn_nginx_exporter
    scrape_interval: 30s
    static_configs:
    - targets: ['10.110.112.32:9113']
  # kubelet
  - job_name: "kube_node_kubelet"
    scheme: https
    tls_config:
      insecure_skip_verify: true
    bearer_token_file: /etc/token
    kubernetes_sd_configs:
    - role: node
      api_server: "https://10.110.112.29:6443"
      tls_config:
        insecure_skip_verify: true
      bearer_token_file: /etc/token
      relabel_configs:
    - target_label: __address__
      # 使用replacement值替换__address__默认值
      replacement: 10.110.112.29:6443
    - source_labels: [__meta_kubernetes_node_name]
      regex: (.+)
      # 使用replacement值替换__metrics_path__默认值
      target_label: __metrics_path__
      replacement: /api/v1/nodes/${1}:10250/proxy/metrics
    - action: labelmap
      regex: __meta_kubernetes_service_label_(.+)
    - source_labels: [__meta_kubernetes_namespace]
      action: replace
      target_label: kubernetes_namespace
    - source_labels: [__meta_kubernetes_service_name]
      action: replace
      target_label: service_name

  # advisor
  - job_name: "kube_node_cadvisor"
    scheme: https
    tls_config:
      insecure_skip_verify: true
    bearer_token_file: /etc/token
    kubernetes_sd_configs:
    - role: node
      api_server: "https://10.110.112.29:6443"
      tls_config:
        insecure_skip_verify: true
      bearer_token_file: /etc/token
      relabel_configs:
    - target_label: __address__
      # 使用replacement值替换__address__默认值
      replacement: 10.110.112.29:6443
    - source_labels: [__meta_kubernetes_node_name]
      regex: (.+)
      # 使用replacement值替换__metrics_path__默认值
      target_label: __metrics_path__
      replacement: /api/v1/nodes/${1}:10250/proxy/metrics/cadvisor
    - action: labelmap
      regex: __meta_kubernetes_service_label_(.+)
    - source_labels: [__meta_kubernetes_namespace]
      action: replace
      target_label: kubernetes_namespace
    - source_labels: [__meta_kubernetes_service_name]
      action: replace
      target_label: service_name

  - job_name: "kube_state_metrics"
    scheme: https
    tls_config:
      insecure_skip_verify: true
    bearer_token_file: /etc/token
    kubernetes_sd_configs:
    - role: endpoints
      api_server: "https://10.110.112.29:6443"
      tls_config:
        insecure_skip_verify: true
      bearer_token_file: /etc/token
      relabel_configs:
    - source_labels: [__meta_kubernetes_service_name]
      action: keep
      regex: '^(kube-state-metrics)$'
    - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
      action: keep
      regex: true
    - source_labels: [__address__]
      action: replace
      target_label: instance
    - target_label: __address__
      # 使用replacement值替换__address__默认值
      replacement: 10.110.112.29:6443
    - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_pod_name, __meta_kubernetes_pod_container_port_number]
      regex: ([^;]+);([^;]+);([^;]+)
      # 使用replacement值替换__metrics_path__默认值
      target_label: __metrics_path__
      replacement: /api/v1/namespaces/${1}/pods/http:${2}:${3}/proxy/metrics
    - action: labelmap
      regex: __meta_kubernetes_service_label_(.+)
    - source_labels: [__meta_kubernetes_namespace]
      action: replace
      target_label: kubernetes_namespace
    - source_labels: [__meta_kubernetes_service_name]
      action: replace
      target_label: service_name

  - job_name: "kube-state-metrics"
    scheme: https
    tls_config:
      insecure_skip_verify: true
    #使用apiserver授权部分解密的token值，以文件形式存储
    bearer_token_file: /etc/token
    # k8s自动发现具体配置
    kubernetes_sd_configs:
    # 使用endpoint级别自动发现
    - role: endpoints
      api_server: "https://10.110.112.29:6443"
      tls_config:
        insecure_skip_verify: true
      bearer_token_file: /etc/token
      relabel_configs:
    - source_labels: [__meta_kubernetes_service_name]
      # 只保留指定匹配正则的标签，不匹配则删除
      action: keep
      regex: '^(kube-state-metrics)$'
    - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
      # 只保留指定匹配正则的标签，不匹配则删除
      action: keep
      regex: true
    - source_labels: [__address__]
      action: replace
      target_label: instance
    - target_label: __address__
      # 使用replacement值替换__address__默认值
      replacement: 10.110.112.29:6443
    - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_pod_name, __meta_kubernetes_pod_container_port_number]
      # 正则匹配
      regex: ([^;]+);([^;]+);([^;]+)
      # 使用replacement值替换__metrics_path__默认值
      target_label: __metrics_path__
      # 自行构建的apiserver proxy url
      replacement: /api/v1/namespaces/${1}/pods/http:${2}:${3}/proxy/metrics
    - action: labelmap
      regex: __meta_kubernetes_service_label_(.+)
    - source_labels: [__meta_kubernetes_namespace]
      action: replace
      # 将标签__meta_kubernetes_namespace修改为kubernetes_namespace
      target_label: kubernetes_namespace
    - source_labels: [__meta_kubernetes_service_name]
      action: replace
      # 将标签__meta_kubernetes_service_name修改为service_name
      target_label: service_name

  - job_name: "kubernetes-apiservers-cluster-hainan"
    kubernetes_sd_configs:
      - role: endpoints
        api_server: "https://10.110.112.29:6443"
        tls_config:
          insecure_skip_verify: true
        bearer_token_file: /etc/token
        tls_config:
        insecure_skip_verify: true
        bearer_token_file: /etc/token
        scheme: https
        relabel_configs:
      - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
        action: keep
        regex: default;kubernetes;https
      - target_label: __address__
        replacement: 10.110.112.29:6443
```

此时可以发现

![image-20210723144115515](C:\Users\super\AppData\Roaming\Typora\typora-user-images\image-20210723144115515.png)

而后开始配置alertmanager

### 19.3 alertmanager

alertmanager配置两个webhook分别接收两个不同的警报

- webhook2 ： warning
- webhookcritical ： critical

cat alertmanager/config.yml

```
global:
  resolve_timeout: 5m
route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 45m
  receiver: 'webhook2'
  routes:
  - receiver: 'webhookcritical'
    match:
      severity: 'critical'
  - receiver: 'webhook2'
    match:
      severity: '~(warning)$'
    receivers:
- name: 'webhook2'
  webhook_configs:
  - send_resolved: true
    url: 'http://127.0.0.1:8061/dingtalk/webhook/send'
- name: 'webhookcritical'
  webhook_configs:
  - send_resolved: true
    url: 'http://127.0.0.1:8060/dingtalk/webhook/send'
    inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

#### 19.3.1 报警模板

创建模板文件

cat /data/prom/alertmanager/my.tepl

```
{{ define "__subject" }}[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}] {{ .GroupLabels.SortedPairs.Values | join " " }} {{ if gt (len .CommonLabels) (len .GroupLabels) }}({{ with .CommonLabels.Remove .GroupLabels.Names }}{{ .Values | join " " }}{{ end }}){{ end }}{{ end }}
{{ define "__alertmanagerURL" }}{{ .ExternalURL }}/#/alerts?receiver={{ .Receiver }}{{ end }}

{{ define "__text_alert_list" }}{{ range . }}
**Labels**
{{ range .Labels.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}
{{ end }}
**Annotations**
{{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}
{{ end }}
**Source:** [{{ .GeneratorURL }}]({{ .GeneratorURL }})
{{ end }}{{ end }}

{{ define "default.__text_alert_list" }}{{ range . }}
---
**告警级别:** {{ .Labels.severity | upper }}

**触发时间:** {{ dateInZone "2006.01.02 15:04:05" (.StartsAt) "Asia/Shanghai" }}

**事件信息:**
{{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}


{{ end }}

**事件标签:**
{{ range .Labels.SortedPairs }}{{ if and (ne (.Name) "severity") (ne (.Name) "summary") (ne (.Name) "team") }}> - {{ .Name }}: {{ .Value | markdown | html }}
{{ end }}{{ end }}
{{ end }}
{{ end }}
{{ define "default.__text_alertresovle_list" }}{{ range . }}
---
**告警级别:** {{ .Labels.severity | upper }}

**触发时间:** {{ dateInZone "2006.01.02 15:04:05" (.StartsAt) "Asia/Shanghai" }}

**结束时间:** {{ dateInZone "2006.01.02 15:04:05" (.EndsAt) "Asia/Shanghai" }}

**事件信息:**
{{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}


{{ end }}

**事件标签:**
{{ range .Labels.SortedPairs }}{{ if and (ne (.Name) "severity") (ne (.Name) "summary") (ne (.Name) "team") }}> - {{ .Name }}: {{ .Value | markdown | html }}
{{ end }}{{ end }}
{{ end }}
{{ end }}

{{/* Default */}}
{{ define "default.title" }}{{ template "__subject" . }}{{ end }}
{{ define "default.content" }}#### \[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}\] **[{{ index .GroupLabels "alertname" }}]({{ template"__alertmanagerURL" . }})**
{{ if gt (len .Alerts.Firing) 0 -}}

    ![20210716143611.png](https://pic5.58cdn.com.cn/nowater/webim/big/n_v27ab5f3b496a2419ba8ae8b3cf27960f0.png)

**====⚠️⚠️⚠️trigger alarm====**

{{ template "default.__text_alert_list" .Alerts.Firing }}


{{- end }}

{{ if gt (len .Alerts.Resolved) 0 -}}
**====[烟花]recover alarm====**
{{ template "default.__text_alertresovle_list" .Alerts.Resolved }}


{{- end }}
{{- end }}

{{/* Legacy */}}
{{ define "legacy.title" }}{{ template "__subject" . }}{{ end }}
{{ define "legacy.content" }}#### \[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}\] **[{{ index .GroupLabels "alertname" }}]({{ template "__alertmanagerURL" . }})**
{{ template "__text_alert_list" .Alerts.Firing }}
{{- end }}

{{/* Following names for compatibility */}}
{{ define "ding.link.title" }}{{ template "default.title" . }}{{ end }}
{{ define "ding.link.content" }}{{ template "default.content" . }}{{ end }}
```

### 19.4 alert

配置触发器

cat /data/prom/prometheus/alert/host.alert.rules

```
groups:
  - name: external_node_alarm
    rules:
    - alert: node_host_lost
      expr: up{job="hn_node_exporter"} == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "{{$labels.instance}}:service is down"
        description: "{{$labels.instance}}: lost contact for 1 minutes"
    - alert: HostOutOfMemory
      expr: node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100 < 5
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: (instance {{ $labels.instance }})实例内存不足
        description: "节点内存已用完 (剩余小于5%)已持续2分钟\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostOutOfMemory
      expr: node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100 < 2
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: (instance {{ $labels.instance }})实例内存不足
        description: "节点内存已用完 (剩余小于2%)已持续2分钟\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostMemoryUnderMemoryPressure
      expr: rate(node_vmstat_pgmajfault[1m]) > 1000
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: (instance {{ $labels.instance }})实例内存压力很大
        description: "节点内存压力很大major page错误率高已持续2分钟\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostUnusualNetworkThroughputIn
      expr: sum by (instance) (rate(node_network_receive_bytes_total[2m])) / 1024 / 1024 > 100
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: (instance {{ $labels.instance }})实例出现异常的input网络吞吐量
        description: "{{ $labels.instance }}网络接口接收超过100MB/s的数据已持续5分钟 \n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostUnusualNetworkThroughputOut
      expr: sum by (instance) (rate(node_network_transmit_bytes_total[2m])) / 1024 / 1024 > 100
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: (instance {{ $labels.instance }})实例出现异常的output网络吞吐量
        description: "{{ $labels.instance }}实例网络发送了超过100MB/s的数据已持续5分钟 (> 100 MB/s)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostUnusualDiskReadRate
      expr: sum by (instance) (rate(node_disk_read_bytes_total[2m])) / 1024 / 1024 > 50
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: (instance {{ $labels.instance }})实例磁盘读取异常
        description: "{{ $labels.instance }}实例磁盘读取超过>50MB/s已持续5分钟 \n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostUnusualDiskWriteRate
      expr: sum by (instance) (rate(node_disk_written_bytes_total[2m])) / 1024 / 1024 > 50
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: (instance {{ $labels.instance }})实例磁盘写入异常
        description: "{{ $labels.instance }}实例磁盘写入超过>50MB/s已持续两分钟\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostUnusualDiskWriteRate
      expr: sum by (instance) (rate(node_disk_written_bytes_total[2m])) / 1024 / 1024 > 15
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: (instance {{ $labels.instance }})实例磁盘写入异常
        description: "{{ $labels.instance }}实例磁盘写入超过>15MB/s已持续两分钟 \n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    # Please add ignored mountpoints in node_exporter parameters like
    # "--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|run)($|/)".
    # Same rule using "node_filesystem_free_bytes" will fire when disk fills for non-root users.
    - alert: HostOutOfDiskSpace
      expr: (node_filesystem_avail_bytes * 100) / node_filesystem_size_bytes < 10 and ON (instance, device, mountpoint) node_filesystem_readonly == 0
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: (instance {{ $labels.instance }})实例磁盘空间不足
        description: "{{ $labels.instance }}磁盘快满了,剩余不足10%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    # Please add ignored mountpoints in node_exporter parameters like
    # "--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|run)($|/)".
    # Same rule using "node_filesystem_free_bytes" will fire when disk fills for non-root users.
    - alert: HostOutOfDiskSpace
      expr: (node_filesystem_avail_bytes * 100) / node_filesystem_size_bytes < 5 and ON (instance, device, mountpoint) node_filesystem_readonly == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: (instance {{ $labels.instance }})实例磁盘空间不足
        description: "{{ $labels.instance }}磁盘快满了,剩余不足5%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostDiskWillFillIn24Hours
      expr: (node_filesystem_avail_bytes * 100) / node_filesystem_size_bytes < 10 and ON (instance, device, mountpoint) predict_linear(node_filesystem_avail_bytes{fstype!~"tmpfs"}[1h], 24 * 3600) < 0 and ON (instance, device, mountpoint) node_filesystem_readonly == 0
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: (instance {{ $labels.instance }})在24小时内就会写满
        description: "{{ $labels.instance }}在当前的写入速率下,预计文件系统将会在24小时内耗尽磁盘空间\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostOutOfInodes
      expr: node_filesystem_files_free{mountpoint ="/rootfs"} / node_filesystem_files{mountpoint="/rootfs"} * 100 < 10 and ON (instance, device, mountpoint) node_filesystem_readonly{mountpoint="/rootfs"} == 0
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: (instance {{ $labels.instance }})实例inodes不足
        description: "{{ $labels.instance }}磁盘几乎用完了inodes,剩余不足10%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostInodesWillFillIn24Hours
      expr: node_filesystem_files_free{mountpoint ="/rootfs"} / node_filesystem_files{mountpoint="/rootfs"} * 100 < 10 and predict_linear(node_filesystem_files_free{mountpoint="/rootfs"}[1h], 24 *3600) < 0 and ON (instance, device, mountpoint) node_filesystem_readonly{mountpoint="/rootfs"} == 0
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Host inodes will fill in 24 hours (instance {{ $labels.instance }})
        description: "{{ $labels.instance }}以当前写入速率,文件系统预计将在未来 24小时内耗尽inode\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostUnusualDiskReadLatency
      expr: rate(node_disk_read_time_seconds_total[1m]) / rate(node_disk_reads_completed_total[1m]) > 0.1 and rate(node_disk_reads_completed_total[1m]) > 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: (instance {{ $labels.instance }})实例磁盘读取延迟异常
        description: "磁盘读取延迟正在增长,当下大于100ms已持续1分钟\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostUnusualDiskWriteLatency
      expr: rate(node_disk_write_time_seconds_total[1m]) / rate(node_disk_writes_completed_total[1m]) > 0.1 and rate(node_disk_writes_completed_total[1m]) > 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: (instance {{ $labels.instance }})实例磁盘写入延迟异常
        description: "磁盘写入延迟正在增长,当下大于100ms已持续1分钟\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostHighCpuLoad
      expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100) > 98
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: (instance {{ $labels.instance }})实例CPU负载过高
        description: "当前CPU load高于98%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostCpuStealNoisyNeighbor
      expr: avg by(instance) (rate(node_cpu_seconds_total{mode="steal"}[5m])) * 100 > 10
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: (instance {{ $labels.instance }})CPU steal time开始增长
        description: "CPU steal time大于10%已持续5分钟. 这意味着CPU资源不足或vm资源竞争。当前实例计算能力将开始下降.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    # 1000 context switches is an arbitrary number.
    # Alert threshold depends on nature of application.
    # Please read: https://github.com/samber/awesome-prometheus-alerts/issues/58
    - alert: HostContextSwitching
      expr: (rate(node_context_switches_total[5m])) / (count without(cpu, mode) (node_cpu_seconds_total{mode="idle"})) > 3000
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: (instance {{ $labels.instance }})主机上下文切换异常
        description: "实例上下文切换正在增长,当下超过3000/s)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostSwapIsFillingUp
      expr: (1 - (node_memory_SwapFree_bytes / node_memory_SwapTotal_bytes)) * 100 > 80
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: (instance {{ $labels.instance }})swap内存即将用完
        description: "Swap使用超过80% \n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostSystemdServiceCrashed
      expr: node_systemd_unit_state{state="failed"} == 1
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: (instance {{ $labels.instance }})当前实例systemd service崩溃
        description: "当前实例systemd service崩溃\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostKernelVersionDeviations
      expr: count(sum(label_replace(node_uname_info, "kernel", "$1", "release", "([0-9]+.[0-9]+.[0-9]+).*")) by (kernel)) > 1
      for: 6h
      labels:
        severity: warning
      annotations:
        summary: (instance {{ $labels.instance }})实例内核版本偏差
        description: "正在运行的内核版本:\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostOomKillDetected
      expr: increase(node_vmstat_oom_kill[1m]) > 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: (instance {{ $labels.instance }})实例发生OOM kill事件
        description: "检测到OOM kill\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostNetworkReceiveErrors
      expr: rate(node_network_receive_errs_total[2m]) / rate(node_network_receive_packets_total[2m]) > 0.01
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: (instance {{ $labels.instance }})主机网络接收错误
        description: "主机 {{ $labels.instance }} interface {{ $labels.device }} 在过去五分钟内遇到了 {{ printf \"%.0f\" $value }} 接收错误。\n VALUE = { { $value }}\n 标签 = {{ $labels }}"
    - alert: HostNetworkTransmitErrors
      expr: rate(node_network_transmit_errs_total[2m]) / rate(node_network_transmit_packets_total[2m]) > 0.01
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: (instance {{ $labels.instance }})主机网络传输错误
        description: "主机 {{ $labels.instance }} interface {{ $labels.device }} 在过去五分钟内遇到了 {{ printf \"%.0f\" $value }} 传输错误。\n VALUE = { { $value }}\n 标签 = {{ $labels }}"
    - alert: HostNetworkInterfaceSaturated
      expr: (rate(node_network_receive_bytes_total{device!~"^tap.*"}[1m]) + rate(node_network_transmit_bytes_total{device!~"^tap.*"}[1m])) / node_network_speed_bytes{device!~"^tap.*"} > 0.8
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: (instance {{ $labels.instance }})网络接口接近饱和
        description: "\"{{ $labels.instance }}\" 上的网络接口 \"{{ $labels.interface }}\" 正在过载。\n VALUE = {{ $value }}\n LABELS = { { $labels }}"
    - alert: HostConntrackLimit
      expr: node_nf_conntrack_entries / node_nf_conntrack_entries_limit > 0.8
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: 主机 conntrack 限制(instance {{ $labels.instance }})
        description: "conntrack 数量接近限制\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
    - alert: HostClockSkew
      expr: (node_timex_offset_seconds > 0.05 and deriv(node_timex_offset_seconds[5m]) >= 0) or (node_timex_offset_seconds < -0.05 and deriv(node_timex_offset_seconds[5m]) <= 0)
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: 主机时钟偏差(instance {{ $labels.instance }})
        description: "检测到时钟偏差。时钟不同步。\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
    - alert: HostClockNotSynchronising
      expr: min_over_time(node_timex_sync_status[1m]) == 0 and node_timex_maxerror_seconds >= 16
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: 主机时钟不同步(instance {{ $labels.instance }})
        description: "时钟不同步.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostEdacCorrectableErrorsDetected
      expr: increase(node_edac_correctable_errors_total[1m]) > 0
      for: 0m
      labels:
        severity: info
      annotations:
        summary: Host EDAC Correctable Errors detected (instance {{ $labels.instance }})
        description: "Host {{ $labels.instance }} has had {{ printf \"%.0f\" $value }} correctable memory errors reported by EDAC in the last 5 minutes.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostEdacUncorrectableErrorsDetected
      expr: node_edac_uncorrectable_errors_total > 0
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Host EDAC Uncorrectable Errors detected (instance {{ $labels.instance }})
        description: "Host {{ $labels.instance }} has had {{ printf \"%.0f\" $value }} uncorrectable memory errors reported by EDAC in the last 5 minutes.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostPhysicalComponentTooHot
      expr: node_hwmon_temp_celsius > 75
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: (instance {{ $labels.instance }})宿主机组件过热
        description: "物理硬件过热\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostNodeOvertemperatureAlarm
      expr: node_hwmon_temp_crit_alarm_celsius == 1
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: 物理节点超温告警(instance {{ $labels.instance }})
        description: "物理节点超温告警\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostRaidArrayGotInactive
      expr: node_md_state{state="inactive"} > 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary:  RAID阵列处于非活动状态(instance {{ $labels.instance }})
        description: "RAID array {{ $labels.device }} is in degraded state due to one or more disks failures. Number of spare drives is insufficient to fix issue automatically.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: HostRaidDiskFailure
      expr: node_md_disks{state="failed"} > 0
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: RAID磁盘故障(instance {{ $labels.instance }})
        description: "At least one device in RAID array on {{ $labels.instance }} failed. Array {{ $labels.md_device }} needs attention and possibly a disk swap\n  VALUE = {{ $value }}\n  LABELS ={{ $labels }}"
  - name: MySQLStatsAlert
    rules:
    - alert: MySQL is down
      expr: mysql_up == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "{{ $labels.instance }} MySQL is down"
        description: "MySQL database is down. This requires immediate action!"
    - alert: MysqlTooManyConnections(>80%)
      expr: avg by (instance) (rate(mysql_global_status_threads_connected[1m])) / avg by (instance) (mysql_global_variables_max_connections) * 100 > 80
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: MySQL too many connections (> 80%) (instance {{ $labels.instance }})
        description: "超过 80% 的 MySQL 连接在 {{ $labels.instance }} 上使用\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
    - alert: MysqlHighThreadsRunning
      expr: avg by (instance) (rate(mysql_global_status_threads_running[1m])) / avg by (instance) (mysql_global_variables_max_connections) * 100 > 60
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: MySQL high threads running (instance {{ $labels.instance }})
        description: "超过 60% 的 MySQL 连接在 {{ $labels.instance }} 上处于运行状态\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
    - alert: MysqlRestarted
      expr: mysql_global_status_uptime < 60
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: MySQL重新启动 (instance {{ $labels.instance }})
        description: "MySQL 刚刚重新启动，不到一分钟前在 {{ $labels.instance }} 上。\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
  - name: external_kafka_alarm
    rules:
    - alert: KafkaTopicsReplicas
      expr: sum(kafka_topic_partition_in_sync_replica) by (topic) < 3
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Kafka topics replicas (instance {{ $labels.instance }})
        description: "Kafka 副本分区\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
    - alert: KafkaConsumersGroup
      expr: sum(kafka_consumergroup_lag) by (consumergroup) > 50
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: Kafka consumers group (instance {{ $labels.instance }})
        description: "Kafka 消费者组\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
    - alert: KafkaTopicOffsetDecreased
      expr: delta(kafka_burrow_partition_current_offset[1m]) < 0
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Kafka topic offset decreased (instance {{ $labels.instance }})
        description: "Kafka 主题偏移量已减少\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
    - alert: KafkaConsumerLag
      expr: 'kafka_burrow_topic_partition_offset - on(partition, cluster, topic) group_right() kafka_burrow_partition_current_offset >= (kafka_burrow_topic_partition_offset offset 15m - on(partition, cluster, topic) group_right() kafka_burrow_partition_current_offset offset 15m)
      AND kafka_burrow_topic_partition_offset - on(partition, cluster, topic) group_right() kafka_burrow_partition_current_offset > 0'
      for: 15m
      labels:
        severity: critical
      annotations:
        summary: Kafka consumer lag (instance {{ $labels.instance }})
        description: "卡夫卡消费者有 30 分钟的延迟并且越来越多\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
  - name: external_mongodb_alarm
    rules:
    - alert: MongodbDown
      expr: mongodb_up == 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: MongoDB Down (instance {{ $labels.instance }})
        description: "MongoDB 实例已关闭\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
    - alert: MongodbReplicationLag
      #expr: mongodb_mongod_replset_member_optime_date{state="PRIMARY"} - ON (set) mongodb_mongod_replset_member_optime_date{state="SECONDARY"} > 10
      expr: avg(mongodb_mongod_replset_member_optime_date{state="PRIMARY"})-avg(mongodb_mongod_replset_member_optime_date{state="SECONDARY"}) > 10
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: MongoDB replication lag (instance {{ $labels.instance }})
        description: "Mongodb 复制延迟超过 10 秒\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
    - alert: MongodbNumberCursorsOpen
      expr: mongodb_mongod_metrics_cursor_open{state="total"} > 10 * 1000
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: MongoDB number cursors open (instance {{ $labels.instance }})
        description: "MongoDB 为客户端打开了太多cursors （> 10k）\n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
    - alert: MongodbCursorsTimeouts
      expr: increase(mongodb_mongod_metrics_cursor_timed_out_total[1m]) > 100
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: MongoDB cursors timeouts (instance {{ $labels.instance }})
        description: "太多的cursors 超时\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: MongodbTooManyConnections
      expr: avg by(instance) (rate(mongodb_connections{state="current"}[1m])) / avg by(instance) (sum (mongodb_connections) by (instance)) * 100 > 80
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: MongoDB too many connections (instance {{ $labels.instance }})
        description: "连接过多 (> 80%)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
  - name: external_etcd_alarm
    rules:
    - alert: EtcdInsufficientMembers
      expr: count(etcd_server_id) % 2 == 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: etcd cluster 已崩溃 (instance {{ $labels.instance }})
        description: "Etcd 集群应该有奇数个成员, \n VALUE = {{ $value }}\n LABELS = {{ $labels }}"
    - alert: EtcdNoLeader
      expr: etcd_server_has_leader == 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Etcd no Leader (instance {{ $labels.instance }})
        description: "Etcd cluster have no leader\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: EtcdHighNumberOfLeaderChanges
      expr: increase(etcd_server_leader_changes_seen_total[10m]) > 2
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Etcd leader 切换异常 (instance {{ $labels.instance }})
        description: "10分钟内leader切换了两次\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: EtcdHighNumberOfFailedGrpcRequests
      expr: sum(rate(grpc_server_handled_total{grpc_code!="OK"}[1m])) BY (grpc_service, grpc_method) / sum(rate(grpc_server_handled_total[1m])) BY (grpc_service, grpc_method) > 0.01
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Etcd 大量失败的 GRPC 请求 (instance {{ $labels.instance }})
        description: "在 Etcd 中检测到超过 1% 的 GRPC 请求失败\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: EtcdHighNumberOfFailedGrpcRequests
      expr: sum(rate(grpc_server_handled_total{grpc_code!="OK"}[1m])) BY (grpc_service, grpc_method) / sum(rate(grpc_server_handled_total[1m])) BY (grpc_service, grpc_method) > 0.05
      for: 2m
      labels:
        severity: critical
      annotations:
        summary:  Etcd 大量失败的 GRPC 请求  (instance {{ $labels.instance }})
        description: "在 Etcd 中检测到超过 5% 的 GRPC 请求失败\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: EtcdGrpcRequestsSlow
      expr: histogram_quantile(0.99, sum(rate(grpc_server_handling_seconds_bucket{grpc_type="unary"}[1m])) by (grpc_service, grpc_method, le)) > 0.15
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Etcd GRPC 请求缓慢(instance {{ $labels.instance }})
        description: "GRPC 请求变慢，99% 超过 0.15s\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: EtcdHighNumberOfFailedHttpRequests
      expr: sum(rate(etcd_http_failed_total[1m])) BY (method) / sum(rate(etcd_http_received_total[1m])) BY (method) > 0.01
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Etcd 大量失败的 HTTP 请求 (instance {{ $labels.instance }})
        description: "在 Etcd 中检测到超过 1% 的 HTTP 失败\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: EtcdHighNumberOfFailedHttpRequests
      expr: sum(rate(etcd_http_failed_total[1m])) BY (method) / sum(rate(etcd_http_received_total[1m])) BY (method) > 0.05
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: Etcd 大量失败的 HTTP 请求  (instance {{ $labels.instance }})
        description: "在 Etcd 中检测到超过 5% 的 HTTP 失败\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: EtcdHttpRequestsSlow
      expr: histogram_quantile(0.99, rate(etcd_http_successful_duration_seconds_bucket[1m])) > 0.15
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Etcd HTTP 请求缓慢 (instance {{ $labels.instance }})
        description: "Etcd HTTP 请求变慢，99% 超过 0.15s\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: EtcdMemberCommunicationSlow
      expr: histogram_quantile(0.99, rate(etcd_network_peer_round_trip_time_seconds_bucket[1m])) > 0.15
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Etcd成员通讯缓慢 (instance {{ $labels.instance }})
        description: "Etcd 成员通信变慢，99% 超过 0.15s\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: EtcdHighNumberOfFailedProposals
      expr: increase(etcd_server_proposals_failed_total[1h]) > 5
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Etcd 大量失败的proposals  (instance {{ $labels.instance }})
        description: "Etcd 服务器在过去一小时收到了超过 5 个失败的proposals\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: EtcdHighFsyncDurations
      expr: histogram_quantile(0.99, rate(etcd_disk_wal_fsync_duration_seconds_bucket[1m])) > 0.5
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Etcd fsync 持续时间变高 (instance {{ $labels.instance }})
        description: "Etcd WAL fsync 持续时间增加，99% 超过 0.5s\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: EtcdHighCommitDurations
      expr: histogram_quantile(0.99, rate(etcd_disk_backend_commit_duration_seconds_bucket[1m])) > 0.25
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Etcd 提交持续时间较高 (instance {{ $labels.instance }})
        description: "Etcd 提交持续时间增加，99% 超过 0.25s\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### 19.5 配置钉钉alertmanager

alertmanager对接的两个容器

- 注意查看目录挂载位置

cat /data/prom/alertmanager.yaml

```
version: '2.2'
services:
  dingtalk8060:
    image: marksugar/k8s-prometheus:prometheus_dingtalk_v0.2
    container_name: dingtalk8060
    restart: always
    privileged: true
    network_mode: "host"
    command:
      - '--web.listen-address=:8060'
      - '--config.file=/etc/prometheus-webhook-dingtalk/config.yml'
    volumes:
    - /etc/localtime:/etc/localtime
    - /data/prom/alertmanager/dconfig.yml:/etc/prometheus-webhook-dingtalk/config.yml
    - /data/prom/alertmanager/my.tepl:/etc/prometheus-webhook-dingtalk/templates/default.tmpl
    cpu_shares: 60
    mem_limit: 2048m
    restart: unless-stopped
    logging:
      driver: "json-file"
      options:
        max-size: "200M"
    labels:
      SERVICE_TAGS: prometheus
  dingtalk8061:
    image: marksugar/k8s-prometheus:prometheus_dingtalk_v0.2
    container_name: dingtalk8061
    restart: always
    privileged: true
    network_mode: "host"
    command:
      - '--web.listen-address=:8061'
      - '--config.file=/etc/prometheus-webhook-dingtalk/config.yml'
    volumes:
    - /etc/localtime:/etc/localtime
    - /data/prom/alertmanager/dconfig1.yml:/etc/prometheus-webhook-dingtalk/config.yml
    - /data/prom/alertmanager/my.tepl:/etc/prometheus-webhook-dingtalk/templates/default.tmpl
    cpu_shares: 60
    mem_limit: 2048m
    restart: unless-stopped
    logging:
      driver: "json-file"
      options:
        max-size: "200M"
    labels:
      SERVICE_TAGS: prometheus
```

钉钉机器人token的配置文件
[root@yxthn34 prom]# cat alertmanager/dconfig1.yml

```
templates:
  - /etc/prometheus-webhook-dingtalk/templates/default.tmpl
    targets:
    webhook:
    url: https://oapi.dingtalk.com/robot/send?access_token=993838ca3db631db2a0232fe650a23a3243bcaa63a0
    secret: SECfc3985011f4d29d6f86edd200c2e1e7fa
    message:
      title: '{{ template "ding.link.title" . }}'
      text: '{{ template "ding.link.content" . }}'
```
[root@yxthn34 prom]# cat alertmanager/dconfig.yml

```
templates:
  - /etc/prometheus-webhook-dingtalk/templates/default.tmpl
    targets:
    webhook:
    url: https://oapi.dingtalk.com/robot/send?access_token=b988c8a14fd08e2ef13109b828ac6eacf6de0accc236
    secret: SEC97948b57727c732a3568502047e8536
    message:
      title: '{{ template "ding.link.title" . }}'
      text: '{{ template "ding.link.content" . }}'
```



#### 19.5.1 grafana模板

- Cadvisor: 14282
- node: 8919

## 20.ingress

```
apiVersion: v1
kind: Namespace
metadata:
  name: ingress-nginx
  labels:
    app: ingress-nginx

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-controller
  namespace: ingress-nginx
  labels:
    app: ingress-nginx

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: nginx-ingress-controller
  labels:
    app: ingress-nginx
rules:
  - apiGroups:
      - ""
      resources:
      - configmaps
      - endpoints
      - nodes
      - pods
      - secrets
      - namespaces
      - services
      verbs:
      - get
      - list
      - watch
  - apiGroups:
      - "extensions"
      - "networking.k8s.io"
      resources:
      - ingresses
      verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
      resources:
      - events
      verbs:
      - create
      - patch
  - apiGroups:
      - "extensions"
      - "networking.k8s.io"
      resources:
      - ingresses/status
      verbs:
      - update
  - apiGroups:
      - ""
      resources:
      - configmaps
      verbs:
      - create
  - apiGroups:
      - ""
      resources:
      - configmaps
      resourceNames:
      - "ingress-controller-leader-nginx"
      verbs:
      - get
      - update

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: nginx-ingress-controller
  labels:
    app: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: nginx-ingress-controller
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-controller
    namespace: ingress-nginx

---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: ingress-nginx
  name: nginx-ingress-lb
  namespace: ingress-nginx
spec:
  # DaemonSet need:
  # ----------------
  type: ClusterIP
  # ----------------
  # Deployment need:
  # ----------------
  #  type: NodePort
  # ----------------
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
  - name: https
    port: 443
    targetPort: 443
    protocol: TCP
  - name: metrics
    port: 10254
    protocol: TCP
    targetPort: 10254
    selector:
    app: ingress-nginx

---
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
  namespace: ingress-nginx
  labels:
    app: ingress-nginx
data:
  keep-alive: "75"
  keep-alive-requests: "100"
  upstream-keepalive-connections: "10000"
  upstream-keepalive-requests: "100"
  upstream-keepalive-timeout: "60"
  allow-backend-server-header: "true"
  enable-underscores-in-headers: "true"
  generate-request-id: "true"
  http-redirect-code: "301"
  ignore-invalid-headers: "true"
  log-format-upstream: '{"@timestamp": "$time_iso8601","remote_addr": "$remote_addr","x-forward-for": "$proxy_add_x_forwarded_for","request_id": "$req_id","remote_user": "$remote_user","bytes_sent": $bytes_sent,"request_time": $request_time,"status": $status,"vhost": "$host","request_proto": "$server_protocol","path": "$uri","request_query": "$args","request_length": $request_length,"duration": $request_time,"method": "$request_method","http_referrer": "$http_referer","http_user_agent":  "$http_user_agent","upstream-sever":"$proxy_upstream_name","proxy_alternative_upstream_name":"$proxy_alternative_upstream_name","upstream_addr":"$upstream_addr","upstream_response_length":$upstream_response_length,"upstream_response_time":$upstream_response_time,"upstream_status":$upstream_status}'
  max-worker-connections: "65536"
  worker-processes: "2"
  proxy-body-size: 20m
  proxy-connect-timeout: "10"
  proxy_next_upstream: error timeout http_502
  reuse-port: "true"
  server-tokens: "false"
  ssl-ciphers: ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA
  ssl-protocols: TLSv1 TLSv1.1 TLSv1.2
  ssl-redirect: "false"
  worker-cpu-affinity: auto

---
kind: ConfigMap
apiVersion: v1
metadata:
  name: tcp-services
  namespace: ingress-nginx
  labels:
    app: ingress-nginx

---
kind: ConfigMap
apiVersion: v1
metadata:
  name: udp-services
  namespace: ingress-nginx
  labels:
    app: ingress-nginx

---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-ingress-controller
  namespace: ingress-nginx
  labels:
    app: ingress-nginx
  annotations:
    component.version: "v0.30.0"
    component.revision: "v1"
spec:
  # Deployment need:
  # ----------------
  #  replicas: 1
  # ----------------
  selector:
    matchLabels:
      app: ingress-nginx
  template:
    metadata:
      labels:
        app: ingress-nginx
      annotations:
        prometheus.io/port: "10254"
        prometheus.io/scrape: "true"
        scheduler.alpha.kubernetes.io/critical-pod: ""
    spec:
      # DaemonSet need:
      # ----------------
      hostNetwork: true
      # ----------------
      serviceAccountName: nginx-ingress-controller
      priorityClassName: system-node-critical
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - ingress-nginx
              topologyKey: kubernetes.io/hostname
            weight: 100
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: type
                operator: NotIn
                values:
                - virtual-kubelet
      containers:
        - name: nginx-ingress-controller
          image: registry.cn-beijing.aliyuncs.com/acs/aliyun-ingress-controller:v0.30.0.2-9597b3685-aliyun
          imagePullPolicy: IfNotPresent
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
            - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
            - --publish-service=$(POD_NAMESPACE)/nginx-ingress-lb
            - --annotations-prefix=nginx.ingress.kubernetes.io
            - --enable-dynamic-certificates=true
            - --v=2
          securityContext:
            allowPrivilegeEscalation: true
            capabilities:
              drop:
                - ALL
              add:
                - NET_BIND_SERVICE
            runAsUser: 101
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 10
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 10
#          resources:
#            limits:
#              cpu: "1"
#              memory: 2Gi
#            requests:
#              cpu: "1"
#              memory: 2Gi
          volumeMounts:
          - mountPath: /etc/localtime
            name: localtime
            readOnly: true
      volumes:
      - name: localtime
        hostPath:
          path: /etc/localtime
          type: File
      nodeSelector:
        mark/ingress-controller-ready: "true"
      tolerations:
      - operator: Exists
```

###20.1 打标签

```
# 我们现在来打标签
# kubectl label node 192.168.63.2 mark/ingress-controller-ready=true
node/192.168.63.2 labeled
# kubectl label node 192.168.63.3 mark/ingress-controller-ready=true
node/192.168.63.3 labeled
```

###20.2  alertmanager

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: alertmanager
  namespace: monitoring
spec:
  rules:
    - host: alertmanager.k8s.com
      http:
        paths:
          - backend:
              serviceName: alertmanager-main
              servicePort: 9093
            path: /
```

### 20.3 grafana

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: grafana
  namespace: monitoring
spec:
  rules:
    - host: grafana.k8s.com
      http:
        paths:
          - backend:
              serviceName: grafana
              servicePort: 3000
            path: /
```

### 20.4 prometheus

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: prometheus
  namespace: monitoring
spec:
  rules:
    - host: prometheus.k8s.com
      http:
        paths:
          - backend:
              serviceName: prometheus-k8s
              servicePort: 9090
            path: /
```

### 20.5 nginx代理

proxy.conf

```
proxy_connect_timeout 1000s;
proxy_send_timeout  2000;
proxy_read_timeout   2000;
proxy_buffer_size    128k;
proxy_buffers     4 256k;
proxy_busy_buffers_size 256k;
proxy_redirect     off;
proxy_hide_header  Vary;
proxy_set_header   Accept-Encoding '';
proxy_set_header   Host   $host;
proxy_set_header   Referer $http_referer;
proxy_set_header   Cookie $http_cookie;
proxy_set_header   X-Real-IP  $remote_addr;
proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
```

在将nginx代理到端口上

```
upstream k8s {
	server 192.168.63.2 max_fails=3 fail_timeout=1s weight=1;
	server 192.168.63.3 max_fails=3 fail_timeout=1s weight=1;	
}
server {
    listen 30090;
    server_name prometheus.k8s.com;
    #if ($scheme = 'http' ) { rewrite ^(.*)$ https://$host$1 permanent; }
    index index.html index.htm index.php default.html default.htm default.php;

    index index.html index.htm index.php default.html default.htm default.php;
    access_log  /data/logs/linuxea.access.log upstream2;
    
    location / {
      proxy_pass http://k8s;
      include proxy.conf;
    }
}
server {
    listen 30091;
    server_name alertmanager.k8s.com;
    #if ($scheme = 'http' ) { rewrite ^(.*)$ https://$host$1 permanent; }
    index index.html index.htm index.php default.html default.htm default.php;

    index index.html index.htm index.php default.html default.htm default.php;
    access_log  /data/logs/linuxea.access.log upstream2;
    
    location / {
      proxy_pass http://k8s;
      include proxy.conf;
    }
}
server {
    listen 30092;
    server_name grafana.k8s.com;
    #if ($scheme = 'http' ) { rewrite ^(.*)$ https://$host$1 permanent; }
    index index.html index.htm index.php default.html default.htm default.php;

    index index.html index.htm index.php default.html default.htm default.php;
    access_log  /data/logs/linuxea.access.log upstream2;
    
    location / {
      proxy_pass http://k8s;
      include proxy.conf;
    }
}
```

## 21.模板

微服务：https://grafana.com/grafana/dashboards/10341

## 22.钉钉分组

```
kubectl apply -f many.yaml
kustomize build  dingtalk-8060/ | kubectl -f -
kustomize build  dingtalk-8061/ | kubectl -f -
```

