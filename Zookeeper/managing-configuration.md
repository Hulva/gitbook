# Managing configuration with Apache Zookeeper

Default ZNode size limit is 1MB, can be changed by `jute.maxbuffer`
JVM heap settings(max. 3/4 of total amount of memory)
Restrict access to specific environments/services, protect config entry from unapproved changes by setting read-only access to it.
ZooKeeper doesn't track changes to specific nodes itself, so you need to have separate backup/version control systems. ZooKeeper不会跟踪对特定节点本身的更改，因此您需要具有单独的备份/版本控制系统。
