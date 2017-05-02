# Producer-Scala

> Kafka的Producer Scala客户端API实现

Scala OldProducer

scala版本的生产者发送消息示例: 面向KeyedMessage,指定了topic和消息内容.

```code
    Properties props = new Properties();
    props.put("metadata.broker.list", "192.168.4.31:9092");
    props.put("serializer.class", "kafka.serializer.StringEncoder");
    Producer<Integer,String> producer = new Producer<Integer, String>(new ProducerConfig(props));
    KeyedMessage<Integer, String> data = new KeyedMessage<Integer, String>("myTopic", "message");
    producer.send(data);
    producer.close();
```

> 问:core下也有Producer和Consumer.scala, 和clients中的KafkaProducer,KafkaConsumer有什么区别?
答:从ConsoleProducer看到两种不同的实现:OldProducer->scala的Producer,NewShinyProducer->KafkaProducer

```scala
    val producer = if(config.useOldProducer) {
        new OldProducer(getOldProducerProps(config))
    } else {
        new NewShinyProducer(getNewProducerProps(config))
    }
```

旧的Producer消息用 KeyedMessage,新的用ProducerRecord.不同的Producer实现,用trait定义共同的 send接口.
因为消息最终是以字节的形式存储在日志文件中的,所以字节数组的key和 value可以作为两种不同实现的共同存储结构.

```scala
    trait BaseProducer {
    def send(topic: String, key: Array[Byte], value: Array[Byte])
    def close()
    }
```

![](./20160122151636197)

用scala实现的 Producer构造方式是一样的,需要指定分区方式,Key,Value的序列化.如果是异步还有一个发送线程.
在java版本中用了 RecordAccumulator,RecordBatch,MemoryRecords等来缓存一批消息,scala版本用简单的队列来缓存.

```scala
class Producer[K,V](val config: ProducerConfig, private val eventHandler: EventHandler[K,V]) extends Logging {
  private val queue = new LinkedBlockingQueue[KeyedMessage[K,V]](config.queueBufferingMaxMessages)
  private var producerSendThread: ProducerSendThread[K,V] = null
  config.producerType match {
    case "sync" =>
    case "async" =>
      sync = false
      producerSendThread = new ProducerSendThread[K,V]("ProducerSendThread-"+config.clientId, queue, eventHandler,config.queueBufferingMaxMs, config.batchNumMessages, config.clientId)
      producerSendThread.start()
  }

  def this(config: ProducerConfig) = this(config, new DefaultEventHandler[K,V](config,
                                      CoreUtils.createObject[Partitioner](config.partitionerClass, config.props),
                                      CoreUtils.createObject[Encoder[V]](config.serializerClass, config.props),
                                      CoreUtils.createObject[Encoder[K]](config.keySerializerClass, config.props),
                                      new ProducerPool(config)))
}
```

注意ProducerSendThread线程处理消息也是通过事件处理器 eventHandler的,当然少不了阻塞队列queue.
同步发送消息直接调用事件处理器, 异步发送消息则会加入到阻塞队列BlockingQueue,
通过后台ProducerSendThread线程完成异步发送,类似于java版本的 Sender线程

![](./20160122152749505)

```scala
def send(messages: KeyedMessage[K,V]*) {
    lock synchronized {
      sync match {
        case true => eventHandler.handle(messages)
        case false => asyncSend(messages)
      }
    }
  }
```

上面的send方法其实就是最开始的 Producer示例了,参数用*表示可以发送多条KeyedMessage消息.
异步发送就是将客户端代码中传入的消息messages转存到 Producer的队列中.
然后ProducerSendThread在 Producer中被启动了,就可以从队列中消费消息,完成消息的发送动作.

```scala
  private def asyncSend(messages: Seq[KeyedMessage[K,V]]) {
    for (message <- messages) {
      config.queueEnqueueTimeoutMs match {
        case 0  => queue.offer(message)
        case _  => config.queueEnqueueTimeoutMs < 0 match {
            case true => queue.put(message); true
            case _ => queue.offer(message, config.queueEnqueueTimeoutMs, TimeUnit.MILLISECONDS)
        }
      }
    }
  }
```

在async方式中，将产生的数据放入queue时有三种不同的放入方式:
当queue.enqueue.timeout.ms=0，则立即放入queue中并返回 true，若queue已满，则立即返回false
当queue.enqueue.timeout.ms<0，则立即放入queue，若queue已满，则一直等待queue释放空间
当queue.enqueue.timeout.ms>0，则立即放入queue中并返回 true，若queue已满，则等待queue.enqueue.timeout.ms指定的时间以让 queue释放空间，若时间到queue还是没有足够空间，则立即返回false

## ProducerSendThread

批处理的方式是在每次从queue中 poll一条消息,先加入到一个数组events中,并在每次加入之后判断是否超过batchSize.
如果超过batchSize,则进行一次批处理,同时重置events数组和设置最后一次发送的时间.最后还需要有一次 handle处理.
批处理的操作方式和java版本的 RecordAccumulator类似,每次添加新消息时,都要判断加了之后,是否可以进行批处理.

```scala
  private def processEvents() {
    var lastSend = SystemTime.milliseconds
    var events = new ArrayBuffer[KeyedMessage[K,V]]
    var full: Boolean = false

    // drain the queue until you get a shutdown command
    Iterator.continually(queue.poll(scala.math.max(0, (lastSend + queueTime) - SystemTime.milliseconds), TimeUnit.MILLISECONDS))
                      .takeWhile(item => if(item != null) item ne shutdownCommand else true).foreach {
      currentQueueItem =>
        val expired = currentQueueItem == null
        if(currentQueueItem != null) {
          events += currentQueueItem    // 加入到临时数组中
        } 
        full = events.size >= batchSize // check if the batch size is reached
        if(full || expired) {           // 除了batch满了,还可能是没有消息了     
          tryToHandle(events)           // 开始批处理
          events = new ArrayBuffer[KeyedMessage[K,V]]
        }
    }
    tryToHandle(events)                 // send the last batch of events
  }
```

因为创建ProducerSendThread也指定了默认的 eventHandler,所以在得到每一批消息时,可以交给handler处理了.
而对于同步的发送方式,是直接在handler上处理全部数据.而异步是将全部消息先放到队列中,再一小批一小批地处理.

```scala
  def tryToHandle(events: Seq[KeyedMessage[K,V]]) {
    val size = events.size
    if(size > 0) handler.handle(events)
  }
```

目前为止,scala的代码看起来非常简洁.主要是使用了比较简单的阻塞队列来缓存消息,而不像java中自己实现了很多类.

## BrokerPartitionInfo

BrokerPartitionInfo -> topicPartitionInfo -> TopicMetadata -> PartitionMetadata

BrokerPartitionInfo的 getBrokerPartitionInfo会根据 topic名称获取对应的 BrokerPartition列表.
对于客户端而言只关心这个Partition的 Leader副本,所以返回的是PartitionAndLeader列表.

```scala
class BrokerPartitionInfo(producerConfig: ProducerConfig, producerPool: ProducerPool, topicPartitionInfo: HashMap[String, TopicMetadata]) {
  def getBrokerPartitionInfo(topic: String, correlationId: Int): Seq[PartitionAndLeader] = {
    val topicMetadata = topicPartitionInfo.get(topic)   // check if the cache has metadata for this topic
    val metadata: TopicMetadata = topicMetadata match {
        case Some(m) => m
        case None =>        // refresh the topic metadata cache
          updateInfo(Set(topic), correlationId)
          val topicMetadata = topicPartitionInfo.get(topic)
          topicMetadata match {
            case Some(m) => m
            case None => throw new KafkaException("Failed to fetch topic metadata for topic: " + topic)
          }
      }
    // 一个TopicMetadata会有多个PartitionMetadata,每个PartitionMetadata是Partition的元数据
    val partitionMetadata = metadata.partitionsMetadata
    partitionMetadata.map { m =>
      m.leader match {
        case Some(leader) =>    new PartitionAndLeader(topic, m.partitionId, Some(leader.id))
        case None =>            new PartitionAndLeader(topic, m.partitionId, None)
      }
    }.sortWith((s, t) => s.partitionId < t.partitionId)  // 按照partitionId排序
  }
}
```

在java版本中 Cluster保存了 broker-topic-partitions等的关系,PartitionInfo表示一个分区(有Leader,ISR等)
每条消息都要根据Partitioner为它选择一个 PartitionInfo,然后得到这个Partition的 Leader Broker.根据Leader分组.
PartitionAndLeader类似 PartitionInfo,有一个可选的LeaderBrokerId,但是没有isr,replicas等信息.

`case class PartitionAndLeader(topic: String, partitionId: Int, leaderBrokerIdOpt: Option[Int])`
updateInfo会更新每个 Topic的元数据 TopicMetadata. 更新动作也相当于向Kafka发送一种 Producer请求(fetch request).
客户端发送TopicMetadata的 fetch请求后,会收到topicMetadata的 Response响应,最后放到topicPartitionInfo map中.

```scala
  def updateInfo(topics: Set[String], correlationId: Int) {
    var topicsMetadata: Seq[TopicMetadata] = Nil
    val topicMetadataResponse = ClientUtils.fetchTopicMetadata(topics, brokers, producerConfig, correlationId)
    topicsMetadata = topicMetadataResponse.topicsMetadata
    topicsMetadata.foreach(tmd => if(tmd.errorCode == Errors.NONE.code)  topicPartitionInfo.put(tmd.topic, tmd))
    producerPool.updateProducer(topicsMetadata)
  }
```

这里更新之后的TopicMetadata还会返回给 ProducerPool. 而Producer会从 ProducerPool中获取可用的生产者实例服务于生产请求.
TopicMetadata是一个 Topic的元数据.一个topic有多个 Partitions,所以一个TopicMetadata对应多个 PartitionMetadata.

```scala
object TopicMetadata {
  val NoLeaderNodeId = -1
  def readFrom(buffer: ByteBuffer, brokers: Map[Int, BrokerEndPoint]): TopicMetadata = {
    val errorCode = readShortInRange(buffer, "error code", (-1, Short.MaxValue))
    val topic = readShortString(buffer)
    val numPartitions = readIntInRange(buffer, "number of partitions", (0, Int.MaxValue))
    val partitionsMetadata: Array[PartitionMetadata] = new Array[PartitionMetadata](numPartitions)
    for(i <- 0 until numPartitions) {
      val partitionMetadata = PartitionMetadata.readFrom(buffer, brokers)
      partitionsMetadata(i) = partitionMetadata
    }
    new TopicMetadata(topic, partitionsMetadata, errorCode)
  }
}
case class TopicMetadata(topic: String, partitionsMetadata: Seq[PartitionMetadata], errorCode: Short = Errors.NONE.code)
PartitionMetadata就等于PartitionAndLeader的元数据,包含isr,replicas等(类似于java版本的PartitionInfo).
object PartitionMetadata {
  def readFrom(buffer: ByteBuffer, brokers: Map[Int, BrokerEndPoint]): PartitionMetadata = {
    val errorCode = readShortInRange(buffer, "error code", (-1, Short.MaxValue))
    val partitionId = readIntInRange(buffer, "partition id", (0, Int.MaxValue)) /* partition id */
    val leaderId = buffer.getInt
    val leader = brokers.get(leaderId)   // brokers是集群所有节点,每个Partition都有一个leaderId,

    /* list of all replicas */
    val numReplicas = readIntInRange(buffer, "number of all replicas", (0, Int.MaxValue))
    val replicaIds = (0 until numReplicas).map(_ => buffer.getInt)
    val replicas = replicaIds.map(brokers)

    /* list of in-sync replicas */
    val numIsr = readIntInRange(buffer, "number of in-sync replicas", (0, Int.MaxValue))
    val isrIds = (0 until numIsr).map(_ => buffer.getInt)
    val isr = isrIds.map(brokers)

    new PartitionMetadata(partitionId, leader, replicas, isr, errorCode)
  }
}
case class PartitionMetadata(partitionId: Int, leader: Option[BrokerEndPoint],
                             replicas: Seq[BrokerEndPoint], isr: Seq[BrokerEndPoint] = Seq.empty, errorCode: Short = Errors.NONE.code)
```

## Partition

1. 为消息选择Partition

上一步获取每条消息所属的topic对应的 PartitionAndLeader列表. PartitionAndLeader是这个 topic的所有 Partition.
但这些Partition并不一定都有 Leader,所以PartitionAndLeader的 leaderBrokerIdOpt是可选的.即还不确定有没有Leader.
为消息选择Partition一定是要选择有 Leader的 Partition.如果消息没有key则使用缓存,相同topic的消息都分配同一个 partitionId.

![](./20160122150207223)

```scala
  private def getPartitionListForTopic(m: KeyedMessage[K,Message]): Seq[PartitionAndLeader] = {
    val topicPartitionsList = brokerPartitionInfo.getBrokerPartitionInfo(m.topic, correlationId.getAndIncrement)
    topicPartitionsList
  }

  private def getPartition(topic: String, key: Any, topicPartitionList: Seq[PartitionAndLeader]): Int = {
    val numPartitions = topicPartitionList.size
    // 如果消息中没有key, 则从sendPartitionPerTopicCache中为这个topic的消息指定partitionId.  
    val partition = if(key == null) {
        val id = sendPartitionPerTopicCache.get(topic)
        id match {
          case Some(partitionId) => partitionId            
          case None =>
            val availablePartitions = topicPartitionList.filter(_.leaderBrokerIdOpt.isDefined)  // 存在Leader的partitions, 类似于Cluster中的availablePartitionsByTopic
            val index = Utils.abs(Random.nextInt) % availablePartitions.size    // 随机选择一个partition
            val partitionId = availablePartitions(index).partitionId            // 对应的Partition的partitionId
            sendPartitionPerTopicCache.put(topic, partitionId)                  // 放入cache中
            partitionId                                                         // 对于没有key的消息,所有topic都放到了同一个partition里??
        }
      } else partitioner.partition(key, numPartitions)  // 使用Partitioner对key指定partitionId
      partition
  }
```

2. 根据Broker-Partition重新组织消息集

由于生产者的消息集messages可能没有区分 topic. 对每条消息选择所属的Partition,要重新按照Broker组织数据. 通过将乱序的消息按照BrokerId进行分组,这样可以将属于某个Broker的消息一次性发送过去.Int就是BrokerId/NodeId.
对于某个Broker的消息,也要分成不同的TopicPartition,最终每个Partition都会分到一批消息.

```nothing
[Map[Int,                               ⬅️ BrokerId
     Map[TopicAndPartition,             ⬅️ Partition
         Seq[KeyedMessage[K,Message]]]  ⬅️ 属于这个Partition的消息集
```

消息最终会追加到Partition的消息集里. 有多层Map,根据key获取集合,如果不存在,则新建并放入Map;如果存在则直接使用.

```scala
  def partitionAndCollate(messages: Seq[KeyedMessage[K,Message]]): Option[Map[Int, collection.mutable.Map[TopicAndPartition, Seq[KeyedMessage[K,Message]]]]] = {
      val ret = new HashMap[Int, collection.mutable.Map[TopicAndPartition, Seq[KeyedMessage[K,Message]]]]
      for (message <- messages) {
        //一个topic有多个partition
        val topicPartitionsList = getPartitionListForTopic(message)
        //一条消息只会写到一个partition, 根据Partitioner会分到一个partition编号
        val partitionIndex = getPartition(message.topic, message.partitionKey, topicPartitionsList)
        //一个partition因为有副本,所以有多个broker,但是写的时候只写到Leader
        val brokerPartition = topicPartitionsList(partitionIndex)
        // postpone the failure until the send operation, so that requests for other brokers are handled correctly
        val leaderBrokerId = brokerPartition.leaderBrokerIdOpt.getOrElse(-1)

        // 每个Broker的data. ret里有嵌套的Map[Int, Map[Partition,Seq]].     
        var dataPerBroker: HashMap[TopicAndPartition, Seq[KeyedMessage[K,Message]]] = null
        ret.get(leaderBrokerId) match {
          case Some(element) =>     // Broker存在里层的Map,直接使用这个Map
            dataPerBroker = element.asInstanceOf[HashMap[TopicAndPartition, Seq[KeyedMessage[K,Message]]]]
          case None =>              // Broker不存在里层Map, 创建一个新的Map, 并放入ret里
            dataPerBroker = new HashMap[TopicAndPartition, Seq[KeyedMessage[K,Message]]]
            ret.put(leaderBrokerId, dataPerBroker)
        }
        // 构造Topic和Partition对象    
        val topicAndPartition = TopicAndPartition(message.topic, brokerPartition.partitionId)
        // Broker对应的消息集合, 即使是相同的Broker, Topic-Partition组合也不一定一样 
        var dataPerTopicPartition: ArrayBuffer[KeyedMessage[K,Message]] = null
        dataPerBroker.get(topicAndPartition) match {
          case Some(element) =>     // Partition对应的消息集又是一个Seq,如果存在,直接使用
            dataPerTopicPartition = element.asInstanceOf[ArrayBuffer[KeyedMessage[K,Message]]]
          case None =>              // Partition对应的消息集还不存在,创建一个列表, 并放入这个Broker Map里
            dataPerTopicPartition = new ArrayBuffer[KeyedMessage[K,Message]]
            dataPerBroker.put(topicAndPartition, dataPerTopicPartition)
        }
        // 到这里,才是真正将消息添加到集合中
        dataPerTopicPartition.append(message)
      }
      Some(ret)
  }
```

3. 消息分组

上面返回的结构包括了每个Broker的消息集,在实际处理时,会针对每个Broker的消息进一步分组.
groupMessagesToSet关于消息的输入和输出类型由 KeyedMessage转为了 ByteBufferMessageSet

```scala
  private def groupMessagesToSet(messagesPerTopicAndPartition: collection.mutable.Map[TopicAndPartition, Seq[KeyedMessage[K, Message]]]) = {
      val messagesPerTopicPartition = messagesPerTopicAndPartition.map { case (topicAndPartition, messages) =>
        // KeyedMessage包括了Key,Value, 其中message就是value原始数据
        val rawMessages = messages.map(_.message)
        // 输入是个Map(用case元组匹配),返回值也是元组,也会转成Map: key没有变化,value将Seq[Message]转成了MessageSet
        (topicAndPartition, config.compressionCodec match {
            case NoCompressionCodec => new ByteBufferMessageSet(NoCompressionCodec, rawMessages: _*)
            case _ => new ByteBufferMessageSet(config.compressionCodec, rawMessages: _*)
          }
        )
      }
      Some(messagesPerTopicPartition)
  }
```

在有了上面这些基础后,我们来看DefaultEventHandler如何处理一批不同 topic的消息集.

## DefaultEventHandler

事件处理器首先序列化,然后通过dispatchSerializedData发送消息,这里还带了重试发送功能.

```scala
  def handle(events: Seq[KeyedMessage[K,V]]) {
    val serializedData = serialize(events)              // ① 序列化事件
    var outstandingProduceRequests = serializedData     // 未完成的请求
    var remainingRetries = config.messageSendMaxRetries + 1
    while (remainingRetries > 0 && outstandingProduceRequests.size > 0) {
      outstandingProduceRequests = dispatchSerializedData(outstandingProduceRequests)  // ② 分发数据
      if (outstandingProduceRequests.size > 0) {        // 有返回值表示出错的,未完成的,继续重试
        remainingRetries -= 1                           // 重试次数减1
      }
    }
  }
```

在发送消息过程中,只要有失败的消息就加入到failedProduceRequests,这样返回的集合不为空,就会重试

```scala
  private def dispatchSerializedData(messages: Seq[KeyedMessage[K,Message]]): Seq[KeyedMessage[K, Message]] = {
    val partitionedDataOpt = partitionAndCollate(messages)                      // ① BrokerId -> (TopicAndPartition -> Seq[KeyedMessage])
    partitionedDataOpt match {
      case Some(partitionedData) =>
        val failedProduceRequests = new ArrayBuffer[KeyedMessage[K, Message]]
        for ((brokerid, messagesPerBrokerMap) <- partitionedData) {             // ② 每个BrokerId都有Partition->Messages的Map
          val messageSetPerBrokerOpt = groupMessagesToSet(messagesPerBrokerMap) // ③ 消息分组,如果有压缩,则对value压缩
          messageSetPerBrokerOpt match {
            case Some(messageSetPerBroker) =>
              val failedTopicPartitions = send(brokerid, messageSetPerBroker)   // ④ 发送消息集,返回值表示发送失败的Partitions
              failedTopicPartitions.foreach(topicPartition => {
                messagesPerBrokerMap.get(topicPartition) match {                // 在全集中找出属于这个Partition的消息
                  case Some(data) => failedProduceRequests.appendAll(data)      // 添加到failed列表中,在重试时会使用failed继续发送
                  case None =>  // nothing, 所有的消息都发送成功
                }
              })
            case None => messagesPerBrokerMap.values.foreach(m => failedProduceRequests.appendAll(m))  // failed to group messages
          }
        }
        failedProduceRequests
      case None => messages     // failed to collate messages
    }
  }
```

发送消息,从生产者池中获取SyncProducer(每个 Broker一个 Producer),将消息集封装到ProducerRequest,调用Producer.send发送请求
java版本的发送请求是创建 ProduceRequest-RequestSend-ClientRequest.并交给Sender-NetworkClient处理.

```scala
  private def send(brokerId: Int, messagesPerTopic: collection.mutable.Map[TopicAndPartition, ByteBufferMessageSet]) = {
      val currentCorrelationId = correlationId.getAndIncrement
      val producerRequest = new ProducerRequest(currentCorrelationId, config.clientId, config.requestRequiredAcks, config.requestTimeoutMs, messagesPerTopic)
      val syncProducer = producerPool.getProducer(brokerId)
      val response = syncProducer.send(producerRequest)
  }
```

发送消息,只需要指定brokerId,以及消息内容(TopicPartition->MessageSet),同时指定本请求是否需要ack和超时时间.

```scala
case class ProducerRequest(versionId: Short = ProducerRequest.CurrentVersion, correlationId: Int, clientId: String, 
                           requiredAcks: Short, ackTimeoutMs: Int, data: collection.mutable.Map[TopicAndPartition, ByteBufferMessageSet])
```

## ProducerPool

每个Broker都有一个 SyncProducer,因为同步的Producer一次只会有一个请求发生在 Broker上.
如果请求没有结束会一直阻塞的,其他请求没机会执行,所以没有必要一个Broker有多个 Producer实例.

```scala
class ProducerPool(val config: ProducerConfig) extends Logging {
  private val syncProducers = new HashMap[Int, SyncProducer]
  private val lock = new Object()

  def updateProducer(topicMetadata: Seq[TopicMetadata]) {
    val newBrokers = new collection.mutable.HashSet[BrokerEndPoint]
    // 首先根据topicMetadata找出所有的Leader Broker.  
    topicMetadata.foreach(tmd => {
      tmd.partitionsMetadata.foreach(pmd => {
        if(pmd.leader.isDefined) {
          newBrokers += pmd.leader.get
        }
      })
    })
    // 每个Broker都创建一个同步类型的SyncProducer,并放入缓存中,等待getProducer获取
    lock synchronized {
      newBrokers.foreach(b => {
        if(syncProducers.contains(b.id)){
          syncProducers(b.id).close()
          syncProducers.put(b.id, ProducerPool.createSyncProducer(config, b))
        } else
          syncProducers.put(b.id, ProducerPool.createSyncProducer(config, b))
      })
    }
  }

  def getProducer(brokerId: Int) : SyncProducer = {
    lock.synchronized {
      val producer = syncProducers.get(brokerId)
      producer match {
        case Some(p) => p
        case None => throw new UnavailableProducerException("Sync producer for broker id %d does not exist".format(brokerId))
      }
    }
  }
}
```

创建SyncProducer,需要指定Broker的地址,因为这个Producer会负责和 Broker通信,消息通过Producer发送到 Broker.

```scala
    object ProducerPool {
    // Used in ProducerPool to initiate a SyncProducer connection with a broker.
    def createSyncProducer(config: ProducerConfig, broker: BrokerEndPoint): SyncProducer = {
        val props = new Properties()
        props.put("host", broker.host)
        props.put("port", broker.port.toString)
        props.putAll(config.props.props)
        new SyncProducer(new SyncProducerConfig(props))
    }
    }
```

## SyncProducer

如果请求需要ack,则需要返回ProducerResponse给客户端(返回的消息内容放在response的 payload字节缓冲区中).

```scala
  private val blockingChannel = new BlockingChannel(config.host, config.port, BlockingChannel.UseDefaultBufferSize, config.sendBufferBytes, config.requestTimeoutMs)

  def send(producerRequest: ProducerRequest): ProducerResponse = {
    val readResponse = if(producerRequest.requiredAcks == 0) false else true
    var response: NetworkReceive = doSend(producerRequest, needAcks)
    if(readResponse) ProducerResponse.readFrom(response.payload)
    else null  // 如果生产请求需要响应(等待Leader响应, 或者除了Leader,ISR也要响应),从response响应读取内容设置到ProducerResponse对象中.否则直接返回
  }
  private def doSend(request: RequestOrResponse, readResponse: Boolean = true): NetworkReceive = {
      verifyRequest(request)
      getOrMakeConnection()                                 // 创建连接,如果已经有连接,则直接使用已有的连接通道发送数据
      var response: NetworkReceive = null                   // 准备一个NetworkReceive,如果有响应,会读取服务端的响应到这里
      blockingChannel.send(request)                         // 向阻塞类型的连接通道发送请求
      if(readResponse) response = blockingChannel.receive() // 从阻塞类型的连接通道读取响应    
      response 
  }
```

这里还有一个send方法,不过发送的是TopicMetadata请求,这个请求一定是有响应的.

```scala
  def send(request: TopicMetadataRequest): TopicMetadataResponse = {
    val response = doSend(request)
    TopicMetadataResponse.readFrom(response.payload)
  }
```

ProducerResponse和 TopicMetadataResponse的 readFrom参数都是 ByteBuffer,类似于反序列化.
发生在BlockingChannel的读写操作,前提是先建立连接,所以在doSend之前会 getOrMakeConnection.
注意: 由于一个Broker只有一个 SyncProducer,一个Producer也就只有一个 BlockingChannel.

```scala
  private def getOrMakeConnection() {
    if(!blockingChannel.isConnected) connect()
  }

  private def connect(): BlockingChannel = {
    if (!blockingChannel.isConnected && !shutdown) {
      blockingChannel.connect()
    }
    blockingChannel
  }
```

## BlockingChannel

实际的发送请求是交给BlockingChannel,它实现了I/O中的连接 connect,发送请求send,接收响应receive
从它的名字看出这是一个阻塞类型的Channel,所以并没有用到NIO的多路选择特性,难怪这是Old的设计.

```scala
class BlockingChannel( val host: String,  val port: Int,  val readBufferSize: Int,  val writeBufferSize: Int,  val readTimeoutMs: Int ) extends Logging {
  private var connected = false
  private var channel: SocketChannel = null
  private var readChannel: ReadableByteChannel = null
  private var writeChannel: GatheringByteChannel = null
  private val lock = new Object()
  private val connectTimeoutMs = readTimeoutMs
  private var connectionId: String = ""

  def connect() = lock synchronized  {
    if(!connected) {
        channel = SocketChannel.open()
        if(readBufferSize > 0) channel.socket.setReceiveBufferSize(readBufferSize)
        if(writeBufferSize > 0) channel.socket.setSendBufferSize(writeBufferSize)
        channel.configureBlocking(true)
        channel.socket.connect(new InetSocketAddress(host, port), connectTimeoutMs)

        writeChannel = channel
        readChannel = Channels.newChannel(channel.socket().getInputStream)
        connected = true
        connectionId = localHost + ":" + localPort + "-" + remoteHost + ":" + remotePort
    }
  }
}
```

KafkaProducer构造的请求是从 ProduceRequest到 RequestSend最后形成 ClientRequest中.这里把ProduceRequest
(是一种RequestOrResponse)封装到RequestOrResponseSend,他们都是ByteBufferSend的子类.

![](./20160122145250082)

```scala
  def send(request: RequestOrResponse): Long = {
    val send = new RequestOrResponseSend(connectionId, request)
    send.writeCompletely(writeChannel)
  }

  def receive(): NetworkReceive = {
    val response = readCompletely(readChannel)
    response.payload().rewind()     //读取到响应的ByteBuffer,回到缓冲区的最开始,便于读取
    response                        //返回响应, 如果客户端需要ack,则直接使用response.payload即可
  }
  private def readCompletely(channel: ReadableByteChannel): NetworkReceive = {
    val response = new NetworkReceive
    while (!response.complete())
      response.readFromReadableChannel(channel)
    response
  }
```

SyncProducer是阻塞类型的,所以并没有像java版本的 NetworkClient使用 Selector非阻塞异步模式.
