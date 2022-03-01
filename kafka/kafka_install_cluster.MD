
# 设置多个broker集群

到目前，我们只是单一的运行一个broker，没什么意思。对于Kafka，一个broker仅仅只是一个集群的大小，所有让我们多设几个broker。

首先为每个broker创建一个配置文件:

> cp config/server.properties config/server-1.properties 
> cp config/server.properties config/server-2.properties

现在编辑这些新建的文件，设置以下属性：

config/server-1.properties: 
    broker.id=1 
    listeners=PLAINTEXT://:9093 
    log.dir=/tmp/kafka-logs-1

config/server-2.properties: 
    broker.id=2 
    listeners=PLAINTEXT://:9094 
    log.dir=/tmp/kafka-logs-2

broker.id是集群中每个节点的唯一且永久的名称，我们修改端口和日志目录是因为我们现在在同一台机器上运行，我们要防止broker在同一端口上注册和覆盖对方的数据。

我们已经运行了zookeeper和刚才的一个kafka节点，所有我们只需要在启动2个新的kafka节点。

> bin/kafka-server-start.sh config/server-1.properties &
... 
> bin/kafka-server-start.sh config/server-2.properties &
...

现在，我们创建一个新topic，把备份设置为：3

> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic

好了，现在我们已经有了一个集群了，我们怎么知道每个集群在做什么呢？运行命令“describe topics”

> bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic my-replicated-topic
Topic:my-replicated-topic    PartitionCount:1    ReplicationFactor:3    Configs:
Topic: my-replicated-topic    Partition: 0    Leader: 1    Replicas: 1,2,0    Isr: 1,2,0

输出解释：第一行是所有分区的摘要，其次，每一行提供一个分区信息，因为我们只有一个分区，所以只有一行。

    "leader"：该节点负责该分区的所有的读和写，每个节点的leader都是随机选择的。
    "replicas"：备份的节点列表，无论该节点是否是leader或者目前是否还活着，只是显示。
    "isr"：“同步备份”的节点列表，也就是活着的节点并且正在同步leader。

我们运行这个命令，看看一开始我们创建的那个节点：

> bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic test
Topic:test    PartitionCount:1    ReplicationFactor:1    Configs:
Topic: test    Partition: 0    Leader: 0    Replicas: 0    Isr: 0

这并不奇怪，刚才创建的主题没有Replicas，并且在服务器“0”上，我们创建它的时候，集群中只有一个服务器，所以是“0”。

让我们来发布一些信息在新的topic上：

> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-replicated-topic
 ...
my test message 1
my test message 2
^C

现在，消费这些消息。

> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
 ...
my test message 1
my test message 2
^C

我们要测试集群的容错，kill掉leader，Broker1作为当前的leader，也就是kill掉Broker1。

> ps | grep server-1.properties
7564 ttys002    0:15.91 /System/Library/Frameworks/JavaVM.framework/Versions/1.6/Home/bin/java... 
> kill -9 7564

在Windows上使用：

> wmic process where "caption = 'java.exe' and commandline like '%server-1.properties%'" get processid
ProcessId
6016
> taskkill /pid 6016 /f

备份节点之一成为新的leader，而broker1已经不在同步备份集合里了。

> bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic my-replicated-topic
Topic:my-replicated-topic    PartitionCount:1    ReplicationFactor:3    Configs:
Topic: my-replicated-topic    Partition: 0    Leader: 2    Replicas: 1,2,0    Isr: 2,0

但是，消息仍然没丢：

> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
^C
