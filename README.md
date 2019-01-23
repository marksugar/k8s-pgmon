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
    - [配置钉钉](#dingtalk)
	- [配置微信](#wechat_config)
- [配置说明]	
  - [etcd](#etcd)
  - [scheduler/controller](#scheduler/controller)
  - [alertmanager](#alertmanager)
  - [grafana](#grafana)
  - [prometheus](#prometheus)
	
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
kubectl apply -f node-export/
kubectl apply -f kube-controller-schedule/
kubectl apply -f kube-state-metrics/
kubectl apply -f grafana/
kubectl apply -f WeChat/
kubectl apply -f prometheus/
kubectl apply -f alertmanager/
```

你可以使用ip:端口来访问，这些分别是
```
k8s-pgmon-alertmanager       NodePort    IP  <none>        9093:30004/TCP
k8s-pgmon-grafana            NodePort    IP  <none>        3000:30001/TCP
k8s-pgmon-prometheus         NodePort    IP  <none>        9090:30002/TCP
node-exporter                NodePort    IP  <none>        9100:30003/TCP
```
grafana登陆用户:admin  密码:admin
![grafana.png](https://raw.githubusercontent.com/marksugar/k8s-pgmon/master/Dashboard/image/grafana.png)


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

如果你不想使用任何报警媒介，你可以使用默认的[配置文件](https://github.com/marksugar/k8s-pgmon/blob/master/alertmanager/alertmanager-configmap.yaml_default)

## etcd
如果你也是外部的etcd集群，你可以参考[etcd-external](https://github.com/marksugar/k8s-pgmon/tree/master/etcd-external)路径下的文件，在[etcd-ep](https://github.com/marksugar/k8s-pgmon/blob/master/etcd-external/etcd-ep.yaml)文件中修改IP地址，在[etcd-tls](https://github.com/marksugar/k8s-pgmon/blob/master/etcd-external/etcd-tls.yaml)中附上密钥即可完成添加。

## scheduler/controller
如果此刻你的集群内监听的端口仍然是127.0.0.1,那么如果需要，你可以进行修改,命令如下
```
sed -ri '/--address/s#=.+#=0.0.0.0#' /etc/kubernetes/manifests/kube-*
```
另外，在[kube-endpoints](https://github.com/marksugar/k8s-pgmon/blob/master/kube-controller-schedule/kube-endpoints.yaml)文件中需要修改成你集群的IP地址即可
## alertmanager
- 增删改触发器：编辑[alertmanager-configMapRules](https://github.com/marksugar/k8s-pgmon/blob/master/alertmanager/alertmanager-configMapRules.yaml)配置文件即可，报警媒介请参考上述的微信和dingding，推荐使用[微信](https://github.com/marksugar/k8s-pgmon/tree/master/WeChat)

- 微信[配置文件](https://github.com/marksugar/k8s-pgmon/blob/master/WeChat/alertmanager-configMapWechat.yaml)修改

如下

```
    receivers:
    - name: 'wechat'
      wechat_configs:
      - api_secret: 'xxxxxxxxx'
        send_resolved: true
        to_user: '@all'
        to_party: 'marksugar'
        agent_id: '100000X'
        corp_id: 'ww0164xxxxx'
```		
参考[wechat_config](https://github.com/marksugar/k8s-pgmon#wechat_config)

如果你不想使用任何报警媒介，你可以使用默认的[配置文件](https://github.com/marksugar/k8s-pgmon/blob/master/alertmanager/alertmanager-configmap.yaml_default)

## grafana

模板的添加和删除，需要修改[模板配置文件Dashboard](https://github.com/marksugar/k8s-pgmon/blob/master/grafana/grafana-configMapDashboardDefinitions.yaml),可以直接在里面添加，也可以添加单独的configMap文件而后挂载到grafana中

## prometheus

prometheus[配置文件](https://github.com/marksugar/k8s-pgmon/blob/master/prometheus/prometheus-configmap.yaml)做了简单的发现，如有需要可进行调整即可



