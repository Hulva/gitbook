# Kafka High Availability

> [原文](http://www.jasongj.com/2015/04/24/KafkaColumn2/)

# 摘要
>　Kafka在0.8以前的版本中，并不提供High Availablity机制，一旦一个或多个Broker宕机，则宕机期间其上所有Partition都无法继续提供服务。若该Broker永远不能再恢复，亦或磁盘故障，则其上数据将丢失。而Kafka的设计目标之一即是提供数据持久化，同时对于分布式系统来说，尤其当集群规模上升到一定程度后，一台或者多台机器宕机的可能性大大提高，对于Failover机制的需求非常高。因此，Kafka从0.8开始提供High Availability机制。本文从Data Replication和Leader Election两方面介绍了Kafka的HA机制。

# Replication

> 在Kafka在0.8以前的版本中，是没有Replication的，一旦某一个Broker宕机，则其上所有的Partition数据都不可被消费，这与Kafka数据持久性及Delivery Guarantee的设计目标相悖。同时Producer都不能再将数据存于这些Partition中。
    >* 如果Producer使用同步模式则Producer会在尝试重新发送message.send.max.retries（默认值为3）次后抛出Exception，用户可以选择停止发送后续数据也可选择继续选择发送。而前者会造成数据的阻塞，后者会造成本应发往该Broker的数据的丢失。
    >* 如果Producer使用异步模式，则Producer会尝试重新发送message.send.max.retries（默认值为3）次后记录该异常并继续发送后续数据，这会造成数据丢失并且用户只能通过日志发现该问题。
    
> 由此可见，在没有Replication的情况下，一旦某机器宕机或者某个Broker停止工作则会造成整个系统的可用性降低。随着集群规模的增加，整个集群中出现该类异常的几率大大增加，因此对于生产系统而言Replication机制的引入非常重要。 　　

# Leader Election

>（此处的Leader Election主要指Replica之间的Leader Election）
    
引入Replication之后，同一个Partition可能会有多个Replica，而这时需要在这些Replication之间选出一个Leader，Producer和Consumer只与这个Leader交互，其它Replica作为Follower从Leader中复制数据。

因为需要保证同一个Partition的多个Replica之间的数据一致性（其中一个宕机后其它Replica必须要能继续服务并且即不能造成数据重复也不能造成数据丢失）。如果没有一个Leader，所有Replica都可同时读/写数据，那就需要保证多个Replica之间互相（N×N条通路）同步数据，数据的一致性和有序性非常难保证，大大增加了Replication实现的复杂性，同时也增加了出现异常的几率。而引入Leader后，只有Leader负责数据读写，Follower只向Leader顺序Fetch数据（N条通路），系统更加简单且高效。

# 当kafka集群中的broker挂掉的影响：
1.	此broker上所有的partition都不是leader时：这种情况对使用这个集群的producer和consumer的影响是没有的，以为producer的写与consumer的读操作的对象是消息的partition的leader。
2.	此broker上存在有作为leader的partition：

   * a) Broker在当前message写入本地log之前挂掉了。这种情况下，客户端会因为超时而重新发送（重试次数配置message.send.max.retries）这条message给新的leader
   * b) Broker在message写入本地log之后，但在向客户端回复之前挂掉了。这种情况下，客户端同样是会因为超时而重新发送这条message，但是这样就可能会出现消息被写入两遍的问题，那就是，虽然leader挂掉了，但其中的一个副本已经写入了这条消息，并且这个副本被选为了新的leader。
   * c) Broker在消息写入并回复了客户端之后挂掉了。这种情况下，新的leader会被选出并接收请求。
   * d) 另外，如果producer配置 acks 为0即不等待来自broker的回复，这是a)情形将会丢数据。

> Leader挂掉后新leader选举的过程（[源码]( https://github.com/apache/kafka/blob/0.8.2/core/src/main/scala/kafka/controller/PartitionLeaderSelector.scala) OfflinePartitionLeaderSelector）

>（Kafka 0.8.*之后，broker中的一员将作为controller，所有partition的leader的选举都由controller决定。Controller直接将leader的变动直接通过RPC的方式通知需要做出变动的broker。）
1. ISR（In Sync Replica, 同步的副本）中有至少一个broker存活，直接在从中选一个作为新的leader（从源码中看就是选ISR列表中的第一个）
2. 存在有不在ISR中的副本，且配置项unclean.leader.election.enable为true的前提下，选存活的副本中的一个作为新的leader（同样是存活副本列表的第一个）。这种情况非常大的概率会丢数据。
3. 没有存活的副本，这种情况就没辙了。

另外，如果挂掉的那个broker刚好又是controller！作为controller的broker会在zk上创建/controller节点，其内容大致为：
```
    get /controller
    {"version":1,"brokerid":91,"timestamp":"1483079402422"}
    cZxid = 0xa00000a20
    ctime = Fri Dec 30 14:29:57 CST 2016
    mZxid = 0xa00000a20
    mtime = Fri Dec 30 14:29:57 CST 2016
    pZxid = 0xa00000a20
    cversion = 0
    dataVersion = 0
    aclVersion = 0
    ephemeralOwner = 0x258d2938a4e0000
    dataLength = 55
    numChildren = 0
```
其他的broker会监听这个节点，一旦controller挂掉，其他broker监听到这一变动后便会争相创建这个节点，创建成功的成为新一代controller。这个过程非常快，对生产消费几乎没有影响。
