# Zookeeper: A Distributed Coordination Service for Distributed Applications

synchronization
configuration maintenance
groups and naming

a shared hierarchal namespace which is organized similarly to a standard file system. 
in-memory
strict ordering

ensemble

![](./zkservice.jpg)

in-memory image of state
transaction logs and snapshots in a persistent store
a majority of servers are available means ZK service will be avialable

Zookeeper stamps each update with a number that reflects the order of all Zookeeper transactions.
synchronization primitives

read-dominant

Data model and the hierarchical namespace

![](./zknamespace.jpg)

Nodes and ephemeral nodes
ZooKeeper was designed to store coordination data: status information, configuration, location information, etc., so the data stored at each node is usually small, in the byte to kilobyte range.
Ephemeral nodes exists as long as the session that created the znode is active.

Watches
Client can set a watch opon a znode
triggered and removed when the znode changes

* Sequential Consistency 
* Atomicity
* Single System Image
* Reliability 
* Timeliness 

Simple API 
create - creates a node at a location in the tree
delete - deletes a node
exists - tests if a node exists at a location
get data - reads the data from a node
set data - writes data to a node
get children - retrieves a list of children of a node
sync - waits for data to be propagated

Implementation

![](./zkcomponents.jpg)

The replicated database is an in-memory database containing the entire data tree. Updates are logged to disk for recoverability, and writes are serialized to disk before they are applied to the in-memory database.

Data Access - ACL



## That's just for a memo

> [Linux 基础](http://blog.csdn.net/wireless_com/article/details/52534823)
