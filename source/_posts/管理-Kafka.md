---
title: 管理 Kafka
date: 2018-07-25 11:31:12
tags:
- Kafka
categories:
- Tech
---

Kafka 提供了一些用于管理集群变更的命令行工具，这些工具由 Java 实现，Kafka 提供一些脚本来调用这些 Java 类。这些工具只提供一些基本的功能，无法完成复杂操作。Kafka 社区有很多高级工具，如 Kafka Manager。





<!-- more -->

## 一、主题操作

使用 kafka-topics.sh 可以执行主题的大部分操作，如创建、修改、删除和查看集群内的主题（配置变更部分已经移动到 kafka-configs.sh 中），使用时需要通过 --zookeeper 参数提供 ZooKeeper 地址，如 zoo1.example.com:2181/kafka。



### 1.创建主题

在集群内创建主题，需要指定主题名、复制系数和分区数 3 个必要参数，主题名建议不适用英文句号及下划线（会用于度量指标），格式为 `kafka-topics.sh --zookeeper <zookeeper connect> --create --topic <topic name> --replication-factor <integer> --partitions <integer>` 。

如果不需要基于机架信息的分配分区，可以指定参数 `--disable-rack-aware` 。添加 `--if-not-exists` 参数可以仅在主题不存在时创建主题。



### 2.增加分区

主题基于分区进行伸缩和复制，当需要扩展主题容量或降低单个分区吞吐量时可以增加分区，如果消费者群组内需要增加消费者，主题的分区数也需要增加至大于组内消费者数。

对于基于键来划分消息分区的主题，改变分区数会导致键和分区的映射关系发生变化，对于这种主题，最好在一开始就确定好分区数，尽量不要调整。

调整主题分区数的命令格式为 `kafka-topics.sh --zookeeper <zookeeper connect> --alter --topic <topic name> --partitions <integer>` ，`--if-exists` 参数可以在主题不存在时忽略错误。

通常情况下无法减少主题的分区数，如果删除了分区，分区内的数据也会被删除，导致数据不一致。如果一定要减少分区数，只能删除整个主题，然后重新创建。



### 3.删除主题

如果一个主题不再使用，可以删除主题从而减少磁盘占用、释放资源。要删除主题，broker 的 delete.topic.enable 参数必须为 true，否则删除请求会被忽略。

删除命令的格式为 `kafka-topics.sh --zookeeper <zookeeper connect> --delete --topic <topic name>` 。



### 4.列出集群内主题

`kafka-topics.sh --zookeeper <zookeeper connect> --list` 用于列出集群内的所有主题，将 `--list` 参数改为 `--describe` 参数会列出包含分区数等信息的主题。指定 `--topic <topic name>` 可以用于主题过滤。

如果要找出所有包含覆盖配置的主题可以添加 `--topics-with-overrides` 参数，`under-replicated-partitions` 参数用于列出包含不同步副本的分区，`--unavailable-partition` 参数可以列出所有没有首领的分区。注意这些参数都不能和 `--topic` 参数一起使用。





## 二、消费者群组操作

Kafka 提供 kafka-consumer-groups.sh 工具来列出消费者群组信息，对于旧版的 Kafka 还可以删除消费者群组和管理偏移量。新版 Kafka 的消费者群组信息保存在 broker 中，使用命令行工具时需要指定 --bootstrap-server；旧版 Kafka 的消费者群组信息保存在 ZooKeeper 中，使用时需指定 --zookeeper。



### 1.列出并描述群组

通过命令 `kafka-consumer-groups.sh --bootstrap-server <server connect> --list` 可以列出所有的消费者群组，将 `--list` 改为 `--describe` ，并通过 `--group <group name>` 参数指定特定的群组，可以获得该群组更详细的信息。



### 2.删除群组

旧版本的 Kafka 支持使用命令行工具删除群组，也支持在不删除群组的情况下删除单个主题的偏移量。注意，删除群组或偏移量前必须关闭组内所有消费者，否则可能会导致消费者不可预测的行为。

删除群组的命令格式为 `kafka-consumer-groups.sh --zookeeper <zookeeper connect> --delete --group <group name>` 。删除主题偏移量的命令格式为 `kafka-consumer-groups.sh --zookeeper <zookeeper connect> --delete --group <group name> --topic <topic name>` 。



### 3.管理偏移量

旧版本的 Kafka 将偏移量保存在 ZooKeeper 的 /consumers 路径下，通过命令 `kafka-run-class.sh kafka.tools.ExportZkOffsets --zkconnect <zk connect> --group <group name> --output-file <file name>` 可以导出偏移量到文件，而通过命令 `kafka-run-class.sh kafka.tools.ImportZkOffsets --zkconnect <zk connect> --group <group name> --input-file <file name>` 可以将偏移量导入。

注意导入时必须关闭所有消费者，否则有可能将导入的偏移量覆盖掉。

新版本的 Kafka 将偏移量保存在一个内部主题 __consumer_offsets 中，目前没有工具管理客户端提交到 Kafka 的偏移量，只能通过客户端使用相应 API 提交群组偏移量。





## 三、动态配置变更

在集群处于运行状态时，可以通过 kafka-configs.sh 来覆盖主题及客户端的配置参数，一旦设置完毕，就会成为集群的永久配置，保存在 ZooKeeper 中，broker 会读取它们。



### 1.覆盖主题的默认配置

为了满足不同的使用场景，主题可以进行单独的配置，配置在没有被覆盖的情况下使用 broker 的默认配置。更改主题配置的命令格式为 `kafka-configs.sh --zookeeper <zk connect> --alter --entity-type topics --entity-name <topic name> --add-config <key>=<value>[,<key>=<value>...]` 。



### 2.覆盖客户端的默认配置

对于 Kafka 客户端，只能覆盖生产者和消费者的配额，即每秒在单个 broker 上的生产速率和消费速率，如有 2 个 broker，配额为 10 MB/s，则生成数据的总速率为 20 MB/s。

修改客户端配置的命令格式为 `kafka-configs.sh --zookeeper <zk connect> --alter --entity-type clients --entity-name <client-id> --add-config <key>=<value>[,<key>=<value>...]` 。可选参数有 producer_byte_rate 和 consumer_byte_rate。



### 3.列出被覆盖的配置

此功能通过 --describe 来实现，格式为 `kafka-configs.sh --zookeeper <zk connect> --describe --entity-type topics --entity-name <topic name>` 。该命令只能显示被覆盖的配置，包含 broker 的默认配置。



### 4.移除被覆盖的配置

动态配置可以被移除，恢复到集群的默认配置，命令格式为 `kafka-configs.sh --zookeeper <zk connect> --alter --entity-type topics --entity-name <topic name> --delete-config <config key>` 。





## 四、分区管理

Kafka 提供了两个用于管理分区的工具，一个用于重新选举首领，另一个用于将分区分配给 broker。



### 1.首领选举

broker 有一个配置 auto.leader.rebalance.enable 用于启用自动首领再均衡，即如果当前首领不是首选首领会自动触发再均衡，不过不建议在生产环境使用此功能，在大型集群中可能带来严重的性能问题。

Kafka 提供命令行工具来手动触发副本选举，命令格式为 `kafka-perferred-replica-election.sh --zookeeper <zk connect>` 。

进行选举时，集群元数据需要写入 ZooKeeper，如果元数据超过节点允许的大小（默认 1 MB）就会选举失败。这种情况需要将分区清单的信息写到一个 JSON 文件中，通过参数 `--path-to-json <json file>` 分多个步骤进行。JSON 文件的格式为：

```json
{
    "partitions": [
        {
            "partition": 1,
            "topic": "t1"
        },
        {
            "partition": 2,
            "topic": "t2"
        }
    ]
}
```



### 2.修改分区副本

主题的分区在集群内分配不均衡时会造成负载的不均衡，Kafka 提供了 kafka-reassign-partitions.sh 工具来修改分区分布。首先需要根据 broker 清单和主题清单生成一组迁移步骤，然后执行迁移步骤，最后（可选）使用生成的迁移步骤验证分区重新分配的进度和完成情况。

为了生成迁移步骤，首先需要创建一个包含了主题清单的 JSON 文件，格式如下：

```json
{
    "topics": [
        {
            "topic": "t1"
        },
        {
            "topic": "t2"
        }
    ]
}
```

使用命令 `kafka-reassign-partitions.sh --zookeeper <zk connect> --generate --topics-to-move-json-file <json file> --broker-list <brokers>` 会得到两个 JSON，其中 brokers 为使用英文逗号分隔的 broker id。第一个为当前分配情况，可以用于回滚；第二个为分配步骤。

使用命令 `kafka-reassign-partitions.sh --zookeeper <zk connect> --execute --reassignment-json-file <josn file>` 来执行第二个步骤。  

重分配完成后可以使用命令 `kafka-reassign-partitions.sh --zookeeper <zk connect> --verify --reassignment-json-file <json file>` 来验证重分配情况。



### 3. 修改复制系数

分区重分配工具提供了修改分区复制系数的功能，不过没有在文档中说明。如果在创建分区时指定了错误的复制系数，可以通过创建一个 JSON 对象并通过分区重分配工具完成对复制系数的修改。JSON 的格式如下：

```json
{
    "partitions": [
        {
            "partition": 0,
            "replicas": [
                1,
                2
            ],
            "topic": "t1"
        },
        "version": 1
    ]
}
```



### 4.查看日志片段

通过命令 `kafka-run-class.sh kafka.tools.DumpLogSegments --file <file>` 可以在不读取消息的情况下查看消息内容，接收以逗号分隔的日志片段文件清单为参数，打印出每个消息的概要信息及数据内容。

这个工具也会检查日志片段的索引文件，--index-sanity-check 将会检查无用的索引，--verify-index-only 将会只检查索引匹配而不打印出全部索引。



### 5.副本验证

kafka-replica-verification.sh 工具用于验证集群副本分区一致性，从指定分区的副本上获取消息并检查所有副本是否具有相同的消息，使用时需使用正则表达式传入要验证的主题，否则会检查所有的主题，另外需要显示地提供 broker 清单，格式为 `kafka-replica-verification.sh --broker-list <broker list> --topic-white-list <regexp>` 。





## 五、控制台生产者和消费者

有时为了验证应用程序，可以使用命令行工具 kafka-console-producer.sh 和 kafka-console-consumer.sh 手动生成和读取消息，不过不应该在生产环境采用此种方式，控制台生产者和消费者无法使用客户端的所有特性，本篇文章不对这部分做详细介绍。





## 六、不安全操作

有些操作在技术上是可行的，但涉及 ZooKeeper 上的元数据，有可能给应用程序带来风险，除非情况紧急，否则不应该使用。



### 1.移动集群控制器

有时需要将控制器从一个 broker 移到另一个 broker 上，如当前控制器网络波动大无法提供正常服务。手动删除 /controller 节点会释放当前控制器，集群会进行新的控制器选举。



### 2.取消分区重分配

分区重分配是并行进行的，一般情况下没理由取消任务，不过如果在重分配进行一半时，broker 发生故障且无法重启，就会导致重分配无法结束，这是可以让集群忽略这个任务，即从 ZooKeeper 中删除 /admin/reassign_partitions 节点并重新选举控制器（删除 /controller 节点）。

注意，取消重分配时，旧的 broker 不会在副本清单中删除，导致部分分区的复制系数比正常的大。如果主题的分区包含不同的复制系数，broker 会禁止对其操作（如增加分区），所以需要在取消任务后检查分区可用性，确保复制系数正确。



### 3.移除待删除的主题

使用命令行删除主题，会在 ZooKeeper 上创建一个节点作为删除主题的请求，正常会立即执行这个请求，但如果集群没有开启删除主题的功能，请求会被挂起。删除 /admin/delete_topic 节点下创建的以删除主题名字命名的子节点即可移除被挂起的请求。



### 4.手动删除主题

如果集群禁用了删除主题的功能，或者需要通过非正式的途径删除主题，可以进行手动删除。

手动删除前，首先需要关闭集群内所有的 broker，否则会造成集群不稳定。然后删除 /brokers/topics/TOPICNAME 路径（先删除其子节点）。接着删除每个 broker 的分区目录（log-dirs），名称为 `<topic name>-<partition id>`。最后重启所有 broker。





## 七、参考

[Kafka 官方文档](http://kafka.apache.org/documentation/)

[Kafka 权威指南](http://shop.oreilly.com/product/0636920044123.do)