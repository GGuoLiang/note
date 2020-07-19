
# kafka概念

kafka是一个分布式的基于发布/订阅模式的消息对队列，主要应用于大数据实时处理领域。

## 消息队列的好处

1. 解耦
2. 可恢复性
3. 缓冲
4. 灵活性和峰值处理能力
5. 异步通信

## 消息队列的模式

1. 点对点模式（一对一，消费者主动拉取数据，消息收到后消息清除）
2. 发布/订阅模式（一对多，消费者消费数据之后不会清除消息）：生产者将消息发布到topic，同时有多个消费者订阅消费该消息，和点对点不同，发布到topic的消息会被所有的订阅者消费。

## kafka基础架构

一个典型的 Kafka 体系架构包括若干 Producer、若干 Broker、若干 Consumer，以及一个ZooKeeper集群，其中ZooKeeper是Kafka用来负责集群元数据的管理、控制器的选举等操作的。Producer将消息发送到Broker，Broker负责将收到的消息存储到磁盘中，而Consumer负责从Broker订阅并消费消息。

1. Producer：生产者，也就是发送消息的一方。生产者负责创建消息，然后将其投递到Kafka中。
2. Consumer：消费者，也就是接收消息的一方。消费者连接到Kafka上并接收消息，进而进行相应的业务逻辑处理。
3. Broker：服务代理节点。对于Kafka而言，Broker可以简单地看作一个独立的Kafka服务节点或Kafka服务实例。大多数情况下也可以将Broker看作一台Kafka服务器，前提是这台服务器上只部署了一个Kafka实例。一个或多个Broker组成了一个Kafka集群。

### 主题（topic）和分区（partition）

Kafka中的消息以主题为单位进行归类，生产者负责将消息发送到特定的主题（发送到Kafka集群中的每一条消息都要指定一个主题），而消费者负责订阅主题并进行消费。

主题是一个逻辑上的概念，它还可以细分为多个分区，一个分区只属于单个主题，很多时候也会把分区称为主题分区（Topic-Partition）。同一主题下的不同分区包含的消息是不同的，分区在存储层面可以看作一个可追加的日志（Log）文件，消息在被追加到分区日志文件的时候都会分配一个特定的偏移量（offset）。offset是消息在分区中的唯一标识，Kafka通过它来保证消息在分区内的顺序性，不过offset并不跨越分区，也就是说，Kafka保证的是分区有序而不是主题有序。

### 分区（partition）和副本（Replica）

Kafka 为分区引入了多副本（Replica）机制，通过增加副本数量可以提升容灾能力。同一分区的不同副本中保存的是相同的消息（在同一时刻，副本之间并非完全一样），副本之间是“一主多从”的关系，其中leader副本负责处理读写请求，follower副本只负责与leader副本的消息同步。副本处于不同的broker中，当leader副本出现故障时，从follower副本中重新选举新的leader副本对外提供服务。Kafka通过多副本机制实现了故障的自动转移，当Kafka集群中某个broker失效时仍然能保证服务可用。

> Kafka集群中有4个broker，某个主题中有3个分区，且副本因子（即副本个数）也为3，如此每个分区便有1个leader副本和2个follower副本。生产者和消费者只与leader副本进行交互，而follower副本只负责消息的同步，很多时候follower副本中的消息相对leader副本而言会有一定的滞后。

分区中的所有副本统称为AR（Assigned Replicas）。所有与leader副本保持一定程度同步的副本（包括leader副本在内）组成ISR（In-Sync Replicas）。与leader副本同步滞后过多的副本（不包括leader副本）组成OSR（Out-of-Sync Replicas），由此可见，AR=ISR+OSR。在正常情况下，所有的 follower 副本都应该与 leader 副本保持一定程度的同步，即 AR=ISR，OSR集合为空。

leader副本负责维护和跟踪ISR集合中所有follower副本的滞后状态，当follower副本落后太多或失效时，leader副本会把它从ISR集合中剔除。如果OSR集合中有follower副本“追上”了leader副本，那么leader副本会把它从OSR集合转移至ISR集合。默认情况下，当leader副本发生故障时，只有在ISR集合中的副本才有资格被选举为新的leader，而在OSR集合中的副本则没有任何机会（不过这个原则也可以通过修改相应的参数配置来改变）。

ISR与HW和LEO也有紧密的关系。HW是High Watermark的缩写，俗称高水位，它标识了一个特定的消息偏移量（offset），消费者只能拉取到这个offset之前的消息。

> 这个日志文件中有 9 条消息，第一条消息的offset（LogStartOffset）为0，最后一条消息的offset为8，offset为9的消息用虚线框表示，代表下一条待写入的消息。日志文件的HW为6，表示消费者只能拉取到offset在0至5之间的消息，而offset为6的消息对消费者而言是不可见的。

LEO是Log End Offset的缩写，它标识当前日志文件中下一条待写入消息的offset，图1-4中offset为9的位置即为当前日志文件的LEO，LEO的大小相当于当前日志分区中最后一条消息的offset值加1。分区ISR集合中的每个副本都会维护自身的LEO，而ISR集合中最小的LEO即为分区的HW，对消费者而言只能消费HW之前的消息。

### 容灾能力

Kafka 消费端也具备一定的容灾能力。Consumer 使用拉（Pull）模式从服务端拉取消息，并且保存消费的具体位置，当消费者宕机后恢复上线时可以根据之前保存的消费位置重新拉取需要的消息进行消费，这样就不会造成消息丢失。

# 生产者

## 消息发送

1. 构建发送消息的配置信息和生产者实例
2. 构建待发送的消息
3. 发送消息
4. 关闭生产者实例

简单的实例：

~~~~java
Properties properties = new Properties();
        // kafka的broker集群地址
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,"");
        //泛型key的序列化器 注意这里必须填写序列化器的全限定名
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        //泛型同上
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,StringSerializer.class.getName());
        //指定拦截器
        properties.put(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG,"");
        //指定分词器
        properties.put(ProducerConfig.PARTITIONER_CLASS_CONFIG,"");
        KafkaProducer<String,String> kafkaProducer = new KafkaProducer<String, String>(properties);
        String topic = "topicTest";
        String content = "发送的消息";
        ProducerRecord<String,String> producerRecord = new ProducerRecord<String,String>(topic,content);
        //发送消息
        kafkaProducer.send(producerRecord);
        kafkaProducer.close();
~~~~

### 配置参数构建说明

~~~
org.apache.kafka.clients.producer.ProducerConfig
~~~

重要的几个参数

- bootstrap.servers：生产者客户端连接kafka集群的broker地址，格式host1：port1，host2：port2
- key.serializer 和 value.serializer 消息的key和value序列化器，value值一般为序列化器详细的类地址（如：org.apache.kafka.common.serialization.StringSerializer）
- 其他参数。。





# 消费者

消费者和消费者组

消费者对应分区 消费者组对应 topic

创建消费者必要的参数  

订阅主题和分区

反序列化

消息消费 poll（）

位移提交 

控制和关闭消费

指定位移消费

在均衡

消费拦截者

多线程实现

重要的消费者参数

位移提交

# 主题和分区

主题管理方式 .sh脚本和KafkaAdminClient  开始懵逼。。。

自动创建主题和分区 broker端参数auto.create.topics.enable 为true  分区和分区副本都为1

kafka-topics.sh 脚本

1. 创建主题 
2. 分区副本分配


