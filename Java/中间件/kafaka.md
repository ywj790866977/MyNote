# kafaka

## 定义

Kafka 是一个分布式的基于发布/订阅模式的消息队列(Message Queue)，主要应用于 大数据实时处理领域。

###  消息队列

![image-20200227220203453](/Users/yan/Google 云端硬盘/笔记/img/image-20200227220203453.png)

### 使用消息队列的好处

1)解耦 允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。

2)可恢复性 系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所

以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。 

3)缓冲

有助于控制和优化数据流经过系统的速度，解决生产消息和消费消息的处理速度不一致 的情况。

4)灵活性 & 峰值处理能力 在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见。

如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列 能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。

 5)异步通信

很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户 把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要 的时候再去处理它们。

### 消息队列的两种模式

(1)**点对点模式**(一对一，消费者主动拉取数据，消息收到后消息清除) 

​	消息生产者生产消息发送到 Queue 中，然后消息消费者从 Queue 中取出并且消费消息。

消息被消费以后，queue 中不再有存储，所以消息消费者不可能消费到已经被消费的消息。 Queue 支持存在多个消费者，但是对一个消息而言，只会有一个消费者可以消费。

![image-20200227220820567](/Users/yan/Google 云端硬盘/笔记/img/image-20200227220820567.png)

(2)**发布/订阅模式**(一对多，消费者消费数据之后不会清除消息) 消息生产者(发布)将消息发布到 topic 中，同时有多个消息消费者(订阅)消费该消

息。和点对点方式不同，发布到 topic 的消息会被所有订阅者消费。

![image-20200227220908956](/Users/yan/Google 云端硬盘/笔记/img/image-20200227220908956.png)

### Kafka 基础架构

![image-20200227221018105](/Users/yan/Google 云端硬盘/笔记/img/image-20200227221018105.png)

 **1)Producer** : 消息生产者，就是向 kafka broker 发消息的客户端;
 **2)Consumer** : 消息消费者，向 kafka broker 取消息的客户端;
 **3)Consumer Group (CG)**:消费者组，由多个 consumer 组成。消费者组内每个消费者负 责消费不同分区的数据，一个分区只能由一个组内消费者消费;消费者组之间互不影响。所 有的消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者。
 **4)Broker** :一台 kafka 服务器就是一个 broker。一个集群由多个 broker 组成。一个 broker 可以容纳多个 topic。
 **5)Topic** :可以理解为一个队列，生产者和消费者面向的都是一个 topic; 

 **6)Partition**:为了实现扩展性，一个非常大的 topic 可以分布到多个 broker(即服务器)上， 一个 topic 可以分为多个 partition，每个 partition 是一个有序的队列;

 **7)Replica**:副本，为保证集群中的某个节点发生故障时，该节点上的 partition 数据不丢失，且 kafka 仍然能够继续工作，kafka 提供了副本机制，一个 topic 的每个分区都有若干个副本， 一个 leader 和若干个 follower。

**8)leader**:每个分区多个副本的“主”，生产者发送数据的对象，以及消费者消费数据的对 象都是 leader。

**9)follower**:每个分区多个副本中的“从”，实时从 leader 中同步数据，保持和 leader 数据 的同步。leader 发生故障时，某个 follower 会成为新的 follower。



## 安装部署

1. 解压安装包

```shell
tar -zxvf kafka_2.11-0.11.0.0.tgz -C /opt/module/
```

2. 修改解压后的文件名称

```shell
mv kafka_2.11-0.11.0.0/ kafka
```

3. 在/opt/module/kafka 目录下创建 logs 文件夹

```shell
mkdir logs
```

4. 修改配置文件

```shell
cd config/
vi server.properties

####
#broker 的全局唯一编号，不能重复 broker.id=0
#删除 topic 功能使能
delete.topic.enable=true
#处理网络请求的线程数量 num.network.threads=3
#用来处理磁盘 IO 的现成数量 num.io.threads=8 #发送套接字的缓冲区大小 socket.send.buffer.bytes=102400 #接收套接字的缓冲区大小 socket.receive.buffer.bytes=102400 #请求套接字的缓冲区大小 socket.request.max.bytes=104857600 #kafka 运行日志存放的路径 log.dirs=/opt/module/kafka/logs #topic 在当前 broker 上的分区个数 num.partitions=1
#用来恢复和清理 data 下数据的线程数量 num.recovery.threads.per.data.dir=1 #segment 文件保留的最长时间，超时将被删除 log.retention.hours=168
#配置连接 Zookeeper 集群地址 zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181
```

5. 配置环境变量

```shell
sudo vi /etc/profile
#KAFKA_HOME
export KAFKA_HOME=/opt/module/kafka export PATH=$PATH:$KAFKA_HOME/bin

source /etc/profile
```

6. 分发安装包

```
xsync kafka/
注意:分发之后记得配置其他机器的环境变量
```

7. 分别在 hadoop103 和 hadoop104 上修改配置文件/opt/module/kafka/config/server.properties中的 broker.id=1、broker.id=2

注:broker.id 不得重复

8. 启动集群

   依次在 hadoop102、hadoop103、hadoop104 节点上启动 kafka

```shell
[atguigu@hadoop102 kafka]$ config/server.properties
[atguigu@hadoop103 kafka]$ config/server.properties
bin/kafka-server-start.sh
bin/kafka-server-start.sh
-daemon -daemon -daemon
[atguigu@hadoop104 kafka]$ bin/kafka-server-start.sh config/server.properties
```

9. 关闭集群

```shell
 [atguigu@hadoop102 kafka]$ bin/kafka-server-stop.sh stop [atguigu@hadoop103 kafka]$ bin/kafka-server-stop.sh stop [atguigu@hadoop104 kafka]$ bin/kafka-server-stop.sh stop
```

10. kafka 群起脚本

```shell
for i in hadoop102 hadoop103 hadoop104
do
	echo "========== $i =========="
	ssh $i '/opt/module/kafka/bin/kafka-server-start.sh -daemon
	/opt/module/kafka/config/server.properties'
done
```



## Kafka 命令行操作

1)查看当前服务器中的所有 topic

```shell
bin/kafka-topics.sh --zookeeper hadoop102:2181 --list
```

2)创建 topic

```shell
bin/kafka-topics.sh --zookeeper hadoop102:2181 --create --replication-factor 3 --partitions 1 -- topic first

选项说明:
--topic 定义 topic 名
--replication-factor 定义副本数
--partitions 定义分区数
```

3)删除 topic

```shell
bin/kafka-topics.sh --zookeeper hadoop102:2181 --delete --topic first
## 需要 server.properties 中设置 delete.topic.enable=true 否则只是标记删除。
```

4)发送消息

```shell
 bin/kafka-console-producer.sh --broker- list hadoop102:9092 --topic first
>hello world
>atguigu atguigu
```



5)消费消息

```shell
bin/kafka-console-consumer.sh \ --zookeeper hadoop102:2181 --topic first
bin/kafka-console-consumer.sh \ --bootstrap-server hadoop102:9092 --topic first
bin/kafka-console-consumer.sh \ --bootstrap-server hadoop102:9092 --from-beginning --topic first
## --from-beginning:会把主题中以往所有的数据都读取出来。
```

6)查看某个 Topic 的详情

```shell
bin/kafka-topics.sh --zookeeper hadoop102:2181 --describe --topic first
```

7)修改分区数

```shell
bin/kafka-topics.sh --zookeeper
hadoop102:2181 --alter --topic first --partitions 6
```



## Kafka 架构深入

![image-20200227223142938](/Users/yan/Google 云端硬盘/笔记/img/image-20200227223142938.png)



Kafka 中消息是以 topic 进行分类的，生产者生产消息，消费者消费消息，都是面向 topic 的。

topic 是逻辑上的概念，而 partition 是物理上的概念，每个 partition 对应于一个 log 文 件，该 log 文件中存储的就是 producer 生产的数据。Producer 生产的数据会被不断追加到该 log 文件末端，且每条数据都有自己的 offset。消费者组中的每个消费者，都会实时记录自己 消费到了哪个 offset，以便出错恢复时，从上次的位置继续消费。



![image-20200227223223877](/Users/yan/Google 云端硬盘/笔记/img/image-20200227223223877.png)

由于生产者生产的消息会不断追加到 log 文件末尾，为防止 log 文件过大导致数据定位 效率低下，Kafka 采取了分片和索引机制，将每个 partition 分为多个 segment。每个 segment 对应两个文件——“.index”文件和“.log”文件。这些文件位于一个文件夹下，该文件夹的命名 规则为:topic 名称+分区序号。例如，first 这个 topic 有三个分区，则其对应的文件夹为 first- 0,first-1,first-2。

```
00000000000000000000.index
00000000000000000000.log 
00000000000000170410.index 
00000000000000170410.log 
00000000000000239430.index 
00000000000000239430.log
```

index 和 log 文件以当前 segment 的第一条消息的 offset 命名。下图为 index 文件和 log 文件的结构示意图。

![image-20200227223338216](/Users/yan/Google 云端硬盘/笔记/img/image-20200227223338216.png)

“.index”文件存储大量的索引信息，“.log”文件存储大量的数据，索引文件中的元 数据指向对应数据文件中 message 的物理偏移地址。

## Kafka 生产者

### 分区策略

1)分区的原因
	 (1)方便在集群中扩展，每个 Partition 可以通过调整以适应它所在的机器，而一个 topic

又可以有多个 Partition 组成，因此整个集群就可以适应任意大小的数据了;

​	 (2)可以提高并发，因为可以以 Partition 为单位读写了。

2)分区的原则
 我们需要将 producer 发送的数据封装成一个 ProducerRecord 对象。

![image-20200227223913545](/Users/yan/Google 云端硬盘/笔记/img/image-20200227223913545.png)

​	(1)指明 partition 的情况下，直接将指明的值直接作为 partiton 值;

​	(2)没有指明 partition 值但有 key 的情况下，将 key 的 hash 值与 topic 的 partition 数进行取余得到 partition 值;

​	(3)既没有 partition 值又没有 key 值的情况下，第一次调用时随机生成一个整数(后 面每次调用在这个整数上自增)，将这个值与 topic 可用的 partition 总数取余得到 partition 值，也就是常说的 round-robin 算法。

### 数据可靠性保证

为保证 producer 发送的数据，能可靠的发送到指定的 topic，topic 的每个 partition 收到 producer 发送的数据后，都需要向 producer 发送 ack(acknowledgement 确认收到)，如果 producer 收到 ack，就会进行下一轮的发送，否则重新发送数据。

![image-20200227224030009](/Users/yan/Google 云端硬盘/笔记/img/image-20200227224030009.png)

1)副本数据同步策略

| 方案                          | 优点                                                       | 缺点                                                        |
| ----------------------------- | ---------------------------------------------------------- | ----------------------------------------------------------- |
| 半数以上完成同步，就发 送 ack | 延迟低                                                     | 选举新的 leader 时，容忍 n 台 节点的故障，需要 2n+1 个副 本 |
| 全部完成同步，才发送          | 选举新的 leader 时，容忍 n 台 节点的故障，需要 n+1 个副 本 | 延迟高                                                      |

Kafka 选择了第二种方案，原因如下:
	 1.同样为了容忍 n 台节点的故障，第一种方案需要 2n+1 个副本，而第二种方案只需要 n+1 个副本，而 Kafka 的每个分区都有大量的数据，第一种方案会造成大量数据的冗余。 

​	2.虽然第二种方案的网络延迟会比较高，但网络延迟对 Kafka 的影响较小。
 2)ISR

​	采用第二种方案之后，设想以下情景:leader 收到数据，所有 follower 都开始同步数据， 但有一个 follower，因为某种故障，迟迟不能与 leader 进行同步，那 leader 就要一直等下去， 直到它完成同步，才能发送 ack。这个问题怎么解决呢?

​	Leader 维护了一个动态的 in-sync replica set (ISR)，意为和 leader 保持同步的 follower 集 合。当 ISR 中的 follower 完成数据的同步之后，leader 就会给 follower 发送 ack。如果 follower 长时间未向 leader 同步数据，则该 follower 将被踢出 ISR，该时间阈值由replica.lag.time.max.ms 参数设定。Leader 发生故障之后，就会从 ISR 中选举新的 leader。

3)ack 应答机制

​	对于某些不太重要的数据，对数据的可靠性要求不是很高，能够容忍数据的少量丢失， 所以没必要等 ISR 中的 follower 全部接收成功。

所以 Kafka 为用户提供了三种可靠性级别，用户根据对可靠性和延迟的要求进行权衡， 选择以下的配置。

acks 参数配置:

​	 acks:

​	0:producer 不等待 broker 的 ack，这一操作提供了一个最低的延迟，broker 一接收到还 没有写入磁盘就已经返回，当 broker 故障时有可能丢失数据;

​	1:producer 等待 broker 的 ack，partition 的 leader 落盘成功后返回 ack，如果在 follower 同步成功之前 leader 故障，那么将会丢失数据;

![image-20200227224628389](/Users/yan/Google 云端硬盘/笔记/image-20200227224628389.png)

-1(all):producer 等待 broker 的 ack，partition 的 leader 和 follower 全部落盘成功后才 返回 ack。但是如果在 follower 同步完成后，broker 发送 ack 之前，leader 发生故障，那么会 造成数据重复。