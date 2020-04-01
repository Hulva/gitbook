# Kafka Controller

[原文](https://cwiki.apache.org/confluence/display/KAFKA/Kafka+Controller+Internals)

> 在Kafka集群中，broker中的一员将会被选为controller，controller的职责是管理partition和replica的states以及完成一些诸如重新分区的任务。

## PartitionStateChange

### 一个partition所拥有的有效状态有:
   * NonExistentPartition: 这个状态表示的是这个partition从未被创建或者是创建过但被删除了。
   * NewPartition: 在创建后，分区处于NewPartition状态。在这个状态下，partition应该有分配给它的副本，但还没有leader和isr。
   * OnlinePartition: 一旦一个partition的leader被选举出来了，它就处于OnlinePartition状态了。
   * OfflinePartition: 在成功的进行过leader选举之后，partition的leader死了的话，该partition就会别为OfflinePartition状态。

### Partition状态之间的转换:
   * NonExistentPartition->NewPartition
       1. 从zk加载已被分配好的副本（replica）到controller的缓存
   * NewPartition->OnlinePartition
       1. 将isr中的第一个replica指定为leader，将leader和isr信息写到zk
       2. 向所有存活的replica发送LeaderAndIsr请求，向所有存活的broker发送UpdateMetadata请求
   * OnlinePartition,OfflinePartition->OnlinePartition
       1. 为当前分区选择一个新的leader以及用于接收LeaderAndIsr请求的replica，并将leader和isr信息写入zk
           >* a. OfflinePartitionLeaderSelector: new leader = a live replica (preferably in isr); new isr = live isr if not empty or just the new leader if otherwise; receiving replicas = live assigned replicas
           >* b. ReassignedPartitionLeaderSelector: new leader = a live reassigned replica; new isr = current isr; receiving replicas = reassigned replicas
           >* c. PreferredReplicaPartitionLeaderSelector: new leader = first assigned replica (if in isr); new isr = current isr; receiving replicas = assigned replicas
           >* d. ControlledShutdownLeaderSelector: new leader = replica in isr that's not being shutdown; new isr = current isr - shutdown replica; receiving replicas = live assigned replicas
       2. 向所有存活的replica发送LeaderAndIsr请求，向所有存活的broker发送UpdateMetadata请求
   * NewPartition,OnlinePartition->OfflinePartition
       1. partition 的状态被标记为offline
   * OfflinePartition->NonExistentPartition
       1. partition的状态被标记为 NonExistentPartition

## ReplicaStateChange

### Valid states are:
* NewReplica: When replicas are created during topic creation or partition reassignment. In this state, a replica can only get become follower state change request. 
* OnlineReplica: Once a replica is started and part of the assigned replicas for its partition, it is in this state. In this state, it can get either become leader or become follower state change requests.
* OfflineReplica : If a replica dies, it moves to this state. This happens when the broker hosting the replica is down.
* NonExistentReplica: If a replica is deleted, it is moved to this state.

### Valid state transitions are:
* NonExistentReplica --> NewReplica
    1. send LeaderAndIsr request with current leader and isr to the new replica and UpdateMetadata request for the partition to every live broker
* NewReplica-> OnlineReplica
    1. add the new replica to the assigned replica list if needed
* OnlineReplica,OfflineReplica -> OnlineReplica
    1. send LeaderAndIsr request with current leader and isr to the new replica and UpdateMetadata request for the partition to every live broker
* NewReplica,OnlineReplica -> OfflineReplica
    1. send StopReplicaRequest to the replica (w/o deletion)
    2. remove this replica from the isr and send LeaderAndIsr request (with new isr) to the leader replica and UpdateMetadata request for the partition to every live broker.
* OfflineReplica -> NonExistentReplica
    1. send StopReplicaRequest to the replica (with deletion)
