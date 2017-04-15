# Hadoop Ecosystem

> Hadoop is an open-source framework to store and process Big Data in a distributed environment. It contains two modules, one is MapReduce and another is Hadoop Distributed File System (HDFS).

>* MapReduce: It is a parallel programming model for processing large amounts of structured, semi-structured, and unstructured data on large clusters of commodity hardware.

>* HDFS:Hadoop Distributed File System is a part of Hadoop framework, used to store and process the datasets. It provides a fault-tolerant file system to run on commodity hardware.

> The Hadoop ecosystem contains different sub-projects (tools) such as Sqoop, Pig, and Hive that are used to help Hadoop modules.

>* Sqoop: 用来在 HDFS 和 RDBMS 之间导入或导出数据

>* Pig: It is a procedural language platform(过程语言) used to develop a script for MapReduce operations.

>* Hive: It is a platform used to develop SQL type scripts to do MapReduce operations.

> Note: There are various ways to execute MapReduce operations:

>* The traditional approach using Java MapReduce program for structured, semi-structured, and unstructured data.
使用Java MapReduce 程序来进行结构化，半结构化和非结构化数据。
>* The scripting approach for MapReduce to process structured and semi structured data using Pig.
使用Pig脚本处理结构化和半结构化数据。
>* The Hive Query Language (HiveQL or HQL, Hive查询语言) for MapReduce to process structured data using Hive.

## Hadoop docker 镜像制作
```
base: docker.neg/java8:latest
yum install openssh-server wget
# install hadoop 2.7.2
RUN wget https://github.com/kiwenlau/compile-hadoop/releases/download/2.7.2/hadoop-2.7.2.tar.gz && \
    tar -xzvf hadoop-2.7.2.tar.gz && \
    mv hadoop-2.7.2 /usr/local/hadoop && \
    rm hadoop-2.7.2.tar.gz
# set environment variable
ENV JAVA_HOME=/usr/java/latest 
ENV HADOOP_HOME=/usr/local/hadoop 
ENV PATH=$PATH:/usr/local/hadoop/bin:/usr/local/hadoop/sbin 
# ssh without key
RUN ssh-keygen -t rsa -f ~/.ssh/id_rsa -P '' && \
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
RUN mkdir -p ~/hdfs/namenode && \ 
    mkdir -p ~/hdfs/datanode && \
    mkdir $HADOOP_HOME/logs
COPY config/* /tmp/
RUN mv /tmp/ssh_config ~/.ssh/config && \
    mv /tmp/hadoop-env.sh /usr/local/hadoop/etc/hadoop/hadoop-env.sh && \
    mv /tmp/hdfs-site.xml $HADOOP_HOME/etc/hadoop/hdfs-site.xml && \ 
    mv /tmp/core-site.xml $HADOOP_HOME/etc/hadoop/core-site.xml && \
    mv /tmp/mapred-site.xml $HADOOP_HOME/etc/hadoop/mapred-site.xml && \
    mv /tmp/yarn-site.xml $HADOOP_HOME/etc/hadoop/yarn-site.xml && \
    mv /tmp/slaves $HADOOP_HOME/etc/hadoop/slaves && \
    mv /tmp/start-hadoop.sh ~/start-hadoop.sh && \
    mv /tmp/run-wordcount.sh ~/run-wordcount.sh
RUN chmod +x ~/start-hadoop.sh && \
    chmod +x ~/run-wordcount.sh && \
    chmod +x $HADOOP_HOME/sbin/start-dfs.sh && \
    chmod +x $HADOOP_HOME/sbin/start-yarn.sh 
# format namenode
RUN /usr/local/hadoop/bin/hdfs namenode -format
CMD [ "sh", "-c", "service ssh start; bash"]
```
