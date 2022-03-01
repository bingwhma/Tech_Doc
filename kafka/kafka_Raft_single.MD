# 下载并且解压它。

tar -xzf kafka_2.13-3.0.0.tgz
cd kafka_2.13-3.0.0

# 生成集群id，并格式化存储目录

生成集群ID
bin/kafka-storage.sh random-uuid

格式化存储目录
> bin/kafka-storage.sh format -t 生成集群ID -c config/kraft/server.properties

输出：
Formatting /tmp/XXXXXXXXXX

如果是多节点安装，确保每个节点使用的集群ID是相同的。

# 启动服务

    注意：你的本地环境必须安装有Java 9+。

bin/kafka-server-start.sh config/kraft/server.properties &

一旦成功启动，那Kafka已经可以使用了。

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

> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning --consumer.config config/consumer.properties
This is a message
This is another message

如果你有2台不同的终端上运行上述命令，那么当你在运行生产者时，消费者就能消费到生产者发送的消息。

是的，整个过程没有用到zookeeper。
