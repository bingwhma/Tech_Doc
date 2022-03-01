

# 下载并且解压它。

> tar -xzf kafka_2.13-3.1.0.tgz
> cd kafka_2.13-3.1.0

# 启动服务
    注意：你的本地环境必须安装有Java 8+。

运行kafka需要使用Zookeeper，所以你需要先启动Zookeeper，如果你没有Zookeeper，你可以使用kafka自带打包和配置好的Zookeeper。

# 注意：Apache Kafka2.8版本之后可以不需要使用ZooKeeper，内测中，文章末尾有体验的安装方式。
> bin/zookeeper-server-start.sh config/zookeeper.properties
...

打开另一个命令终端启动kafka服务：

> bin/kafka-server-start.sh config/server.properties &

一旦所有服务成功启动，那Kafka已经可以使用了。

# 创建一个主题(topic)

创建一个名为“test”的Topic，只有一个分区和一个备份：

bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test

创建好之后，可以通过运行以下命令，查看已创建的topic信息：

> bin/kafka-topics.sh --describe --topic test --bootstrap-server localhost:9092
Topic:test  PartitionCount:1    ReplicationFactor:1 Configs:
Topic: test Partition: 0    Leader: 0   Replicas: 0 Isr: 0

或者，除了手工创建topic外，你也可以配置你的broker，当发布一个不存在的topic时自动创建topic，点击这里查看如何配置自动创建topic时设置默认的分区和副本数。
# 发送消息

Kafka提供了一个命令行的工具，可以从输入文件或者命令行中读取消息并发送给Kafka集群。每一行是一条消息。

运行 producer（生产者）,然后在控制台输入几条消息到服务器。

> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
This is a message
This is another message

# 消费消息

Kafka也提供了一个消费消息的命令行工具，将存储的信息输出出来，新打开一个命令控制台，输入：

> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
This is a message
This is another message

如果你有2台不同的终端上运行上述命令，那么当你在运行生产者时，消费者就能消费到生产者发送的消息。

    可以使用Ctrl-C停止消费者客户端。

# 使用 Kafka Connect 来 导入/导出 数据

你可能在现有的系统中拥有大量的数据，如关系型数据库或传统的消息传递系统，以及许多已经使用这些系统的应用程序。Kafka Connect允许你不断地从外部系统提取数据到Kafka，反之亦然。用Kafka整合现有的系统是非常容易的。为了使这个过程更加容易，有数百个这样的连接器现成可用。

查看Kafka Connect部分，了解更多关于如何将你的数据持续导入和导出Kafka。
# 使用Kafka Stream来处理数据

一旦你的数据存储在Kafka中，你就可以用Kafka Streams客户端库来处理这些数据，该库适用于Java/Scala。它允许你实现自己的实时应用程序和微服务，其中输入和/或输出数据存储在Kafka主题中。Kafka Streams将在客户端编写和部署标准Java和Scala应用程序的简单性与Kafka服务器端集群技术的优势相结合，使这些应用程序具有可扩展性、弹性、容错性和分布式。该库支持精确的一次性处理、有状态操作和聚合、窗口化、连接、基于事件时间的处理等等。

为了给你一个初步的体验，这里是如何实现流行的WordCount算法的：

KStream<String, String> textLines = builder.stream("quickstart-events");

KTable<String, Long> wordCounts = textLines
            .flatMapValues(line -> Arrays.asList(line.toLowerCase().split(" ")))
            .groupBy((keyIgnored, word) -> word)
            .count();

wordCounts.toStream().to("output-topic", Produced.with(Serdes.String(), Serdes.Long()));

Kafka流演示例子和应用开发教程展示了如何从头到尾编码和运行这样一个流式应用。

# 停止Kafka

现在你已经完成了快速入门，可以随时卸载Kafka环境了，或者继续玩下去。

    使用Ctrl-C停止生产者和消费者客户端。
    使用Ctrl-C停止Kafka broker。
    最后，用Ctrl-C停止ZooKeeper。

如果你还想删除你的本地Kafka环境的数据，包括你创建的消息，运行以下命令：

$ rm -rf /tmp/kafka-logs /tmp/zookeeper
