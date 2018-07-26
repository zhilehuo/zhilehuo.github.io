---
title: Kafka 消费者
date: 2018-07-20 19:23:02
tags:
- Kafka
categories:
- Tech
---

应用程序使用 KafkaConsumer 向 Kafka 订阅主题，并从订阅的主题上接收消息。Kafka 的消费者涉及一些独特的概念，本篇文章首先解析一些重要概念，然后介绍如何读取消息及相关配置。





<!-- more -->

## 概念



### 消费者和消费者群组

Kafka 消费者从属于消费者群组，群组内的消费者订阅同一个主题，每个消费者接收主题一部分分区的消息。

如果主题 T1 有 4 个分区，消费者 C1 是订阅了 T1 的群组 G1 内的唯一消费者，则 C1 将接收到全部 4 个分区的消息。而如果 G1 内包含 2 个消费者，则每个消费者接收 2 个分区的消息；如果 G1 包含 4 个消费者，则每个消费者接收 1 个分区的消息。但如果 G1 内包含 5 个消费者，则只有 4 个消费者会分配到分区并接收消息。

在群组中增加消费者是横向伸缩消费能力的主要方式。由于当消费者数量超过分区数时，多余的消费者会被闲置，所以有必要为主题创建较多的分区，以便在负载增长时加入更多消费者。横向伸缩 Kafka 消费者和消费者群组并不会对性能造成负面影响。



### 分区再均衡

群组内的消费者共同读取主题的分区，当新的消费者加入或某个消费者被关闭时，分区会重新分配。分区所有权从一个消费者转移到另一个消费者的行为称为再均衡。

消费者通过向被指派为群组协调器的 broker 发送心跳来维持和群组的从属关系及分区所有权。消费者在轮询消息或提交偏移量时发送心跳，当停止发送心跳的时间足够长时被认为死亡，触发再均衡。

0.10.1 版本中 Kafka 引用了一个独立的心跳线程，在轮询消息的空档发送心跳。





## 读取消息

要读取 Kafka 中消息，首要需要创建 KafkaConsumer 消费者对象，然后订阅主题、轮训消息及提交偏移量。



### 创建消费者

创建 KafkaConsumer 对象和创建 KafkaProducer 对象非常类似，需要将一些必要的属性在 Properties 对象中指明。

KafkaConsumer 的必要属性包括 bootstrap.servers，key.deserializer 和 value.deserializer，分别为 Kafka 集群的地址，键和值的反序列化器。

另外 group.id 指定所属的消费者群组，不指定时消费者不输入任何一个群组。



### 订阅主题

KafkaConsumer 对象通过 subscribe 方法订阅主题，既可以订阅单个主题，也可以订阅多个主题，还可以通过正则表达式订阅相关的主题。



### 轮询

轮询是消费者 API 的核心，消费者通过轮询想发服务发送请求，开发者只需使用一组简单的 API 处理分区返回的数据，轮询会透明地处理群组协调、分区再均衡、发送心跳等细节。



### 代码示例

创建消费者对象、订阅主题并轮询消息的代码示例如下：

```java
package com.ulyssesss.kafka;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import java.util.Arrays;
import java.util.Properties;

public class Consumer {
    public static void main(String[] args) {
        Properties properties = new Properties();
        properties.put("bootstrap.servers", "localhost:9092");
        properties.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        properties.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        properties.put("group.id", "Group1");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(properties);
        consumer.subscribe(Arrays.asList("Topic1", "Topic2"));

        try {
            while (true) {
                ConsumerRecords<String, String> records = consumer.poll(100);
                for (ConsumerRecord<String, String> record : records) {
                    System.out.println(record.topic() + "\n" + record.partition() + "\n"
                            + record.offset() + "\n" + record.key() + "\n" + record.value() + "\n\n");
                }
            }
        } finally {
            consumer.close();
        }
    }
}
```

其中 poll 方法返回一个消息列表，没条记录都包含了所属主题、分区、偏移量及键值等信息。

poll 方法在消费者缓存区没有可用数据时发生阻塞，poll 方法的参数控制最大阻塞时间，达到阻塞时间上限时不论是否有可用数据都必须返回。超时时间取决于应用对响应速度的要求，即要在多长时间内把控制权归还给轮询消息的线程。

close 方法用于在退出应用前关闭消费者，网络连接和 socket 随之关闭，触发再均衡。

一个消费者应该使用一个线程，如果要在群组中运行多个消费者，需要让消费者运行在自己的线程中。最好将逻辑封装在对象中，然后使用 ExecutorService 启动多个线程并执行相应逻辑。





## 消费者的配置

Kafka 消费者大部分的配置都有合理的默认值，一般情况下不需要改动。一下为一些重要的属性，跟性能和可用性有很大关系。



### fetch.min.bytes

指定消费者从服务器获取消息的最小字节数，如果数据量小于指定的大小，会等到有足够的可用数据再返回。该配置可以在一定程度上杜绝在主题不活跃期间来来回回地处理消息，降低 broker 工作负载。



### fetch.max.wait.ms

指定最长等待时间，与 fetch.min.bytes 联合使用，如 fetch.min.bytes 指定为 1 MB，fetch.max.wait.ms 指定为 100 ms，则 broker 在收到请求后，要么返回 1 MB 的数据，要么在 100 ms 内返回所有可用数据，任一条件满足就会返回。



### max.partition.fetch.bytes

指定每个分区返回给消费者的最大字节数，默认 1 MB，即如果有 20 个分区，4 个消费者，每个消费者至少需要 5 MB 的内存来接收消息。分配内存时可以多分配一些，以便在某个消费者崩溃时其他消费者处理更多的分区。

需要注意的是该配置需要比 broker 的 max.message.size 大，否则可能消费者无法接收消息而一直挂起重试。另外需要注意处理时间，单次返回数据越多处理时间也越多，可以将此配置的值改小或延长会话过期时间。



### session.timeout.ms

指定消费者在被认为死亡之前可以与服务器断开的时间，默认 3 秒。如果在该配置指定的时间内发送心跳，就会被认为死亡并触发再均衡。heartbeat.interval.ms 指定 poll 方法发送心跳的频率，一般为 session.timeout.ms 的三分之一，要同时注意这两个配置。



### auto.offset.reset

指定消费者在读取一个没有偏移量或偏移量无效的分区时该从哪个位置开始读取，默认为 latest，即从最近记录开始读取。另一个值为 earliest，即从起始位置开始读取。



### enable.auto.commit

指定是否自动提交偏移量，默认为 true，通过 auto.commit.interval.ms 可以控制自动提交的频率。稍后会介绍几种不同的提交偏移量的方式。



### partition.assignment.strategy

指定分区分配给消费者的策略，Kafka 默认提供两种策略，参数值分别为 org.apache.kafka.clients.consumer.RangeAssignor 和 org.apache.kafka.clients.consumer.RoundRobinAssignor。

Range 策略将主题的若干个连续分区分配给消费者，如果主题 T1 和 T2 都有 3 个分区，消费者 C1 和 C2 同时订阅 T1 和 T2，由于分区在主题内独立分配，C1 可能分配到两个主题的分区 0 和分区 1，而 C2 分配到两个主题的分区 2。使用 Range 策略在分区数无法被整除时就会出现上述情况。

RoundRobin 策略会把所有分区逐个分配给消费者。一般来说，如果所有消费者订阅相同的主题，RoundRobin 策略会尽量给所有消费者分配相同数量的分区。



### client.id

用于标示客户端，可以为任意字符串。



### max.poll.records

用于控制单次返回的消息数量。



### receive.buffer.bytes 和 send.buffer.bytes

用于设置 TCP 缓冲区大小，-1 时使用操作系统默认值。





## 提交偏移量

Kafka 通过偏移量判断消息是否被读取过。消费者向一个特殊的主题 _consumer_offset 发送消息，消息中包括分区的偏移量。提交偏移量在消费者一直处于运行状态时没什么用，不过在消费者崩溃或有新的消费者加入群组并触发再均衡时，消费者通过最后提交的偏移量来从指定位置继续处理。



### 自动提交

默认情况下 enable.auto.commit 配置的值为 true，效果为每 5 秒消费自动把 poll 方法接收到的最大偏移量提交上去。提交时间间隔可以通过 auto.commit.interval.ms 控制。自动提交发生在轮询时，轮询时检查是否该提交偏移量。

由于自动提交为每隔一段时间提交一次，假如在上一次提交几秒后发生再均衡，均衡后从最后一次提交的偏移量位置处理消息，则会出现重复处理消息的现象。自动提交虽然方便，可是无法解决重复处理消息的问题。



### 提交当前偏移量

消费者 API 提供了主动提交当前偏移量的方法，通过控制提交时间来减少重复消息。

首先将 auto.commit.offset 设为 false，然后通过 commitSync 方法提交由 poll 方法返回的最新偏移量，提交成功后立即返回，提交失败则抛出异常。由于 commitSync 方法提交的为 poll 方法返回的最新偏移量，所以要在处理完所有消息后调用方法。



### 异步提交

通过 commitSync 方法提交偏移量在 broker 回应之前会阻塞程序，限制吞吐量。通过异步方式 commitAsync 提交，可以只管发送提交请求无需等待。由于 commitAsync 方法会异步提交偏移量，所以不会像 commitSync 方法一样在失败后重试。



### 同步与异步组合提交

通常情况下，偶尔出现提交失败不重试也不会造成影响，但如果发生在关闭消费者或再均衡之前的最后一次提交，就要确保能够提交成功。可以组合使用 commitSync 和 commitAsync 方法，在处理完一批消息后异步提交，在跳出循环后同步提交，代码如下：

```java
try {
    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(100);
        for (ConsumerRecord<String, String> record : records) {
            System.out.println(record.value());
        }
        consumer.commitAsync();
    }
} catch (Exception e) {
    e.printStackTrace();
} finally {
    try {
        consumer.commitSync();
    } finally {
        consumer.close();
    }
}
```



### 提交特定偏移量

前面提到的提交方式无法提交批次中间的偏移量，而在调用 commitAsync 和 commitSync 方法时，通过传入包含分区和偏移量的 map 来提交特定的偏移量，所提交的偏移量为期望在下一次进行处理的偏移量。





## 再均衡监听器

消费者在退出或进行分区再均衡之前，可能会做一些如关闭数据库连接的清理操作，在调用 subscribe 方法时传入 ConsumerRebalanceListener 实例可以实现再均衡的监听。

ConsumerRebalanceListener 有两个方法，onPartitionsRevoked 在停止读取消息之后、再均衡开始之前被调用；onPartitionsAssigned 会在重新分配分区后、消费者开始读取消息之前被调用。可以在 onPartitionsRevoked 方法中提交偏移量的 map 来记录已经处理过的消息。





## 从特定偏移量处理消息

消费者提供了从特定偏移量处理消息的 API。如果想从分区的起始位置或末尾开始处理消息，可以直接调用 seekToBeginning 或 seekToEnd 方法。如果想从其他偏移量处理消息，可以使用 seek 方法并传入分区信息、偏移量作为参数。

Kafka 不支持处理消息和提交偏移量在一个原子操作中完成，而如果将偏移量记录在数据库中，再均衡时从数据库中记录的位置开始处理数据，就能使处理消息和提交偏移量要么都成功、要么都失败。





## 退出轮询

消费者如果要退出轮询，需要通过另一个线程调用 consumer.wakeup() 方法，退出 poll 并抛出 WakeupException。如果调用时线程没有等待轮询，则在下一轮调用 poll 时抛出。无需处理 WakeupException，只是通过它来跳出循环。





## 独立消费者

如果消费者只需从一个主题的所有分区或特定几个分区读取数据，就不需要加入消费者群组，而是直接通过 assign 方法为自己分配分区。注意不能同时为自己分配分区和加入消费者群组订阅主题。






## 参考

[Kafka 官方文档](http://kafka.apache.org/documentation/)

[Kafka 权威指南](http://shop.oreilly.com/product/0636920044123.do)