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
  - [仪表盘](#仪表盘)
  - [警报](警报)
    - [配置钉钉][#dingtalk]
	- [配置微信][#wechat_config]


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
kubectl apply -f WeChat/
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
| master      | dingtalk            |         | 3005      |
| None      | wechat            |         |       |
## 仪表盘

提供了如下图

![top.png](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/top.png)

容器占用率
![cadvisor.png](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/cadvisor-brief.png)
![host-docker](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/docker.png)

kuberntes API
![api.png](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/api.png)
node资源
![node1.png](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/node1.png)
![node2.png](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/node2.png)
![host-node3](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/node3.png)
![host-node4](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/node4.png)
etcd
![etcd.png](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/etcd.png)
名称空间资源
![namespace.png](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/namespace.png)
## dingtalk
参考[prometheus-webhook-dingtalk](https://github.com/timonwong/prometheus-webhook-dingtalk)的[Prometheus AlertManager WebHook](Prometheus AlertManager WebHook)文章

修改dingtalk目录下的deploy文件中的
```
args:
- "--ding.profile=pgmon=https://oapi.dingtalk.com/robot/send?access_token=XXXXXXXXXXXXX"
- "--web.listen-address=:30005"
- "--log.level=info"
- "--ding.timeout=5s"
```		
![dingtalk.png](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/dingtalk.png)

## wechat_config

配置参考[prometheus官方的文档即可](https://prometheus.io/docs/alerting/configuration/#wechat_config)
- corp_id: 企业微信账号唯一 ID， 可以在我的企业中查看。
- to_party: 需要发送的组。
- agent_id: 第三方企业应用的 ID，可以在自己创建的第三方企业应用详情页面查看。
- api_secret: 第三方企业应用的密钥，可以在自己创建的第三方企业应用详情页面查看。

微信API详情请参考[文档](https://work.weixin.qq.com/api/doc#90000/90135/90236/%E6%96%87%E6%9C%AC%E6%B6%88%E6%81%AF)。

发送告警：
![wechat.png](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/wechat.png)
恢复：
![wechat-ok.png](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/wechat-ok.png)
