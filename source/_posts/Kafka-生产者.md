---
title: Kafka 生产者
date: 2018-06-07 11:46:47
tags:
- Kafka
categories:
- Tech
---

不管把 Kafka 用作消息队列还是数据存储平台，总需要向 Kafka 写入消息的生产者和从 Kafka 读取消息的消费者。

本篇文章介绍如何创建生产者和消息对象、如何发送消息到 Kafka、生产者的相关配置，以及如何使用分区。





<!-- more -->

## 创建生产者和消息对象

要往 Kafka 写入消息，首先要创建一个生产者对象，并设置一些必要的属性。Kafka 生产者有 3 个必选属性。



* bootstrap.servers

指定 broker 的地址，格式为 `host:port` ，多个地址用逗号 `,` 分隔。建议至少提供两个 broker 地址，其中一台宕机仍然可以连接到集群上。



* key.serializer

broker 接收到的消息的键为字节数组，而生产者接口允许使用参数化类型，可以把 Java 对象作为键，所以需要知道如何将 Java 对象转为字节数组。

key.serializer 要求设置为一个实现了 `org.apache.kafka.common.serialization.Serializer` 接口的类。客户端默认提供了 `ByteArraySerializer` , `StringSerializer` 和 `IntegerSerializer` 。



* value.serializer

和 key.serializer 一样，value.serializer 指定的类用于序列化消息的值。



创建消息对象需要指明主题 topic 和值 value，分区 partition 和键 key 可选。指定分区时消息在发送后会直接到达所选分区，不指定分区时会根据键选择一个分区，都不指定时均匀分布到各个分区。



创建生产者和消息对象的代码示例如下：

```java
package com.ulyssesss.kafka;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import java.util.Properties;

public class Producer {
    public static void main(String[] args) {
        Properties properties = new Properties();
        properties.put("bootstrap.servers", "localhost:9092");
        properties.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        properties.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(properties);
        ProducerRecord<String, String> record = new ProducerRecord<String, String>("topic", "key", "value");
    }
}
```





## 发送消息

发送消息主要有 3 中方式，即发送并忘记（fire-and-forget）、同步发送和异步发送。



### 发送并忘记

将消息发送给服务器，但不关心消息是否正常到达。大多数时候消息会正常到达，因为集群高可用，并且生产者会在发送失败时自动尝试重发。不过仍然有消息丢失的可能。记录不太重要的日志可以使用这种发送方式。



### 同步发送

发送消息时，生产者的 send 方法会先返回一个 Future 对象，调用 Future 的 get 方法等待 Kafka 响应。如果服务器返回错误，get 方法会抛出异常，否则会得到一个 RecordMetadata 对象。

生产者可能发生两种错误，一种是连接异常等可以通过重发消息来解决的错误，Kafka 可以配置成自动重试，如果重试多次仍存在问题，会收到重试异常。另一种是消息过大等无法通过重试解决的错误，生产者不会进行重试直接抛出异常。



### 异步发送

大多数时候我们不需要等待响应，不过在遇到消息发送失败的情况时，需要进行抛出异常、记录错误日志等操作。生产者支持异步发送消息的回调，需要在发送消息时传入实现了 `org.apache.kafka.clients.producer.Callback` 接口的回调对象。



使用 3 中方式发送消息的代码示例如下：

```java
package com.ulyssesss.kafka;

import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;
import java.util.Properties;
import java.util.concurrent.ExecutionException;

public class Producer {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        Properties properties = new Properties();
        // ... properties 添加 producer 配置
        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(properties);
        ProducerRecord<String, String> record = new ProducerRecord<String, String>("topic", "key", "value");

        //发送并忘记
        producer.send(record);

        //同步发送
        RecordMetadata result = producer.send(record).get();

        //异步发送
        producer.send(record, new Callback() {
            public void onCompletion(RecordMetadata metadata, Exception exception) {
                if (exception != null) {
                    exception.printStackTrace();
                }
            }
        });
    }
}
```







## 生产者的配置

前面在创建生产者时只配置了 3 个必要参数，生产者还有很多可配置的参数，都有合理的默认值。下面介绍一些常用的配置，在内存使用、性能和可靠性方面对生产者影响较大。



### acks

指定需要有多少个分区副本收到消息，生产者才认为消息写入成功。

acks=0 时生产者不会等待来自任何服务器的响应，如果当中出了任何问题消息都会丢失，不过能够以最大速度发送消息。acks=1 时只要集群首领节点收到消息生产者就会收到服务器的成功响应。acks=all 时需要等全部参与复制的节点都收到消息时才有成功响应。



### buffer.memory

生产者内存缓冲区大小，用于缓冲要发送到服务器的消息。当发送消息的速度超过发送到服务器的速度，会在内存中形成消息堆积，当堆积的消息大小超过缓冲区大小时，根据 max.block.ms 所配置的阻塞时间决定被阻塞还是抛出异常。



### compression.type

默认情况下消息发送时不会被压缩，改配置可以设置为 snappy、gzip 或 lz4，指定发送给 broker 前采用的压缩算法。snappy 占用较少传递 CPU，能够提供较好的性能好压缩比。gzip 占用较多 CPU，提供更高的压缩比。



### retries

生产者收到服务器返回的错误时，如果为临时性错误，会依据该配置决定可以重试的次数。如果达到所配置的重试次数依然没有成功，则放弃并返回错误。默认的重试间隔为 100 ms，可通过 retry.backoff.ms 来调整。

因为生产者会自动进行重试，所以只需要处理不可重试的错误和重试次数到达上线的情况。



### batch.size

多个消息要被发送到同一分区时，生产者会将消息放到同一个批次中统一发送。该参数指定批次可以使用的内存字节数，批次被填满时发送所有消息。不过并非要等到批次填满才会发送，只包含一条消息的批次也可能被发送。就算批次大小设置的很大，也不会造成延迟，只会占用更多的内存。



### linger.ms

指定等待更多消息加入批次的时间。生产者在批次填满或等待 linger.ms 所配置的时间后发送批次。默认情况下只要有可用的线程就会立即发送消息。设置为大于 0 的数虽然会增加延迟，但能明显提升吞吐量。



### client.id

任意字符串，用于标记消息来源。



### max.in.flight.requests.per.connection

指定生产者在收到服务器响应之前可以发送多少次消息，为 1 可以保证按照发送顺序写入到服务器，即使发生了重试。该参数的值越高，占用内存越大，吞吐量也越大。



### timeout.ms、request.timeout.ms 和 metadata.fetch.timeout.ms

分别指定 broker 间同步副本等待响应的超时时间，生产者发送数据等待响应的超时时间和生产者获取元数据（如目标分区的首领是哪个 broker）等待服务器响应的超时时间。



### max.block.ms

该参数指定在调用 send 方法或使用 partitionsFor 方法获取元数据时的最大阻塞时，阻塞时间达到 max.block.ms 时生产者会抛出异常。



### max.request.size

用于控制生产者发送请求的大小，即可以指单个消息，也可以指批次内所有消息的总大小。broker 也有对消息大小的限制，最好配置可以匹配，避免请求被 broker 拒绝。



### receive.buffer.bytes 和 send.buffer.bytes

指定 TCP socket 接收和发送数据包的缓冲区大小，为 -1 时使用操作系统的默认值。如果生产者与 broker 处于不同的数据中心，可以适当增大配置。





## 如何使用分区

ProducerRecord 对象可以只包含主题和值，键默认为 null。键既可以作为消息的附加信息，也可以用于决定消息被写入哪个分区。

键为 null 时，如使用默认分区器，则通过轮训将消息随机写入到各个可用分区。键不为空时，默认分区器会使用散列算法将消息映射到所有分区上。

如果需要自定义分区器，可以实现 Partitioner 接口，并重载 partition 方法，实例代码如下：

```java
package com.ulyssesss.kafka;
import org.apache.kafka.clients.producer.Partitioner;
import org.apache.kafka.common.Cluster;
import java.util.Map;

public class MyPartitioner implements Partitioner {
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        //分区逻辑
        return 0;
    }
    public void close() {}
    public void configure(Map<String, ?> configs) {}
}
```

实现自定义分区器后，需要在生产者的配置中指明所用的分区器。

```java
properties.put("partitioner.class", "com.ulyssesss.kafka.MyPartitioner");
```





## 参考

[Kafka 官方文档](http://kafka.apache.org/documentation/)

[Kafka 权威指南](http://shop.oreilly.com/product/0636920044123.do)

