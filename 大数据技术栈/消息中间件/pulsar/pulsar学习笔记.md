# pulsar 2.8.0 文档



## 一 、pulsar 安装

### 1. 本地安装（单机模式）

```sh
wget https://archive.apache.org/dist/pulsar/pulsar-2.8.0/apache-pulsar-2.8.0-bin.tar.gz 
```

![1626656682896](pulsar学习笔记.assets/1626656682896.png)

```sh
$ tar xvfz apache-pulsar-2.8.0-bin.tar.gz 
$ cd apache-pulsar-2.8.0
```

单机启动

```sh
$ bin/pulsar standalone
```

consumer 一条消息

```sh
$ bin/pulsar-client consume my-topic -s "first-subscription"
```

produce一条消息

```sh
$ bin/pulsar-client produce my-topic --messages "hello-pulsar" 
```

### 2. docker模式安装

```
$ docker run -it \
-p 6650:6650 \
-p 8080:8080 \
--mount source=pulsardata,target=/pulsar/data \
--mount source=pulsarconf,target=/pulsar/conf \
apachepulsar/pulsar:2.8.0 \
bin/pulsar standalone
```

## 二、 概念与架构

> ​	Pulsar 是一个用于服务器到服务器的消息系统，具有多租户、高性能等优势。 Pulsar 最初由 [Yahoo](http://yahoo.github.io/) 开发，目前由 [Apache 软件基金会](https://www.apache.org/)管理。
> Pulsar 的关键特性如下：
>
> - Pulsar 的单个实例原生支持多个集群，可[跨机房](https://pulsar.apache.org/docs/zh-CN/administration-geo)在集群间无缝地完成消息复制。
> - 极低的发布延迟和端到端延迟。
> - 可无缝扩展到超过一百万个 topic。
> - 简单的[客户端 API](https://pulsar.apache.org/docs/zh-CN/concepts-clients)，支持 [Java](https://pulsar.apache.org/docs/zh-CN/client-libraries-java)、[Go](https://pulsar.apache.org/docs/zh-CN/client-libraries-go)、[Python](https://pulsar.apache.org/docs/zh-CN/client-libraries-python) 和 [C++](https://pulsar.apache.org/docs/zh-CN/client-libraries-cpp)。
> - 支持多种[ topic 订阅模式](https://pulsar.apache.org/docs/zh-CN/concepts-messaging#subscription-modes)（[独占订阅](https://pulsar.apache.org/docs/zh-CN/concepts-messaging#exclusive)、[共享订阅](https://pulsar.apache.org/docs/zh-CN/concepts-messaging#shared)、[故障转移订阅](https://pulsar.apache.org/docs/zh-CN/concepts-messaging#failover)）。
> - 通过 [Apache BookKeeper](http://bookkeeper.apache.org/) 提供的[持久化消息存储机制](https://pulsar.apache.org/docs/zh-CN/concepts-architecture-overview#persistent-storage)保证消息传递。
> - 由轻量级的 serverless 计算框架 [Pulsar Functions](https://pulsar.apache.org/docs/zh-CN/functions-overview) 实现流原生的数据处理。
> - 基于 Pulsar Functions 的 serverless connector 框架 [Pulsar IO](https://pulsar.apache.org/docs/zh-CN/io-overview) 使得数据更易移入、移出 Apache Pulsar。
> - [分层式存储](https://pulsar.apache.org/docs/zh-CN/concepts-tiered-storage)可在数据陈旧时，将数据从热存储卸载到冷/长期存储（如S3、GCS）中。











