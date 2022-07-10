---
title: Kafka基础 
categories: 
- GolangStudy
---
# Kafka基础 

* Kafka是一个分布式的基于发布/订阅模式的消息队列，主要用于大数据实时处理

## 消息队列

* 使用消息队列的好处:
	- 解耦: 可以独立地扩展或者修改两边的处理过程
	- 可恢复性: 即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理
	- 缓冲: 解决生产者和消费者处理消息速度不一致的情况(生产 > 消费)
	- 灵活性和削峰处理能力: 不会因为突发的超负荷的请求而完全崩溃，消息队列能够使关键组件顶住突发的访问压力。
	- 异步通信: 消息队列允许用户把消息放入队列但不立即处理它。

* 消息队列的两种模式
	- 点对点模式(一对一[消息 对 消费者]，消费者主动拉取数据，消息收到后消息清除)
		```
		* 消息生产者生产消息发送到queue中，然后消息消费者从queue中取出并且消费消息。

		* 消息被消费以后，queue中不再有存储，所以消息消费者不可能消费到已经被消费的消息。Queue支持存在多个消费者，但是对一个消息而言，只会有一个消费者可以消费
		```

		![查看源图像](https://img-blog.csdnimg.cn/20200802210501168.png)

	- 发布订阅(一对多[消息 对 消费者], 消费者消费数据之后不会清除数据，消费者主动拉取的模式)

		```
		* 消息生产者（发布）将消息发布到topic中，同时有多个消息消费者（订阅）消费该消息。

		* 和点对点方式不同，发布到topic的消息会被所有订阅者消费。
		```

		![查看源图像](https://pic2.zhimg.com/v2-08a36aa5e745285aebe8352d9f7fbebf_r.jpg)

## Kafka基础架构

![查看源图像](https://www.diguage.com/images/kafka/kafka-architecture.png)

* Producer生产者进程产生的数据可以被发布到不同的Topic主题下的不同 Partition 分区
* 在一个分区内，这些消息被索引并连同时间戳存储在一起。
* Consumer 消费者进程可以从分区订阅消息

* Kafka重要概念
	- Producer： 消息生产者，向 Kafka Broker 发消息的客户端。
	- Consumer： 消息消费者，从 Kafka Broker 取消息的客户端。
	- Consumer Group： 消费者组（CG），消费者组内每个消费者负责消费不同分区的数据，提高消费能力。一个分区只能由组内一个消费者消费，消费者组之间互不影响。所有的消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者。
	- Broker： 一台 Kafka 机器就是一个 broker。一个集群由多个 broker 组成。一个 broker 可以容纳多个 topic。
	- Topic： 可以理解为一个队列，topic 将消息分类，生产者和消费者面向的是同一个 topic。
	- Partition： 为了实现扩展性，提高并发能力，一个非常大的 topic 可以分布到多个 broker （即服务器）上，一个 topic 可以分为多个 partition，每个 partition 是一个 有序的队列。
	- Replica： 副本，为实现备份的功能，保证集群中的某个节点发生故障时，该节点上的 partition 数据不丢失，且 Kafka 仍然能够继续工作，Kafka 提供了副本机制，一个 topic 的每个分区都有若干个副本，一个 leader 和若干个 follower。
	- Leader： 每个分区多个副本的“主”副本，生产者发送数据的对象，以及消费者消费数据的对象，都是 leader。
	- Follower： 每个分区多个副本的“从”副本，实时从 leader 中同步数据，保持和 leader 数据的同步。leader 发生故障时，某个 follower 还会成为新的 leader。
	- offset：消费者消费的位置信息，监控数据消费到什么位置，当消费者挂掉再重新恢复的时候，可以从消费位置继续消费。
	- Zookeeper： Kafka 集群能够正常工作，需要依赖于 zookeeper，zookeeper 帮助 Kafka 存储和管理集群信息。

## Kafka安装和基本使用

* 启动Kafka
	- 启动zookeeper
		``` bash
		sudo bin/zookeeper-server-start.sh  config/zookeeper.properties 
		# 后台启动zookeeper
		sudo bin/zookeeper-server-start.sh -daemon  config/zookeeper.properties 
		```
	- 启动kafka
		``` bash
		sudo bin/kafka-server-start.sh config/server.properties 
		# 后台启动kafka
		sudo bin/kafka-server-start.sh -daemon config/server.properties 
		```

* 停止Kafka
	- 停止kafka
	`sudo bin/kafka-server-stop.sh`
	- 停止zookeeper
	`sudo bin/zookeeper-server-stop.sh`


* 测试kafaka

* 创建一个测试的Topic
	``` bash
	bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

		- --replication-factor  副本数
		- --partitions  分区数
		- --topic 指定topic名字
	```

* 查看Topic
	``` bash
	#连接zookkeeper模式
	bin/kafka-topics.sh --list --zookeeper localhost:2181  
	#直接kafka模式
	bin/kafka-topics.sh --list --bootstrap-server localhost:9092
	```

* 删除Topic
	``` bash
	bin/kafka-topics.sh --delete --zookeeper localhost:2181 --topic test 
	```

* 查看Topic描述信息
	``` bash
	bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test 
	```

* 产生消息
	``` bash
	#直接kafka模式
	bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test 
	```

* 消费消息
	``` bash
	bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
	#直接kafka模式
	bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
	```

## Kafka架构深入

`Kafka工作流程`

![查看源图像](https://www.freesion.com/images/679/61ba1660d8ee0128987a3e012674cad7.png)

* Kafka 中消息是以 topic 进行分类的，生产者生产消息，消费者消费消息，面向的都是同一个 topic。

* topic 是逻辑上的概念，而 partition 是物理上的概念，每个 partition 对应于一个 log 文件，该 log 文件中存储的就是 Producer 生产的数据。Producer 生产的数据会不断追加到该 log 文件末端，且每条数据都有自己的 offset。消费者组中的每个消费者，都会实时记录自己消费到了哪个 offset，以便出错恢复时，从上次的位置继续消费。

`Kafka文件存储`

![preview](https://pic2.zhimg.com/v2-23dcdca68db7e5406e3f036297f68c4d_r.jpg)

* 由于生产者生产的消息会不断追加到 log 文件末尾，为防止 log 文件过大导致数据定位效率低下，Kafka 采取了分片和索引机制，将每个 partition 分为多个 segment，每个 segment 对应两个文件：“.index” 索引文件和 “.log” 数据文件

## Kafka生产者

* 分区原因
	- 方便在集群中扩展，每个 partition 可以通过调整以适应它所在的机器，而一个 topic 又可以有多个 partition 组成，因此可以以 partition 为单位读写了。
	- 可以提高并发，以 partition 为单位读写了。

* 分区原则
	```
	我们需要将 Producer 发送的数据封装成一个 ProducerRecord 对象。该对象需要指定一些参数：
	- topic：string 类型，NotNull
	- partition：int 类型，可选
	- timestamp：long 类型，可选
	- key：string类型，可选
	- value：string 类型，可选
	- headers：array 类型，Nullable
	```
	- 指明 partition 的情况下，直接指明的值 作为 partition 的值。
	- 没有指明 partition 但有 key 的情况下，将 key 的 hash 值与分区数取余得到 partition 值。
	- 既没有 partition 有没有 key 的情况下，第一次调用时随机生成一个整数（后面每次调用都在这个整数上自增），将这个值与可用的分区数取余，得到 partition 值，也就是常说的 round-robin 轮询算法。

* 生产者数据可靠性保证
	- 为保证 producer 发送的数据，能可靠地发送到指定的 topic，topic 的每个 partition 收到 producer 发送的数据后，都需要向 producer 发送 ack（acknowledge 确认收到），如果 producer 收到 ack，就会进行下一轮的发送，否则重新发送数据

	![preview](https://pic3.zhimg.com/v2-2225e28f85f77380de5fcededf77249e_r.jpg)

	- 副本数据同步策略
		```
		* 何时发送ack?
			- leader 收到数据之后
			- leader数据并同步到follower之后
		* 多少follower同步完成之后发送ack?
			- 半数同步完成
			- 全部同步完成
		* Kafka选择全部同步完成才发送ack的方案在忍受相同机器故障的情况下所需的及取数更少
		```
		![preview](https://pic4.zhimg.com/v2-05f1d9b679c8db558d56dc9afcc8c4c3_r.jpg)

	- ISR
		```
		* 采用第二种方案，所有 follower 完成同步，producer 才能继续发送数据，设想有一个 follower 因为某种原因出现故障，那 leader 就要一直等到它完成同步。这个问题怎么解决？ 

		* leader维护了一个动态的 in-sync replica set（ISR), 意为和 leader 保持同步的 follower 集合。当 ISR 集合中的 follower 完成数据的同步之后，leader 就会给 producer 发送 ack。如果 follower 长时间未向 leader 同步数据，则该 follower 将被踢出 ISR 集合，该时间阈值由 replica.lag.time.max.ms 参数设定。

		* leader 发生故障后，就会从 ISR 中选举出新的 leader
		```
	
	- acks 参数配置
		```
		* 对于某些不太重要的数据，对数据的可靠性要求不是很高，能够容忍数据的少量丢失，所以没必要等 ISR 中的 follower 全部接受成功。 

		* 所以 Kafka 为用户提供了三种可靠性级别，用户根据可靠性和延迟的要求进行权衡，选择以下的配置
		```

		* 0：producer 不等待 broker 的 ack，这提供了最低延迟，broker 一收到数据还没有写入磁盘就已经返回，当 broker 故障时有可能丢失数据。
		* 1：producer 等待 broker 的 ack，partition 的 leader 落盘成功后返回 ack，如果在 follower 同步成功之前 leader 故障，那么将会丢失数据。
		* -1（all）：producer 等待 broker 的 ack，partition 的 leader 和 follower 全部落盘成功后才返回 ack。但是在 broker 发送 ack 时，leader 发生故障，则会造成数据重复。

		![img](https://pic2.zhimg.com/80/v2-73974b936cd969c98eab1a088e680bd5_720w.jpg)
	
	- 故障处理细节
		![preview](https://pic2.zhimg.com/v2-b417f990a4378583e76dfde8672bf4d1_r.jpg)

		```
		* LEO：每个副本最大的 offset。 
		* HW：消费者能见到的最大的 offset，ISR 队列中最小的 LEO
		```
		* Follower 故障
			```
			follower 发生故障后会被临时踢出 ISR 集合，待该 follower 恢复后，follower 会 读取本地磁盘记录的上次的 HW，并将 log 文件高于 HW 的部分截取掉，从 HW 开始向 leader 进行同步数据操作。等该 follower 的 LEO 大于等于该 partition 的 HW，即 follower 追上 leader 后，就可以重新加入 ISR 了。
			```
		* Leader 故障
			```
			* leader 发生故障后，会从 ISR 中选出一个新的 leader，之后，为保证多个副本之间的数据一致性，其余的 follower 会先将各自的 log 文件高于 HW 的部分截掉，然后从新的 leader 同步数据。 

			* 注意：这只能保证副本之间的数据一致性，并不能保证数据不丢失或者不重复，acks机制会影响数据丢失性
			```

	- Exactly Once 语义
		```
		* 将服务器的 ACK 级别设置为-1，可以保证 producer 到 server 之间不会丢失数据，即 At Least Once 语义。

		* 将服务器 ACK 级别设置为0，可以保证生产者每条消息只会被发送一次，即At Most Once 语义

		* At Least Once 可以保证数据不丢失，但是不能保证数据不重复；相对的，At Most Once 可以保证数据不重复，但是不能保证数据不丢失。

		* 但是，对于一些非常重要的信息，比如交易数据，下游数据消费者要求数据既不重复也不丢失，即 Exactly Once 语义

		* 引入了幂等性：producer 不论向 server 发送多少重复数据，server 端都只会持久化一条。即：At Least Once + 幂等性 = Exactly Once

		* 启用幂等性，只需要将 producer 的参数中 enable.idompotence 设置为 true 即可

		* 开启幂等性的 producer 在初始化时会被分配一个 PID，发往同一 partition 的消息会附带 Sequence Number。而 borker 端会对 <PID,Partition,SeqNumber> 做缓存，当具有相同主键的消息提交时，broker 只会持久化一条。 

		* 但是 PID 重启后就会变化，同时不同的 partition 也具有不同主键，所以幂等性无法保证跨分区会话的 Exactly Once
		```

* 总结
	- ack: 决定数据丢失情况
	- ISR: 同步副本集合
	- HW和LEO

## Kafka消费者

* 消费方式
	- consumer 采用 pull（拉取）模式从 broker 中读取数据

	- broker给consumer推送消息的模式,很难适应消费速率不同的消费者。它的目标是尽可能以最快速度传递消息，但是这样很容易造成 consumer 来不及处理消息，典型的表现就是拒绝服务以及网络拥塞， 而 pull 模式则可以根据 consumer 的消费能力以适当的速率消费消息

	- pull 模式不足之处是，如果 Kafka 没有数据，消费者可能会陷入循环中，一直返回空数据。因为消费者从 broker 主动拉取数据，需要维护一个长轮询，针对这一点， Kafka 的消费者在消费数据时会传入一个时长参数 timeout，如果当前没有数据可供消费，consumer 会等待一段时间之后再返回，这段时长即为 timeout。

* 分区分配策略
	- 一个 consumer group 中有多个 consumer，一个 topic 有多个 partition，所以必然会涉及到 partition 的分配问题，即确定哪个 partition 由哪个 consumer 来消费

	- Kafka 有两种分配策略，一个是 RoundRobin，一个是 Range，默认为range，当消费者组内消费者发生变化时，会触发分区分配策略（方法重新分配

	- 当消费者组中的消费者数量发生变化的时候会触发分区分配策略
	- RoundRobin

		![preview](https://pic2.zhimg.com/v2-71b0b3eba72505d5658e94446bc159b9_r.jpg)

	- Range range 方式是按照主题来分的，不会产生轮询方式的消费混乱问题。

		![img](https://pic4.zhimg.com/80/v2-587357091d733d72a6fb62555ec0a247_720w.jpg)

* offset维护
	- consumer 在消费过程中可能会出现断电宕机等故障，consumer 恢复后，需要从故障前的位置继续消费，所以 consumer 需要实时记录自己消费到了哪个 offset，以便故障恢复后继续消费。 
	- Kafka 0.9 版本之前，consumer 默认将 offset 保存在 Zookeeper 中，从 0.9 版本开始，consumer 默认将 offset 保存在 Kafka 一个内置的 topic 中，该 topic 为 __consumer_offsets，都是按照<组-主题-分区>三个条件确定一个offset

## Kafka高效读写数据

* 顺序写磁盘
	```
	kafka 生产者写数据是有序的，即 Partition 内部有序，数据以 append 的方式顺序追加写入。Consumer 消费数据也是有序的，指定 offset 后顺序读出 offset 之后的数据。顺序读写可以避免磁盘读数据时的多次寻道和旋转延迟
	```

* 零拷贝技术
	![preview](https://pic1.zhimg.com/v2-023030c309ec9406c0a69763f451bcb0_r.jpg)
	```
	- 传统IO流程:
		1、第一次：将磁盘文件，读取到操作系统内核缓冲区；
		2、第二次：将内核缓冲区的数据，copy到application应用程序的buffer；
		3、第三步：将application应用程序buffer中的数据，copy到socket网络发送缓冲区(属于操作系统内核的缓冲区)；
		4、第四次：将socket buffer的数据，copy到网卡，由网卡进行网络传输。

		* 传统方式，读取磁盘文件并进行网络发送，经过的四次数据copy是非常繁琐的。实际IO读写，需要进行IO中断，需要CPU响应中断(带来上下文切换)，尽管后来引入DMA来接管CPU的中断请求，但四次copy是存在 不必要的拷⻉的。
	```
	![preview](https://pic3.zhimg.com/v2-798463e8afa943f27eec0c980f910f86_r.jpg)
	```
	零拷贝技术: 
		1.将文件拷⻉到kernel buffer中；
		2.向socket buffer中追加当前要发生的数据在kernel buffer中的位置和偏移量；
		3.根据socket buffer中的位置和偏移量直接将kernel buffer的数据copy到网卡设备中；

		*  经过上述过程，数据只经过了2次copy就从磁盘传送出去了。这个才是真正的Zero-Copy(这里的零拷⻉是针对kernel来讲的，数据在kernel模式下是Zero-Copy)。
	```

## Kafka事务

* Kafka事务特性是指一系列的生产者生产消息和消费者提交偏移量的操作在一个事务中，或者说是一个原子操作，生产消息和提交偏移量同时成功或者失败。

* Kafka事务使用
	- 生产者发送多条消息可以封装在一个事务中，形成一个原子操作。多条消息要么都发送成功，要么都发送失败。
	- read-process-write模式：将消息消费和生产封装在一个事务中，形成一个原子操作。在一个流式处理的应用中，常常一个服务需要从上游接收消息，然后经过处理后送达到下游，这就对应着消息的消费和生成。

```
* 没有事务机制的幂等性只能保证单个 Producer 对于同一个<Topic, Partition>的Exactly Once语义

* 并不能保证以下几个操作的Exactly Once操作
	- 写操作的原子性——即多个写操作，要么全部被 Commit 要么全部不被 Commit
	- 多个读写操作的原子性——对于 Kafka Stream 应用而言，典型的操作即是从某个 Topic 消费数据，经过一系列转换后写回另一个 Topic，保证从源 Topic 的读取与向目标 Topic 的写入的原子性有助于从故障中恢复。

* 为了实现事务，应用程序必须提供一个稳定的（重启后不变）唯一的 ID，也即Transaction ID。Transactin ID与PID绑定。区别在于Transaction ID由用户提供，而PID是内部的实现对用户透明。

* Kafka为了支持事务特性，引入一个新的组件：Transaction Coordinator。主要负责分配pid，记录事务状态等操作
```

## 生产者发送消息有哪些模式

* 发后即忘(fire-and-forget): 它只管往 Kafka 里面发送消息，但是不关心消息是否正确到达，这种方式的效率最高，但是可靠性也最差，比如当发生某些不可充实异常的时候会造成消息的丢失
* 同步(ync): producer.send()返回一个Future对象，调用get()方法变回进行同步等待，就知道消息是否发送成功，发送一条消息需要等上个消息发送成功后才可以继续发送
* 异步(async): Kafka支持 producer.send() 传入一个回调函数，消息不管成功或者失败都会调用这个回调函数，这样就算是异步发送，我们也知道消息的发送情况，然后再回调函数中选择记录日志还是重试都取决于调用方 

## Kafka实现负载均衡的方式:

* Kafka 的负责均衡主要是通过分区来实现的，Kafka 是主写主读的架构，如下图:
<img src="https://pic4.zhimg.com/v2-1d7719726f1cdcc2bd608a8e9df2c71b_r.jpg" alt="preview" style="zoom:50%;" />
* 每个 broker 都有消费者拉取消息，每个 broker 也都有生产者发送消息，每个 broker 上的读写负载都是一样的，这也说明了 kafka 独特的架构方式可以通过主写主读来实现负载均衡。

## Kafka 的负责均衡存在的问题

* broker 端分配不均: 当创建 topic 的时候可能会出现某些 broker 分配到的分区数多，而有些 broker 分配的分区少，这就导致了 leader 多副本不均
* 生产者写入消息不均: 生产者可能只对某些 broker 中的 leader 副本进行大量的写入操作，而对其他的 leader 副本不闻不问。
* 消费者消费不均: 消费者可能只对某些 broker 中的 leader 副本进行大量的拉取操作，而对其他的 leader 副本不闻不问。
* leader 副本切换不均：当主从副本切换或者分区副本进行了重分配后，可能会导致各个 broker 中的 leader 副本分配不均匀。

## 分区再分配

* 分区再分配主要是用来维护 kafka 集群的负载均衡
* 场景一: 如果一个节点的分区是单副本的,那么结点下线会导致分区将会变得不可用
* 场景二: 当集群新增 broker 时，只有新的主题分区会分配在该 broker 上，而老的主题分区不会分配在该 broker 上，就造成了老节点和新节点之间的负载不均衡。
* 分区再分配，它可以在集群扩容，broker 失效的场景下进行分区迁移
* 分区再分配的原理就是通过控制器给分区新增新的副本，然后通过网络把旧的副本数据复制到新的副本上，在复制完成后，将旧副本清除。 当然，为了不影响集群正常的性能，在此复制期间还会有一些列保证性能的操作，比如复制限流

## 增强消费者消费能力的方法

* 增加 topic 的分区数，并且同时提升消费组的消费者数量，消费者数=分区数。
* 可以采用多线程的方式进行消费，并且优化业务方法流程

## Kafka控制器作用

* Kafka 集群中有一台broker 会被选举为控制器，它负责管理整个集群中所有分区和副本的状态，kafka 集群中只能有一个控制器
	- 当某个分区的 leader 副本出现故障时，由控制器负责为该分区选举新的 leader 副本
	- 检测到某个分区的ISR集合发生变化时，由控制器负责通知所有 broker 更新其元数据信息
	- 当为某个 topic 增加分区数量时，由控制器负责分区的重新分配

## Kafka速度快的原因

* 顺序读写
* Page Cache: 为了优化读写性能，Kafka 利用了操作系统本身的 Page Cache
* 零拷贝: Kafka使用了零拷贝技术，也就是直接将数据从内核空间的读缓冲区直接拷贝到内核空间的 socket 缓冲区，然后再写入到 NIC 缓冲区，避免了在内核空间和用户空间之间穿梭
* 分区分段+索引
* 批量读写
* 批量压缩
