> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7316353049315409935)

准备环境
----

> 在此专栏的前几篇文章中已经准备了一台服务器作为我们进行环境的准备. 大家也可以通过虚拟机创建俩台服务器, 一台作为 Prometheus 的安装另外一台进行其他软件安装并且进行监控的服务器.

> 这里我就不赘述 nginx 的安装教程, 相信大家都可以搜到, 使用 docker 或者直接通过安装包解压的方式都可以, 我这里是通过 docker 的方式进行安装的, 后面的操作其实都是大差不差的.

nginx 开启 stub_status
--------------------

*   监控 nginx 需要 with-http_stub_status_module 这个模块

首先检查是否有安装 with-http_stub_status_module 模块

**docker 方式安装**

```
docker exec -it nginx nginx -v 2>&1  |grep -o with-http_stub_status_module


```

**nginx 安装包方式安装**

```
nginx nginx -v 2>&1  |grep -o with-http_stub_status_module


```

### nginx 开启 stub_status 配置

将下面配置文件写到 nginx.conf 配置文件中:

```
server {
	.....
	location /stub_status {
		stub_status on;
		access_log off;
		allow 0.0.0.0/0;
		deny all;
	}
	....
}


```

### 重新加载配置文件

```
docker exec -it nginx nginx -s reload


```

### 检查是否开启成功

```
curl http://localhost/syub_status


```

成功如下图:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b0e7a00b1df64f24b85925a4e13bf44a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=464&h=73&s=31852&e=png&b=020202)

安装 Exporter
-----------

> 在上篇文章中说了 Prometheus 需要监控什么软件需要对应安装的 Exporter, 当然这里可以使用二进制安装也可以使用 docker 安装. 这里为了方便, 还是选择 docker-compose 的方式直接安装

### docker-compose 方式进行安装

这里直接通过创建 docker-compose.yaml 然后进行追加配置

```
cat >docker-compose.yaml <<FOF
version: '3.3'
services:
 nginx_exproter:
 	image:nginx/nginx-prometheus-exporter:0.11
 	container_name: nginx_exporter
 	hostname: nginx_exporter
 	command:
 	 - '-nginx.scrape-uri=http://localhost/stub_status'
 	 restart: always
 	 port:
 	 - "9113:9113"
EOF


```

### 启动

```
docker-compose up -d


```

### 检查

```
查看正在运行的容器
docker ps

或者:

查看nginx_exporter容器的运行日志
docker logs -f nginx_exporter


```

### metrics 地址

安装好 Exporter 后会暴露一个 / metrics 结尾的服务

<table><thead><tr><th>名称</th><th>地址</th></tr></thead><tbody><tr><td>nginx_exporter</td><td><a href="https://link.juejin.cn?target=http%3A%2F%2Flocalhost%3A9113%2Fmetrics" target="_blank" title="http://localhost:9113/metrics" ref="nofollow noopener noreferrer">http://localhost:9113/metrics</a></td></tr></tbody></table>

### Prometheus 配置

配置 Prometheus 去采集 (拉取)nginx_exporter 的监控样本数据

```
cd /data/docker-prometheus

# 在scrapc_configs(搜刮配置):下面增加如下配置:
cat >prometheus/prometheus.yml <<FOF
 - job_name: 'nginx_exporter'
   static_configs:
   - targets: ['localhost:9113']
   	 labels:
   	 	instance: test服务器 
EOF


```

重新加载配置

```
curl -x POST http://localhost:9090/-/reload


```

### 检查

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e097bb0a0924f7090ab090dc8eb7f4b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1664&h=474&s=119692&e=png&b=ffffff)

### 常用的 nginx 监控指标

```
nginx_connections_accepted	接受请求数
nginx_connections_active	活动连接数
nginx_connections_handled	成功处理请求数
nginx_connections_reding	正在进行读操作的请求数
nginx_connections_waiting	正在等待的请求数
nginx_connections_writing	正在进行写操作的请求数
nginx_connections_requests	总请求数


```

添加触发器
-----

```
cd /data/docker-prometheus


```

```
cat >prometheus/alert.yml <<FOF
 -name: nginx
  rules:
  # 任何势力超过30秒无法联系的情况发出警报
  - alert: NginxDown
  	expr: nginx_up ==0
  	for: 30s
  	labels:
  	  severity: critical
  	annotations:
  		summary:"nginx异常,实例:{{$labels.instance }}"
  		description: "{{$lables.job}} nginx已关闭"
EOF


```

检查:

```
vim prometheus/alert.yml


```

### 检查配置

```
docker exec -it prometheus promtool check config /etc/prometheus/prometheus.yml


```

### 重新加载配置

```
curl -x POST http://localhost:9090/-/reload


```

### 检查

[http://localhost:9090/alerts?search=](https://link.juejin.cn?target=http%3A%2F%2Flocalhost%3A9090%2Falerts%3Fsearch%3D "http://localhost:9090/alerts?search=")

或:

[http://localhost:9090/rules](https://link.juejin.cn?target=http%3A%2F%2Flocalhost%3A9090%2Frules "http://localhost:9090/rules")

### dashboard

grafana 展示 Prometheus 从 nginx_exporter 收集到的数据