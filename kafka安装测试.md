# kafka安装测试



## 解压

```

[app@h101 ~]$ tar -zxvf kafka_2.12-2.1.0.tgz 
[app@h101 ~]$ ln -s kafka_2.12-2.1.0 kafka
[app@h101 ~]$ rm -rf *.tgz


```



```
[app@h101 ~]$ cd kafka
[app@h101 kafka]$ bin/zookeeper-server-start.sh config/zookeeper.properties

```



## 单机测试

```
[app@h101 ~]$ cd kafka
[app@h101 kafka]$ bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
Created topic "test".

[app@h101 kafka]$ bin/kafka-topics.sh --list --zookeeper localhost:2181
test

[app@h101 kafka]$ bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
>111111111111111111111111111
>222222222222222222222222222

[app@h101 kafka]$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
111111111111111111111111111
222222222222222222222222222

关闭后再重启，还是能消费到2条消息
[app@h101 kafka]$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
111111111111111111111111111
222222222222222222222222222


去掉--from-beginning就没有再重复消费到了

```



## 集群配置



```
[hadoop@h101 ~]$ cexec zkServer.sh start
************************* h *************************
--------- h101---------
bash: zkServer.sh: 未找到命令
--------- h102---------
ZooKeeper JMX enabled by default
Using config: /home/hadoop/zookeeper-3.4.12/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
--------- h103---------
ZooKeeper JMX enabled by default
Using config: /home/hadoop/zookeeper-3.4.12/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
--------- h104---------
ZooKeeper JMX enabled by default
Using config: /home/hadoop/zookeeper-3.4.12/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[hadoop@h101 ~]$ cexec zkServer.sh status
************************* h *************************
--------- h101---------
bash: zkServer.sh: 未找到命令
--------- h102---------
ZooKeeper JMX enabled by default
Using config: /home/hadoop/zookeeper-3.4.12/bin/../conf/zoo.cfg
Mode: follower
--------- h103---------
ZooKeeper JMX enabled by default
Using config: /home/hadoop/zookeeper-3.4.12/bin/../conf/zoo.cfg
Mode: follower
--------- h104---------
ZooKeeper JMX enabled by default
Using config: /home/hadoop/zookeeper-3.4.12/bin/../conf/zoo.cfg
Mode: leader

```



修改server.properties 

> [app@h101 config]$ vi server.properties 

```
broker.id=1
zookeeper.connect=h102:2181,h103:2181,h104:2181
num.partitions=3
```

```
[app@h101 ~]$ cpush kafka_2.12-2.1.0/ /home/app/kafka_2.12-2.1.0/
```

设置kafka环境变量

```
export KAFKA_HOME=/home/app/kafka
export PATH=$KAFKA_HOME/bin:$PATH
```



```
[app@h101 ~]$ cpush .bash_profile /home/app/.bash_profile 
[app@h101 ~]$ cpush .bashrc /home/app/.bashrc
[app@h101 ~]$ cexec which kafka-server-start.sh
************************* h *************************
--------- h101---------
/home/app/kafka/bin/kafka-server-start.sh
--------- h102---------
/home/app/kafka/bin/kafka-server-start.sh
--------- h103---------
/home/app/kafka/bin/kafka-server-start.sh
--------- h104---------
/home/app/kafka/bin/kafka-server-start.sh
```





启动kafka

```
kafka-server-start.sh -daemon  /home/app/kafka/config/server.properties
```



先启动2台

```
[app@h101 ~]$ kafka-server-start.sh -daemon  /home/app/kafka/config/server.properties
[app@h101 ~]$ jps
2800 Kafka
6020 Jps
2502 QuorumPeerMain
[app@h101 ~]$ ssh h102
Last login: Thu Jan 17 15:42:56 2019 from h101
[app@h102 ~]$ kafka-server-start.sh -daemon  /home/app/kafka/config/server.properties
[app@h102 ~]$ jps
3324 Kafka
3341 Jps

```



检查zookeeper

```
[zk: localhost:2181(CONNECTED) 0] ls /
[log_dir_event_notification, consumers, latest_producer_id_block, rmstore, yarn-leader-election, controller_epoch, isr_change_notification, hbase, hadoop-ha, zookeeper, admin, cluster, config, controller, brokers]
[zk: localhost:2181(CONNECTED) 1] ls /brokers
[seqid, topics, ids]
[zk: localhost:2181(CONNECTED) 2] ls /brokers/topics
[]
[zk: localhost:2181(CONNECTED) 3] 

```



创建主题

报错：

```
[app@h101 ~]$ kafka-topics.sh --create --zookeeper h102:2181,h103:2181,h104:2181 --replication-factor 3 --partitions 4 --topic topic2
Error while executing topic command : Replication factor: 3 larger than available brokers: 1.
[2019-01-17 15:54:50,180] ERROR org.apache.kafka.common.errors.InvalidReplicationFactorException: Replication factor: 3 larger than available brokers: 1.
 (kafka.admin.TopicCommand$)

```



```
[zk: localhost:2181(CONNECTED) 4] ls /brokers
[seqid, topics, ids]
[zk: localhost:2181(CONNECTED) 5] ls /brokers/ids
[2]

```



再在h103上启动，启动之后，zookeeper上的brokers也多了一个，

```
[app@h103 ~]$ kafka-server-start.sh -daemon  /home/app/kafka/config/server.properties
[app@h103 ~]$ 

```

```
[zk: localhost:2181(CONNECTED) 6] ls /brokers/ids
[3, 2]

```

为什么没有id为1的broker呢？

那是因为刚刚单机启动的没有关掉，导致1的id实际没有启动成功，杀掉原来的进程再重启

还是失败，去掉-deamon参数，在前端启动，看下报错日志

```
 kafka-server-start.sh  /home/app/kafka/config/server.properties
```



```
kafka.common.InconsistentBrokerIdException: Configured broker.id 1 doesn't match stored broker.id 0 in meta.properties. If you moved your data, make sure your configured broker.id matches. If you intend to create a new broker, you should remove all data in your data directories (log.dirs).
	at kafka.server.KafkaServer.getBrokerIdAndOfflineDirs(KafkaServer.scala:689)
	at kafka.server.KafkaServer.startup(KafkaServer.scala:209)
	at kafka.server.KafkaServerStartable.startup(KafkaServerStartable.scala:38)
	at kafka.Kafka$.main(Kafka.scala:75)
	at kafka.Kafka.main(Kafka.scala)
```

参考

[Kafka：Configured broker.id 2 doesn't match stored broker.id 0 in meta.properties.](https://www.cnblogs.com/gudi/p/7847100.html)

```
log.dirs目录下的meta.properties中配置的broker.id和配置目录下的server.properties中的broker.id不一致了，解决问题的方法是将两者修改一致后再重启
```



```
[app@h101 kafka-logs]$ pwd
/tmp/kafka-logs
[app@h101 kafka-logs]$ ls
cleaner-offset-checkpoint  __consumer_offsets-15  __consumer_offsets-22  __consumer_offsets-3   __consumer_offsets-37  __consumer_offsets-44  __consumer_offsets-7
__consumer_offsets-0       __consumer_offsets-16  __consumer_offsets-23  __consumer_offsets-30  __consumer_offsets-38  __consumer_offsets-45  __consumer_offsets-8
__consumer_offsets-1       __consumer_offsets-17  __consumer_offsets-24  __consumer_offsets-31  __consumer_offsets-39  __consumer_offsets-46  __consumer_offsets-9
__consumer_offsets-10      __consumer_offsets-18  __consumer_offsets-25  __consumer_offsets-32  __consumer_offsets-4   __consumer_offsets-47  log-start-offset-checkpoint
__consumer_offsets-11      __consumer_offsets-19  __consumer_offsets-26  __consumer_offsets-33  __consumer_offsets-40  __consumer_offsets-48  meta.properties
__consumer_offsets-12      __consumer_offsets-2   __consumer_offsets-27  __consumer_offsets-34  __consumer_offsets-41  __consumer_offsets-49  recovery-point-offset-checkpoint
__consumer_offsets-13      __consumer_offsets-20  __consumer_offsets-28  __consumer_offsets-35  __consumer_offsets-42  __consumer_offsets-5   replication-offset-checkpoint
__consumer_offsets-14      __consumer_offsets-21  __consumer_offsets-29  __consumer_offsets-36  __consumer_offsets-43  __consumer_offsets-6   test-0

```

这里先直接删掉这个目录吧。

```
[app@h101 tmp]$ rm -rf /tmp/kafka-logs/

```



再启动就成功了，zookeeper中也有id=1的信息了

```
[zk: localhost:2181(CONNECTED) 9] ls /brokers/ids
[3, 2, 1]

```

ctrl +c 停止后，再加-deamon参数后台启动

```
[app@h101 ~]$ kafka-server-start.sh -daemon  /home/app/kafka/config/server.properties

```



## 参考

http://kafka.apache.org/documentation/#quickstart

https://www.cnblogs.com/5iTech/articles/6043224.html

[kafka2.1.0高可用HA集群搭建方法](https://blog.csdn.net/hsg77/article/details/84934227)

[Kafka：Configured broker.id 2 doesn't match stored broker.id 0 in meta.properties.](https://www.cnblogs.com/gudi/p/7847100.html)







