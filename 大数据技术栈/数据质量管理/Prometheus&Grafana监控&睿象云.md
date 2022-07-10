尚硅谷大数据项目之实时项目Prometheus&Grafana 监控

(作者：尚硅谷大数据研发部)

版本：V2.0

# 第1章 Prometheus 入门

Prometheus 受启发于 Google 的 Brogmon 监控系统（相似的 Kubernetes 是从 Google的 Brog 系统演变而来），从 2012 年开始由前 Google 工程师在 Soundcloud 以开源软件的形式进行研发，并且于 2015 年早期对外发布早期版本。

2016 年 5 月继 Kubernetes 之后成为第二个正式加入 CNCF 基金会的项目，同年 6 月正式发布 1.0 版本。2017 年底发布了基于全新存储层的 2.0 版本，能更好地与容器平台、云平台配合。

Prometheus 作为新一代的云原生监控系统，目前已经有超过 650+位贡献者参与到Prometheus 的研发工作上，并且超过 120+项的第三方集成。

## 1.1 Prometheus 的特点

Prometheus 是一个开源的完整监控解决方案，其对传统监控系统的测试和告警模型进行了彻底的颠覆，形成了基于中央化的规则计算、统一分析和告警的新模型。 相比于传统监控系统，Prometheus 具有以下优点：

### 1.1.1 易于管理

➢Prometheus 核心部分只有一个单独的二进制文件，不存在任何的第三方依赖(数据库，缓存等等)。唯一需要的就是本地磁盘，因此不会有潜在级联故障的风险。

➢Prometheus 基于 Pull 模型的架构方式，可以在任何地方（本地电脑，开发环境，测试环境）搭建我们的监控系统。

➢对于一些复杂的情况，还可以使用 Prometheus 服务发现(Service Discovery)的能力动态管理监控目标。

### 1.1.2 监控服务的内部运行状态

Pometheus 鼓励用户监控服务的内部状态，基于 Prometheus 丰富的 Client 库，用户可以轻松的在应用程序中添加对 Prometheus 的支持，从而让用户可以获取服务和应用内部真正的运行状态。

![image-20220527154205465](Prometheus&Grafana监控&睿象云.assets/image-20220527154205465.png)

### 1.1.3 强大的数据模型

所有采集的监控数据均以指标(metric)的形式保存在内置的时间序列数据库当中(TSDB)。所有的样本除了基本的指标名称以外，还包含一组用于描述该样本特征的标签。

如下所示：

```sh
http_request_status{code='200',content_path='/api/path',environment='produment'} =>
[value1@timestamp1,value2@timestamp2...]

http_request_status{code='200',content_path='/api/path2',environment='produment'} =>
[value1@timestamp1,value2@timestamp2...]
```

每一条时间序列由指标名称(Metrics Name)以及一组标签(Labels)唯一标识。每条时间序列按照时间的先后顺序存储一系列的样本值。

➢ http_request_status：指标名称(Metrics Name)

➢ {code='200',content_path='/api/path',environment='produment'}：表示维度的标签，基于这些 Labels 我们可以方便地对监控数据进行聚合，过滤，裁剪。

➢[value1@timestamp1,value2@timestamp2...]：按照时间的先后顺序 存储的样本值。

### 1.1.4 强大的查询语言 PromQL

Prometheus 内置了一个强大的数据查询语言 PromQL。 通过 PromQL 可以实现对监控数据的查询、聚合。同时 PromQL 也被应用于数据可视化(如 Grafana)以及告警当中。

通过 PromQL 可以轻松回答类似于以下问题：

➢ 在过去一段时间中 95%应用延迟时间的分布范围？

➢ 预测在 4 小时后，磁盘空间占用大致会是什么情况？

➢ CPU 占用率前 5 位的服务有哪些？(过滤)

### 1.1.5 高效

对于监控系统而言，大量的监控任务必然导致有大量的数据产生。而 Prometheus 可以高效地处理这些数据，对于单一 Prometheus Server 实例而言它可以处理：

➢ 数以百万的监控指标

➢ 每秒处理数十万的数据点


### 1.1.6 可扩展

可以在每个数据中心、每个团队运行独立的 Prometheus Sevrer。Prometheus 对于联邦集群的支持，可以让多个 Prometheus 实例产生一个逻辑集群，当单实例 Prometheus Server 处理的任务量过大时，通过使用功能分区(sharding)+联邦集群(federation)可以对其进行扩展。

### 1.1.7 易于集成

使用 Prometheus 可以快速搭建监控服务，并且可以非常方便地在应用程序中进行集成。目前支持： Java， JMX， Python， Go， Ruby， .Net， Node.js 等等语言的客户端 SDK，基于这些 SDK 可以快速让应用程序纳入到 Prometheus 的监控当中，或者开发自己的监控
数据收集程序。

同时这些客户端收集的监控数据，不仅仅支持 Prometheus，还能支持 Graphite 这些其他的监控工具。同时 Prometheus 还支持与其他的监控系统进行集成： Graphite， Statsd， Collected，Scollector， muini， Nagios 等。 Prometheus 社区还提供了大量第三方实现的监控数据采集支持： JMX， CloudWatch， EC2， MySQL， PostgresSQL， Haskell， Bash， SNMP，Consul，Haproxy，Mesos，Bind，CouchDB，Django，Memcached，RabbitMQ，Redis，RethinkDB，Rsyslog 等等。

### 1.1.8 可视化

➢Prometheus Server 中自带的 Prometheus UI，可以方便地直接对数据进行查询，并且支持直接以图形化的形式展示数据。同时 Prometheus 还提供了一个独立的基于Ruby On Rails 的 Dashboard 解决方案 Promdash。

➢最新的 Grafana 可视化工具也已经提供了完整的 Prometheus 支持，基于 Grafana 可以创建更加精美的监控图标。

➢基于 Prometheus 提供的 API 还可以实现自己的监控可视化 UI。

### 1.1.9 开放性

通常来说当我们需要监控一个应用程序时，一般需要该应用程序提供对相应监控系统协议的支持，因此应用程序会与所选择的监控系统进行绑定。为了减少这种绑定所带来的限制，对于决策者而言要么你就直接在应用中集成该监控系统的支持，要么就在外部创建单独的服
务来适配不同的监控系统。而对于 Prometheus 来说，使用 Prometheus 的 client library 的输出格式不止支持Prometheus 的格式化数据，也可以输出支持其它监控系统的格式化数据，比如 Graphite。

因此你甚至可以在不使用 Prometheus 的情况下，采用 Prometheus 的 client library 来让你的应用程序支持监控数据采集。

## 1.2 Prometheus 的架构

![image-20220527155028377](Prometheus&Grafana监控&睿象云.assets/image-20220527155028377.png)

### 1.2.1 Prometheus 生态圈组件

➢ Prometheus Server：主服务器，负责收集和存储时间序列数据

➢ client libraies：应用程序代码插桩，将监控指标嵌入到被监控应用程序中

➢ Pushgateway：推送网关，为支持 short-lived 作业提供一个推送网关

➢ exporter：专门为一些应用开发的数据摄取组件—exporter，例如： HAProxy、 StatsD、Graphite 等等。

➢ Alertmanager：专门用于处理 alert 的组件

### 1.2.2 架构理解

Prometheus 既然设计为一个维度存储模型，可以把它理解为一个 OLAP 系统。

**1、存储计算层**

➢ Prometheus Server，里面包含了存储引擎和计算引擎。

➢ Retrieval 组件为取数组件，它会主动从 Pushgateway 或者 Exporter 拉取指标数据。

➢ Service discovery，可以动态发现要监控的目标。

➢ TSDB，数据核心存储与查询。

➢ HTTP server，对外提供 HTTP 服务。

**2、采集层**

采集层分为两类，一类是生命周期较短的作业，还有一类是生命周期较长的作业。

➢ 短作业：直接通过 API，在退出时间指标推送给 Pushgateway。

➢ 长作业：Retrieval 组件直接从 Job 或者 Exporter 拉取数据。

**3、应用层**

应用层主要分为两种，一种是 AlertManager，另一种是数据可视化。

➢AlertManager

对接 Pagerduty，是一套付费的监控报警系统。可实现短信报警、5 分钟无人 ack 

打电话通知、仍然无人 ack，通知值班人员 Manager...

Emial，发送邮件

... ...

➢数据可视化

Prometheus build-in WebUI 

Grafana

其他基于 API 开发的客户端


# 第2章 Prometheus 的安装

官网：https://prometheus.io/

下载地址：https://prometheus.io/download/

## 2.1 安装 Prometheus Server

Prometheus 基于 Golang 编写，编译后的软件包，不依赖于任何的第三方依赖。只需要下载对应平台的二进制包，解压并且添加基本的配置即可正常启动 Prometheus Server。

### 2.1.1 上传安装包

上传 prometheus-2.29.1.linux-amd64.tar.gz 到虚拟机的/opt/software 目录

### 2.1.2 解压安装包

➢解压到/opt/module 目录下

```sh
[atguigu@hadoop202 software]$ tar -zxvf prometheus-2.29.1.linux-amd64.tar.gz -C  /opt/module
```

➢修改目录名

```SH
[atguigu@hadoop202 ~] cd /opt/module
[atguigu@hadoop202 module] mv prometheus-2.29.1.linux-amd64 prometheus-2.29.1
```

### 2.1.3 修改配置文件 prometheus.yml

```SH
[atguigu@hadoop202 prometheus-2.29.1]$ vim prometheus.yml
```

在 scrape_configs 配置项下添加配置：

```SH
scrape_configs:
	- job_name: 'prometheus'
  	    static_configs:
	  - targets: ['hadoop202:9090']

# 添加 PushGateway 监控配置
	- job_name: 'pushgateway'
 	   static_configs:
	  - targets: ['hadoop202:9091']
	    labels:
	    instance: pushgateway

# 添加 Node Exporter 监控配置
- job_name: 'node exporter'
  static_configs:
  - targets: ['hadoop202:9100', 'hadoop203:9100', 'hadoop204:9100']
```

配置说明：

1、global 配置块：控制 Prometheus 服务器的全局配置

➢ scrape_interval：配置拉取数据的时间间隔，默认为 1 分钟。

➢ evaluation_interval：规则验证（生成 alert）的时间间隔，默认为 1 分钟。

2、rule_files 配置块：规则配置文件

3、 scrape_configs 配置块：配置采集目标相关， prometheus 监视的目标。 Prometheus自身的运行信息可以通过 HTTP 访问，所以 Prometheus 可以监控自己的运行数据。

➢ job_name：监控作业的名称

➢ static_configs：表示静态目标配置，就是固定从某个 target 拉取数据

➢ targets ： 指 定 监 控 的 目 标 ， 其 实 就 是 从 哪 儿 拉 取 数 据 。 Prometheus 会 从http://hadoop202:9090/metrics 上拉取数据。

Prometheus 是可以在运行时自动加载配置的。启动时需要添加：--web.enable-lifecycle

## 2.2 安装 Pushgateway

Prometheus 在正常情况下是采用拉模式从产生 metric 的作业或者 exporter （比如专门监控主机的 NodeExporter）拉取监控数据。但是我们要监控的是 Flink on YARN 作业，想要让 Prometheus 自动发现作业的提交、结束以及自动拉取数据显然是比较困难的。

PushGateway 就是一个中转组件，通过配置 Flink on YARN 作业将 metric 推到PushGateway，Prometheus 再从 PushGateway 拉取就可以了。

### 2.2.1 上传安装包

上传 pushgateway-1.4.1.linux-amd64.tar.gz 到虚拟机的/opt/software 目录

### 2.2.2 解压安装包

➢解压到/opt/module 目录下

```sh
[atguigu@hadoop202 software]$tar -zxvf pushgateway-1.4.1.linux-amd64.tar.gz -C /opt/module
```

➢ 修改目录名

```sh
[atguigu@hadoop202 ~] cd /opt/module
[atguigu@hadoop202 module] mv pushgateway-1.4.1.linux-amd64 pushgateway-1.4.1
```

## 2.3 安装 Alertmanager（选择性安装）

### 2.3.1 上传安装包

上传 alertmanager-0.23.0.linux-amd64.tar.gz 到虚拟机的/opt/software 目录

### 2.3.2 解压安装包

➢解压到/opt/module 目录下

```sh
[atguigu@hadoop202 software]$tar -zxvf alertmanager-0.23.0.linux-amd64.tar.gz -C /opt/module
```

➢ 修改目录名

```sh
[atguigu@hadoop202 ~] cd /opt/module
[atguigu@hadoop202 module] mv alertmanager-0.23.0.linux-amd64 alertmanager-0.23.0
```

## 2.4 安装 Node Exporter（选择性安装）

在 Prometheus 的架构设计中，Prometheus Server 主要负责数据的收集，存储并且对外提供数据查询支持，而实际的监控样本数据的收集则是由 Exporter 完成。因此为了能够监控到某些东西，如主机的 CPU 使用率，我们需要使用到 Exporter。Prometheus 周期性的从 Exporter 暴露的 HTTP 服务地址（通常是/metrics）拉取监控样本数据。

Exporter 可以是一个相对开放的概念，其可以是一个独立运行的程序独立于监控目标以外，也可以是直接内置在监控目标中。只要能够向 Prometheus 提供标准格式的监控样本数据即可。

为了能够采集到主机的运行指标如 CPU, 内存，磁盘等信息。我们可以使用 Node Exporter。Node Exporter 同样采用 Golang 编写，并且不存在任何的第三方依赖，只需要下载，解压即可运行。可以从 https://prometheus.io/download/ 获取最新的 node exporter 版本的二进制包。

### 2.4.1 上传安装包

上传 node_exporter-1.2.2.linux-amd64.tar.gz 到虚拟机的/opt/software 目录

### 2.4.2 解压安装包

➢解压到/opt/module 目录下

```sh
[atguigu@hadoop202 software]$ tar -zxvf node_exporter-1.2.2.linux-amd64.tar.gz -C /opt/module
```

➢修改目录名

```sh
[atguigu@hadoop202 ~] cd /opt/module
[atguigu@hadoop202 module] mv node_exporter-1.2.2.linux-amd64 node_exporter-1.2.2
```

➢启动并通过页面查看是否成功

执行./node_exporter

浏览器输入：http://hadoop202:9100/metrics，可以看到当前 node exporter 获取到的当前主机的所有监控数据。

### 2.4.3 节点分发

➢将解压后的目录分发到要监控的节点

```sh
[atguigu@hadoop202 module] xsync node_exporter-1.2.2
```



➢修改 Prometheus 配置文件 prometheus.yml，在 2.1.3 的时候已经添加过配置

```sh
- targets: ['hadoop202:9100', 'hadoop203:9100', 'hadoop204:9100']
```

### 2.4.4 设置为开机自启

➢ 创建 service 文件

```sh
[atguigu@hadoop202 module] sudo vim /usr/lib/systemd/system/node_exporter.service
[Unit]
Description=node_export
Documentation=https://github.com/prometheus/node_exporter
After=network.target
[Service]
Type=simple
User=atguigu
ExecStart= /opt/module/node_exporter-1.2.2/node_exporter
Restart=on-failure
[Install]
WantedBy=multi-user.target
```

➢分发文件

```sh
[atguigu@hadoop202 module] sudo /home/atguigu/bin/xsync /usr/lib/systemd/system/node_exporter.service
```

➢设为开机自启动（所有机器都执行）

```sh
[atguigu@hadoop202 module] sudo systemctl enable node_exporter.service
```

➢启动服务（所有机器都执行）

```sh
[atguigu@hadoop202 module] sudo systemctl start node_exporter.service
```

## 2.5 启 动Prometheus Server 、 Pushgateway和Alertmanager



### 2.5.1 Prometheus Server 目录下执行启动命令

```sh
[atguigu@hadoop202 prometheus-2.29.1]$ nohup ./prometheus --config.file=prometheus.yml > ./prometheus.log 2>&1 & 

```



### 2.5.2 Pushgateway 目录下执行启动命令

```sh
[atguigu@hadoop202
pushgateway-1.4.1]$ nohup ./pushgateway --web.listen-address :9091 > ./pushgateway.log 2>&1 &
```

#### 2.5.3 在 Alertmanager 目录下启动

```sh
[atguigu@hadoop202
alertmanager-0.23.0]$ nohup ./alertmanager --config.file=alertmanager.yml > ./alertmanager.log 2>&1 &
```

### 2.5.4 打开 web 页面查看

➢ 浏览器输入：http://hadoop202:9090/

➢ 点击 Status，选中 Targets：

![image-20220530092008814](Prometheus&Grafana监控&睿象云.assets/image-20220530092008814.png)

➢ prometheus、pushgateway 和 node exporter 都是 up 状态，表示安装启动成功：

![image-20220530092108664](Prometheus&Grafana监控&睿象云.assets/image-20220530092108664.png)



# 第3章 PromQL 介绍

Prometheus 通过指标名称（metrics name）以及对应的一组标签（labelset）唯一定义一条时间序列。指标名称反映了监控样本的基本标识，而 label 则在这个基本特征上为采集到的数据提供了多种特征维度。用户可以基于这些特征维度过滤，聚合，统计从而产生新的计算后的一条时间序列。PromQL 是 Prometheus 内置的数据查询语言，其提供对时间序列数据丰富的查询，聚合以及逻辑运算能力的支持。并且被广泛应用在 Prometheus的日常应用当中，包括对数据查询、可视化、告警处理当中。可以这么说，PromQL 是Prometheus 所有应用场景的基础，理解和掌握 PromQL 是 Prometheus 入门的第一课。

## 3.1 基本用法

### 3.1.1 查询时间序列

当 Prometheus 通过 Exporter 采集到相应的监控指标样本数据后，我们就可以通过PromQL 对监控样本数据进行查询。当我们直接使用监控指标名称查询时，可以查询该指标下的所有时间序列。如：

```sh
prometheus_http_requests_total
```

等同于：

```sh
prometheus_http_requests_total{}
```

该表达式会返回指标名称为 prometheus_http_requests_total 的所有时间序列：

```sh
prometheus_http_requests_total{code="200",handler="alerts",instance="localhost:9090",job="prometheus",method="get"}= (20889@1518096812.326)
prometheus_http_requests_total{code="200",handler="graph",instance="localhost:9090",job="prometheus",method="get"}= (21287@1518096812.326)
```

PromQL 还支持用户根据时间序列的标签匹配模式来对时间序列进行过滤，目前主要支持两种匹配模式：完全匹配和正则匹配。

➢PromQL 支持使用 = 和 != 两种完全匹配模式：

⚫ 通过使用 label=value 可以选择那些标签满足表达式定义的时间序列；

⚫ 反之使用 label!=value 则可以根据标签匹配排除时间序列；

例如，如果我们只需要查询所有 prometheus_http_requests_total 时间序列中满足标签 instance 为 localhost:9090 的时间序列，则可以使用如下表达式：

```sh
prometheus_http_requests_total{instance="localhost:9090"}
```

反之使用 instance!="localhost:9090" 则可以排除这些时间序列：

```sh
prometheus_http_requests_total{instance!="localhost:9090"}
```

➢PromQL 还可以支持使用正则表达式作为匹配条件，多个表达式之间使用 | 进行分离：

⚫ 使用 label=~regx 表示选择那些标签符合正则表达式定义的时间序列；

⚫ 反之使用 label!~regx 进行排除；

例如，如果想查询多个环节下的时间序列序列可以使用如下表达式：

```sh
prometheus_http_requests_total{environment=~"staging|testing|development",method!="GET"} 
```

排除用法

```sh
prometheus_http_requests_total{environment!~"staging|testing|development",method!="GET"}
```

### 3.1.2 范围查询

直接通过类似于 PromQL 表达式 httprequeststotal 查询时间序列时，返回值中只会包含该时间序列中的最新的一个样本值，这样的返回结果我们称之为瞬时向量。而相应的这样的表达式称之为__瞬时向量表达式。

而如果我们想过去一段时间范围内的样本数据时，我们则需要使用区间向量表达式。区间向量表达式和瞬时向量表达式之间的差异在于在区间向量表达式中我们需要定义时间选择的范围，时间范围通过时间范围选择器 [] 进行定义。 例如，通过以下表达式可以选择最近 5 分钟内的所有样本数据：

```sh
prometheus_http_requests_total{}[5m]
```

该表达式将会返回查询到的时间序列中最近 5 分钟的所有样本数据：

```sh
prometheus_http_requests_total{code="200",handler="alerts",instance="localhost:9090",job="prometheus",method="get"}=[
1@1518096812.326
1@1518096817.326
1@1518096822.326
1@1518096827.326
1@1518096832.326
1@1518096837.325
]

prometheus_http_requests_total{code="200",handler="graph",instance="localhost:9090",job="prometheus",method="get"}=[
4@1518096812.326
4@1518096817.326
4@1518096822.326
4@1518096827.326
4@1518096832.326
4@1518096837.325
]
```

通过区间向量表达式查询到的结果我们称为区间向量。 除了使用 m 表示分钟以外，PromQL 的时间范围选择器支持其它时间单位：

⚫ s - 秒

⚫ m - 分钟

⚫ h - 小时

⚫ d - 天

⚫ w - 周

⚫ y - 年

### 3.1.3 时间位移操作

在瞬时向量表达式或者区间向量表达式中，都是以当前时间为基准：

```txt
prometheus_http_requests_total{} # 瞬时向量表达式，选择当前最新的数据

prometheus_http_requests_total{}[5m] # 区间向量表达式，选择以当前时间为基准，5 分钟内的数据
```

而如果我们想查询，5 分钟前的瞬时样本数据，或昨天一天的区间内的样本数据呢? 这个时候我们就可以使用位移操作，位移操作的关键字为 offset。 可以使用 offset 时间位移操作：

```sh
prometheus_http_requests_total{} offset 5m

prometheus_http_requests_total{}[1d] offset 1d
```



### 3.1.4 使用聚合操作

一般来说，如果描述样本特征的标签(label)在并非唯一的情况下，通过 PromQL 查询数据，会返回多条满足这些特征维度的时间序列。而 PromQL 提供的聚合操作可以用来对这些时间序列进行处理，形成一条新的时间序列：

```sh
# 查询系统所有 http 请求的总量
sum(prometheus_http_requests_total)

# 按照 mode 计算主机 CPU 的平均使用时间
avg(node_cpu_seconds_total) by (mode)

# 按照主机查询各个主机的 CPU 使用率
sum(sum(irate(node_cpu_seconds_total{mode!='idle'}[5m]))
/sum(irate(node_cpu_seconds_total [5m]))) by (instance)
```

### 3.1.5 标量和字符串

除了使用瞬时向量表达式和区间向量表达式以外，PromQL 还直接支持用户使用标量(Scalar)和字符串(String)。

➢标量（Scalar）：一个浮点型的数字值

标量只有一个数字，没有时序。 例如：10

需要注意的是，当使用表达式 count(prometheus_http_requests_total)，返回的数据类型，依然是瞬时向量。用户可以通过内置函数 scalar()将单个瞬时向量转换为标量。

➢字符串（String）：一个简单的字符串值

直接使用字符串，作为 PromQL 表达式，则会直接返回字符串。

```sh
"this is a string"
'these are unescaped: \n \\ \t'
`these are not unescaped: \n ' " \t`
```

### 3.1.6 合法的 PromQL 表达式

所有的 PromQL 表达式都必须至少包含一个指标名称(例如 http_request_total)，或者一个不会匹配到空字符串的标签过滤器(例如{code=”200”})。

因此以下两种方式，均为合法的表达式：

```sh
prometheus_http_requests_total # 合法
prometheus_http_requests_total{} # 合法
{method="get"} # 合法
```

而如下表达式，则不合法：

```sh
{job=~".*"} # 不合法
```

同时，除了使用 {label=value} 的形式以外，我们还可以使用内置的 __name__ 标签来指定监控指标名称：

```sh
{__name__=~"prometheus_http_requests_total"} # 合法
{__name__=~"node_disk_bytes_read|node_disk_bytes_written"} # 合法
```



## 3.2 PromQL 操作符

使用 PromQL 除了能够方便的按照查询和过滤时间序列以外，PromQL 还支持丰富的操作符，用户可以使用这些操作符对进一步的对事件序列进行二次加工。这些操作符包括：数学运算符，逻辑运算符，布尔运算符等等。

### 3.2.1 数学运算

PromQL 支持的所有数学运算符如下所示：

⚫ + (加法)

⚫ - (减法)

⚫ * (乘法)

⚫ / (除法)

⚫ % (求余)

⚫ ^ (幂运算)

### 3.2.2 布尔运算


➢**Prometheus 支持以下布尔运算符如下：**

⚫ == (相等)

⚫ != (不相等)

⚫ >(大于)

⚫ < (小于)

⚫ >= (大于等于)

⚫ <= (小于等于)

➢**使用 bool 修饰符改变布尔运算符的行为**

布尔运算符的默认行为是对时序数据进行过滤。而在其它的情况下我们可能需要的是真正的布尔结果。例如，只需要知道当前模块的 HTTP 请求量是否>=1000，如果大于等于1000 则返回 1（true）否则返回 0（false）。这时可以使用 bool 修饰符改变布尔运算的默认行为。 例如：

```sh
prometheus_http_requests_total > bool 1000
```

使用 bool 修改符后，布尔运算不会对时间序列进行过滤，而是直接依次瞬时向量中的各个样本数据与标量的比较结果 0 或者 1。从而形成一条新的时间序列。

```sh
prometheus_http_requests_total{code="200",handler="query",instance="localhost:9090",job="prometheus",method="get"}		1
prometheus_http_requests_total{code="200",handler="query_range",instance="localhost:9090",job="prometheus",method="get"} 0
```

同时需要注意的是，如果是在两个标量之间使用布尔运算，则必须使用 bool 修饰符

```sh
2 == bool 2 # 结果为 1
```

### 3.2.3 使用集合运算符

使用瞬时向量表达式能够获取到一个包含多个时间序列的集合，我们称为瞬时向量。 通过集合运算，可以在两个瞬时向量与瞬时向量之间进行相应的集合操作。

目前，Prometheus 支持以下集合运算符：

⚫ and (并且)

⚫ or (或者)

⚫ unless (排除)

vector1 and vector2 会产生一个由 vector1 的元素组成的新的向量。该向量包含vector1 中完全匹配 vector2 中的元素组成。

vector1 or vector2 会产生一个新的向量，该向量包含 vector1 中所有的样本数据，以及 vector2 中没有与 vector1 匹配到的样本数据。

vector1 unless vector2 会产生一个新的向量，新向量中的元素由 vector1 中没有与vector2 匹配的元素组成。

### 3.2.4 操作符优先级

对于复杂类型的表达式，需要了解运算操作的运行优先级。例如，查询主机的 CPU 使用率，可以使用表达式：

```sh
100 * (1 - avg (irate(node_cpu_seconds_total{mode='idle'}[5m])) by(job) )
```

其中 irate 是 PromQL 中的内置函数，用于计算区间向量中时间序列每秒的即时增长率。在 PromQL 操作符中优先级由高到低依次为：

⚫ ^

⚫ *, /, %

⚫ +, -

⚫ ==, !=, <=, =, >

⚫ and, unless

⚫ or

### 3.2.5 PromQL 聚合操作

Prometheus 还提供了下列内置的聚合操作符，这些操作符作用域瞬时向量。可以将瞬时表达式返回的样本数据进行 聚合，形成一个新的时间序列。

⚫ sum (求和)

⚫ min (最小值)

⚫ max (最大值)

⚫ avg (平均值)

⚫ stddev (标准差)

⚫ stdvar (标准差异)

⚫ count (计数)

⚫ count_values (对 value 进行计数)

⚫ bottomk (后 n 条时序)

⚫ topk (前 n 条时序)

⚫ quantile (分布统计)

使用聚合操作的语法如下：

```sh
<aggr-op>([parameter,] <vector expression>) [without|by (<label list>)]
```

其中只有 count_values , quantile , topk , bottomk 支持参数(parameter)。

without 用于从计算结果中移除列举的标签，而保留其它标签。by 则正好相反，结果向量中只保留列出的标签，其余标签则移除。通过 without 和 by 可以按照样本的问题对数据进行聚合。

例如：

```sh
sum(prometheus_http_requests_total) without (instance)
```

等价于

```sh
sum(prometheus_http_requests_total) by (code,handler,job,method)
```

如果只需要计算整个应用的 HTTP 请求总量，可以直接使用表达式：

```sh
sum(prometheus_http_requests_total)
```

count_values 用于时间序列中每一个样本值出现的次数。 count_values 会为每一个唯一的样本值输出一个时间序列，并且每一个时间序列包含一个额外的标签。 例如：

```sh
count_values("count", prometheus_http_requests_total)
```

topk 和 bottomk 则用于对样本值进行排序，返回当前样本值前 n 位，或者后 n 位的时间序列。

获取 HTTP 请求数前 5 位的时序样本数据，可以使用表达式：

```sh
topk(5, prometheus_http_requests_total)
```

quantile 用于计算当前样本数据值的分布情况 quantile(φ, express)其中 0 ≤ φ ≤ 1。

例如，当 φ 为 0.5 时，即表示找到当前样本数据中的中位数：

```sh
quantile(0.5, prometheus_http_requests_total)
```

# 第4章 Prometheus 和 Flink 集成

Flink 提供的 Metrics 可以在 Flink 内部收集一些指标，通过这些指标让开发人员更好地理解作业或集群的状态。由于集群运行后很难发现内部的实际状况，跑得慢或快，是否异常等，开发人员无法实时查看所有的 Task 日志。比如作业很大或者有很多作业的情况下，该如何处理？此时 Metrics 可以很好的帮助开发人员了解作业的当前状况。

从 Flink 的源码结构我们可以看到，Flink 官方支持 Prometheus，并且提供了对接Prometheus 的 jar 包，很方便就可以集成。

## 4.1 拷贝 jar 包

➢ 拷贝新的 flink 目录，flink-prometheus

➢ 将 flink-metrics-prometheus-1.12.0.jar 拷贝到 <flink_home>/lib 目录下

```sh
[atguigu@hadoop202 flink-prometheus]$ cp /opt/module/flink-prometheus/plugins/metrics-prometheus/flink-metrics-prometheus-1.12.0.jar /opt/module/flink-prometheus/lib/
```

Flink 的 Classpath 位于 lib 目录下，所以插件的 jar 包需要放到该目录下

## 4.2 修改 Flink 配置

进入到 Flink 的 conf 目录，修改 flink-conf.yaml

```sh
[atguigu@hadoop202 conf]$ vim flink-conf.yaml
```

添加如下配置：

```sh
##### 与 Prometheus 集成配置 #####

metrics.reporter.promgateway.class:
org.apache.flink.metrics.prometheus.PrometheusPushGatewayReporter

# PushGateway 的主机名与端口号

metrics.reporter.promgateway.host: hadoop202
metrics.reporter.promgateway.port: 9091

# Flink metric 在前端展示的标签（前缀）与随机后缀

metrics.reporter.promgateway.jobName: flink-metrics-ppg


metrics.reporter.promgateway.randomJobNameSuffix: true
metrics.reporter.promgateway.deleteOnShutdown: false
metrics.reporter.promgateway.interval: 30 SECONDS
```

## 4.3 为了运行测试程序，启动 netcat

```sh
[atguigu@hadoop202 sbin]$ nc -lk 9999
```



## 4.4 启动 hdfs、yarn，提交 flink 任务到 yarn 上

```sh
[atguigu@hadoop202 flink-prometheus]$ bin/flink run -t yarn-per-job -c com.atguigu.flink.chapter02.Flink03_WordCount_UnboundStream ./flink-base-1.0-SNAPSHOT-jar-with-dependencies.jar
```



## 4.5 可以通过 8088 跳到 flinkUI 的 job 页面，查看指标统计



## 4.6 刷新 Prometheus 页面，如果有 flink 指标，集成成功

![image-20220531100242881](Prometheus&Grafana监控&睿象云.assets/image-20220531100242881.png)

# 第5章 Prometheus 和 Grafana 集成

grafana 是一款采用 Go 语言编写的开源应用，主要用于大规模指标数据的可视化展现，是网络架构和应用分析中最流行的时序数据展示工具，目前已经支持绝大部分常用的时序数据库。下载地址：https://grafana.com/grafana/download

## 5.1 上传并解压

➢将 grafana-8.1.2.linux-amd64.tar.gz 上传至/opt/software/目录下，解压：

```sh
[atguigu@hadoop202 software]$ tar -zxvf grafana-enterprise-8.1.2.linux-amd64.tar.gz -C /opt/module/
```

## 5.2 启动 Grafana

```sh
[atguigu@hadoop202 grafana-8.1.2]$ nohup ./bin/grafana-server web > ./grafana.log 2>&1 &
```

➢打开 web：http://hadoop202:3000,默认用户名和密码：admin

## 5.3 添加数据源 Prometheus

点击配置，点击 Data Sources：

![image-20220531142407959](Prometheus&Grafana监控&睿象云.assets/image-20220531142407959.png)

点击添加按钮：

![image-20220531142418459](Prometheus&Grafana监控&睿象云.assets/image-20220531142418459.png)

找到 Prometheus，点击 Select

![image-20220531142447708](Prometheus&Grafana监控&睿象云.assets/image-20220531142447708.png)

配置 Prometheus Server 地址：

![image-20220531142500231](Prometheus&Grafana监控&睿象云.assets/image-20220531142500231.png)

点击下方的 Save&Test：

![image-20220531142514385](Prometheus&Grafana监控&睿象云.assets/image-20220531142514385.png)

出现绿色的提示框，表示与 Prometheus 正常联通：

![image-20220531142525568](Prometheus&Grafana监控&睿象云.assets/image-20220531142525568.png)

点击 Back 返回即可，可以看到 Data Sources 页面，出现了添加的 Prometheus:

![image-20220531142549115](Prometheus&Grafana监控&睿象云.assets/image-20220531142549115.png)

## 5.4 手动创建仪表盘 Dashboard

点击左边栏的 “+”号，选择 Dashboard:

 ![image-20220531142628619](Prometheus&Grafana监控&睿象云.assets/image-20220531142628619.png)

添加新的仪表板，点击 Add an empty panel：

![image-20220531142648756](Prometheus&Grafana监控&睿象云.assets/image-20220531142648756.png)

配置仪表板监控项：

![image-20220531142659668](Prometheus&Grafana监控&睿象云.assets/image-20220531142659668.png)

一个仪表板可以配置多个监控项，添加其他监控项：

![image-20220531142710920](Prometheus&Grafana监控&睿象云.assets/image-20220531142710920.png)

配置新的监控项：

![image-20220531142721096](Prometheus&Grafana监控&睿象云.assets/image-20220531142721096.png)

## 5.5 直接添加 Flink 模板

手动一个个添加 Dashboard 比较繁琐，Grafana 社区鼓励用户分享 Dashboard，通过 https://grafana.com/dashboards 网站，可以找到大量可直接使用的 Dashboard 模板。

Grafana 中所有的 Dashboard 通过 JSON 进行共享，下载并且导入这些 JSON 文件，就可以直接使用这些已经定义好的 Dashboard：

进入官网，搜索 Flink 模板：

![image-20220531142810666](Prometheus&Grafana监控&睿象云.assets/image-20220531142810666.png)

选择自己喜欢的模板（800+下载的这个模板相对指标较多）

选中跳转页面后，点击 Download JSON：

![image-20220531142834793](Prometheus&Grafana监控&睿象云.assets/image-20220531142834793.png)

点击 Grafana 界面左侧 ”+”号，选择 import：

![image-20220531143832751](Prometheus&Grafana监控&睿象云.assets/image-20220531143832751.png)

上传 JSON 文件：

![image-20220531143842483](Prometheus&Grafana监控&睿象云.assets/image-20220531143842483.png)

配置模板信息：

![image-20220531143854340](Prometheus&Grafana监控&睿象云.assets/image-20220531143854340.png)

导入完，在首页即可看见添加的仪表盘，点击进去查看：

![image-20220531143906385](Prometheus&Grafana监控&睿象云.assets/image-20220531143906385.png)

正常提交 job，即可在 grafana 看到相关监控项的情况。

注意：代码里 env.execute(“作业名”),最好指定不同的作业名用于区分，不指定会使用默认的作业名：Flink Streaming Job，在 Grafana 页面就无法区分不同 job！！！

## 5.6 添加 Node Exporter 模板

同 5.5，进入 https://grafana.com/dashboards 页面，

➢ 搜索 Node Exporter，选择下载量最高的中文版本：

![image-20220531143956937](Prometheus&Grafana监控&睿象云.assets/image-20220531143956937.png)

➢ 下载模板 json 文件

![image-20220531144059652](Prometheus&Grafana监控&睿象云.assets/image-20220531144059652.png)

➢ 在 Grafana 中导入模板：

![image-20220531144113509](Prometheus&Grafana监控&睿象云.assets/image-20220531144113509.png)

![image-20220531144123233](Prometheus&Grafana监控&睿象云.assets/image-20220531144123233.png)

➢ 欣赏酷炫又详细的监控页：

![image-20220531144134518](Prometheus&Grafana监控&睿象云.assets/image-20220531144134518.png)

![image-20220531144144309](Prometheus&Grafana监控&睿象云.assets/image-20220531144144309.png)

![image-20220531144152674](Prometheus&Grafana监控&睿象云.assets/image-20220531144152674.png)

## 5.7 组件启停脚本

➢进入到/home/atguigu/bin 目录下，创建脚本 flink-monitor.sh

```sh
#!/bin/bash
case $1 in
"start"){
	echo '----- 启动 prometheus -----'
	nohup /opt/module/prometheus-2.29.1/prometheus --web.enable-admin-api --config.file=/opt/module/prometheus-2.29.1/prometheus.yml > /opt/module/prometheus-2.29.1/prometheus.log 2>&1 &
	echo '----- 启动 pushgateway -----'
	nohup /opt/module/pushgateway-1.4.1/pushgateway --web.listen-address :9091 > /opt/module/pushgateway-1.4.1/pushgateway.log 2>&1 &
	echo '----- 启动 grafana -----'
	nohup /opt/module/grafana-8.1.2/bin/grafana-server --homepath /opt/module/grafana-8.1.2 web > /opt/module/grafana-8.1.2/grafana.log 2>&1 &
};;

"stop"){
	echo '----- 停止 grafana -----'
	pgrep -f grafana | xargs kill
	echo '----- 停止 pushgateway -----'
	pgrep -f pushgateway | xargs kill
	echo '----- 停止 prometheus -----'
	pgrep -f prometheus | xargs kill
};;
esac
```

➢脚本添加执行权限

```SH
[atguigu@hadoop202 bin]$ chmod +x flink-monitor.sh
```

## 5.8 配置案例

### 5.8.1 任务失败监控

这一个指标监控主要是基于 flink_jobmanager_job_uptime 这个指标进行了监控。原理是在 job 任务存活时，会按照配置 metrics.reporter.promgateway.interval 上报频率递增。基于这个特点，当任务失败后这个数值就不会改变，就能监控到任务失败。

➢**添加监控项：**

![image-20220531144602839](Prometheus&Grafana监控&睿象云.assets/image-20220531144602839.png)

![image-20220531144614247](Prometheus&Grafana监控&睿象云.assets/image-20220531144614247.png)

30 秒为数据上报到 promgateway 频率，除以 100 为了数据好看，当 job 任务失败后数 flink 上报的 promgateway 的 flink_jobmanager_job_uptime 指标值不会变化。((flink_jobmanager_job_uptime)-(flink_jobmanager_job_uptime offset 30s))/100 值就会是 0，可以配置告警。

➢**配置告警**

![image-20220531144704025](Prometheus&Grafana监控&睿象云.assets/image-20220531144704025.png)

![image-20220531144714944](Prometheus&Grafana监控&睿象云.assets/image-20220531144714944.png)

![image-20220531144725215](Prometheus&Grafana监控&睿象云.assets/image-20220531144725215.png)

在告警通知中可以邮件和 webhook,webhook 可以调用相关接口，执行一些动作。webhook 需要提前配置，在这里配置告警时就可以直接引入。

### 5.8.2 网络延时或任务重启监控

这个告警也是基于 flink_jobmanager_job_uptime 指标，在出现网络延时或者重启后进行监控通知，监控指标如下：

((flink_jobmanager_job_uptime offset 30s)-(flink_jobmanager_job_uptime))/1000

1）延时会导致值突然小于-30（正常情况为-30）

2）重启会导致 flink_jobmanager_job_uptime 指标清零从新从 0 值上报，导致查询公式值突然大于 0（正常情况为-30）

➢ 添加监控项

![image-20220531145010173](Prometheus&Grafana监控&睿象云.assets/image-20220531145010173.png)

➢ 配置告警规则：

![image-20220531145019839](Prometheus&Grafana监控&睿象云.assets/image-20220531145019839.png)

### 5.8.3 重启次数

基于 flink_jobmanager_job_numRestarts 指标，表示 flink job 的重启次数。一般设置重启策略后，在任务异常重启后这个数值会递增+1。可以单纯的监控重启次数，也可以每次重启都进行告警（差值）。

![image-20220531145111386](Prometheus&Grafana监控&睿象云.assets/image-20220531145111386.png)

利用当前值减去 30 秒前的值，如果等于 1 证明重启了一次。

➢添加告警规则：

![image-20220531145121696](Prometheus&Grafana监控&睿象云.assets/image-20220531145121696.png)

# 第6章 集成第三方告警平台睿象云

邮件通知常会出现接收不及时的问题，为确保通知信息被及时接收，可通过配置Prometheus 或者 Grafana 与第三方平台告警平台（例如睿象云）集成，进而通过第三方平台提供的多种告警媒介（例如电话，短信）等发送告警信息。

本文以第三方告警平台睿象云为例，进行集成演示。

## 6.1 注册睿象云账号

集成睿象云之前须在其官网进行注册并登录，注册时需填入个人手机号和电子邮箱，以下是其官方网站 https://www.aiops.com。登录之后会看到如下界面。

![image-20220531145213870](Prometheus&Grafana监控&睿象云.assets/image-20220531145213870.png)

## 6.2 集成 Grafana

1. 点击 CA 智能告警平台

   ![image-20220531145226555](Prometheus&Grafana监控&睿象云.assets/image-20220531145226555.png)

2. 点击集成

   ![image-20220531145236047](Prometheus&Grafana监控&睿象云.assets/image-20220531145236047.png)

3. 选择 Grafana

   ![image-20220531145247014](Prometheus&Grafana监控&睿象云.assets/image-20220531145247014.png)


4. 填入应用名称，并点击“保存并获取应用 key”

   ![image-20220531145256747](Prometheus&Grafana监控&睿象云.assets/image-20220531145256747.png)

5. 得到 AppKey 之后，配置 Grafana

   ![image-20220531145308367](Prometheus&Grafana监控&睿象云.assets/image-20220531145308367.png)

6. 在 Grafana 中创建 Notification channel，

   ![image-20220531145324285](Prometheus&Grafana监控&睿象云.assets/image-20220531145324285.png)


7. 配置 channel

   ![image-20220531145338315](Prometheus&Grafana监控&睿象云.assets/image-20220531145338315.png)

8. Test&Save 测试后会接到电话以及邮件

   ![image-20220531145354307](Prometheus&Grafana监控&睿象云.assets/image-20220531145354307.png)

### 6.3 配置分派策略

分派策略可以配置，哪些应用的告警信息，发送给哪些用户。例如实时数仓的告警信息发送给张三

1. 点击“配置”→“分派策略”→“新建分派”

   ![image-20220531145437768](Prometheus&Grafana监控&睿象云.assets/image-20220531145437768.png)

2. 配置具体分派策略

   ![image-20220531145449072](Prometheus&Grafana监控&睿象云.assets/image-20220531145449072.png)

## 6.4 配置通知策略

通知策略，可以配置被分派人接收告警的通知方式，通知时间，通知延时等等。

1）点击“配置”→“通知策略”→“新建通知”

![image-20220531145505123](Prometheus&Grafana监控&睿象云.assets/image-20220531145505123.png)

2）配置具体的通知策略

![image-20220531145514190](Prometheus&Grafana监控&睿象云.assets/image-20220531145514190.png)

## 6.5 测试电话、短信和邮件通知

我们将 netcat 停掉，flink 服务就会停止，随后即可触发睿象云的动作，进而根据我们配置的分派策略和通知策略，发送告警信息。按照本文的配置，告警信息会以邮件、短信和电话的方式发送到注册时填入的手机号码。
