prometheus与grafana在kubernetes的配置文件，开箱即用！

kubernetes: 1.13.1
```
[root@master-0 ~]# kubectl version  --short 
Client Version: v1.13.1
Server Version: v1.13.1
```
## 目录

- [部署](#部署)
  - [版本](#版本信息)
  - [截图](#截图)



## 部署

需要修改一些配置，如etcd信息。

克隆
```
git clone https://github.com/marksugar/k8s-pgmon.git
cd $PWD/k8s-pgmon
```
开始配置
```
kubectl apply -f k8s-pgmom-namespace.yaml 
kubectl apply -f etcd-external/
kubectl apply -f prometheus/
kubectl apply -f node-export/
kubectl apply -f kube-controller-schedule/
kubectl apply -f kube-state-metrics/
kubectl apply -f grafana/
kubectl apply -f alertmanager/
```

## 版本信息

| Version     | type                | User ID | port      |
| ----------- | --------------------| ------- | --------- |
| marksugar/grafana:5.4.2-p      | grafana             | 472     | 3000      |
| v0.15.3     | alertmanager        |         | 9093/9094 |
| v2.5.0      | prometheus          | 665534  | 9090      |
| v0.16.0     | node_exporter       |         | 9100      |
| 1.8.4       | addon-resizer       |         | 18880     |
| v1.3.0      | kube-state-metrics  |         |       |
| latest      | blackbox-exporter   |         |       |
| 3.3.10      | etcd                |         | 2379      |

## 截图

提供了如下图

![1.png](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/1.png)

容器占用率
![2.png](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/2.png)
kuberntes API
![api.png](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/api.png)
主机与docker
![docker-host.png](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/docker-host.png)
![host.png](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/host.png)
![host-describe](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/host-describe.png)
etcd
![etcd.png](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/etcd.png)
名称空间资源
![namespace.png](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/namespace.png)
