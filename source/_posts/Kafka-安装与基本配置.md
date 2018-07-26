---
title: Kafka 安装与基本配置
date: 2018-06-06 10:59:06
tags:
- Kafka
categories:
- Tech
---

Kafka 是使用 Java 开发的应用程序，可以在安装了 Java 环境的多种操作系统上运行。另外 Kafka 使用 ZooKeeper 保存 broker 的元数据。

在配置好 Java 环境和 ZooKeeper 之后，就可以开始下载并安装 Kafka 了。





<!-- more -->

## 安装 Kafka Broker

在 [Kafka 官网](http://kafka.apache.org/) 可以下载到最新版本的 Kafka，如 `kafka_2.11-1.1.0.tgz` 。使用命令

```shell
$ tar -zxf kafka_2.11-1.1.0.tgz
```

解压文件，在启动 ZooKeeper 后进入 Kafka 主目录，通过命令启动

```shell
$ bin/kafka-server-start.sh config/server.properties
```

启动 Kafka。





## broker 的基本配置

在单机调试时可以使用默认配置，如果要在正式环境下部署 broker 集群，需要对配置进行调整。



### broker.id

broker 的标识符，默认为 0，可以设置为任意整数，在整个 Kafka 集群内唯一。

建议设置成与机器名相关的整数。



### port

Kafka 监听的端口号，默认为 9092，可以设置成任意可用端口号。



### zookeeper.connect

用于存储 broker 元数据的 ZooKeeper 地址，默认为 localhost:2181，可以按照 `hostname:port/path` 格式进行配置，多个地址用冒号 `:` 分隔。

推荐配置 path 作为 Kafka 集群的 chroot 环境，如不配置则默认使用 ZooKeeper 根目录。



### log.dirs

Kafka 使用磁盘保存消息，log.dirs 指定存放消息的路径，默认为 /tmp/kafka-logs，多个地址使用逗号 `,` 分隔。

同一个分区的数据会保存到同一个路径下。broker 根据 `最少使用` 原则，向拥有最少分区的路径下添加分区数据，而不是向拥有最小磁盘空间的路径添加分区数据。



### num.recovery.threads.per.data.dir

broker 使用可配置的线程池来完成启动、关闭或崩溃时重启时的操作，对于包含大量分区的服务器，并行操作可能会节省大量时间。需要注意的是，该配置针对单个目录，最终使用的线程数是该配置与目录数的乘积。



### auto.create.topics.enable

默认情况下，broker 对不存在的主题进行操作时会自动创建主题，即向不存在的主题写入消息、读取消息或请求主题元数据。如果需要控制主题的创建可以将该配置改为 false。





## topic 的基本配置

Kafka 可以通过管理工具对每个主题进行单独的配置，如分区个数、数据保留策略等，broker 提供的默认的配置适用于常规场景，可以作为基准。



### num.partitions

该参数指定新创建的主题包含的分区个数，默认为 1。Kafka 通过分区对主题进行横向扩展，当有新的 broker 加入时，可以通过分区来实现集群的负载均衡。

为了使分区能够分布到所有 broker 上，分区个数必须大于 broker 个数。如需要对包含大量消息的主题进行负载均衡，就需要大量的分区。

通常可以用主题吞吐量除以消费者吞吐量估算出需要分区的个数，即如果每秒对主题写入和读取 1 GB 的数据，每个消费者每秒可以处理 50 MB，则至少需要 20 个分区。如果无法确定吞吐量，根据经验，单个分区的数据量应该控制在 25 GB 以内。



### log.retention.ms

数据保留的毫秒数。Kafka 默认使用 log.retention.hours 来配置数据保留时间，默认值为 168 小时，即 1 周。推荐使用 log.retention.ms 来配置数据保留时间。

当同时配置多个数据保留时间，会优先使用具有最小值的那个参数。



### log.retention.bytes

该配置作用于每一个分区，表示保留的最大数据量。比如该配置为 1 GB 时，一个包含 8 个分区的主题最多可以保留 8 GB 的数据量。

此配置和 log.retention.ms 等数据保留时间的配置，在任意一个条件到达边界时删除旧数据。



### log.segment.bytes

Kafka 以日志片段的形式存储数据，前面提到的 log.retention.ms 和 log.retention.bytes 都作用于日志片段而不是单条消息。log.segment.bytes 指定单个日志片段的大小，默认为 1 GB。消息写入到日志片段，到达日志片段上限时关闭该片段，并开启新片段。

日志片段关闭前，片段内的消息是不会过期的。如果片段上限为 1 GB，每天接收 100 MB 消息，数据保留时间为 1 周，则日志片段最多需要 17 天才会被删除。



### log.segment.ms

该配置指定日志片段开启后多长时间关闭片段，没有默认值。此配置和 log.segment.bytes 一起使用时，任意一个发生作用时关闭片段并启用新的日志片段。



### message.max.bytes

限制单个消息大小，默认为 1000000，即 1 MB。当生产者尝试发送大于此配置的消息时，消息不会被接受，返回错误信息。此配置只压缩后的消息大小，实际大小可以大于配置的值。





## broker 集群

使用集群可以跨服务器进行负载均衡，还可以通过复制功能避免单点故障造成的数据丢失，提供高可用性。



### broker 个数

判断所需要的 broker 个数首先需要考虑数据量，如果整个集群要保留 10 TB 的数据，单个 broker 可以存储 2 TB，则至少需要 5 个 broker。如果启用数据复制，则至少还需要一倍的空间，即至少需要 10 个 broker。

另外需要考虑考虑集群处理请求的能力，通过增加 broker 可以提高每秒处理请求的上限，也可以解决因磁盘吞吐量低或内存不足等造成的性能问题。



### broker 集群配置

要把 broker 加到集群中，只需修改两个配置参数。

一是修改 zookeeper.connect ，所有 broker 需使用相同的 ZooKeeper 地址来保存元数据。

二是修改 broker.id 的值，broker.id 需在集群内唯一，否则将无法启动。






## 参考

[Kafka 官方文档](http://kafka.apache.org/documentation/)

[Kafka 权威指南](http://shop.oreilly.com/product/0636920044123.do)