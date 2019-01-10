# hbase集群安装

h101 h102 h103

安装到这3台中去。

## 上传解压

```
[hadoop@h101 ~]$ tar -zxvf hbase-1.3.3-bin.tar.gz 
```



## 修改hbase-env.sh

```
[hadoop@h101 ~]$ cd hbase-1.3.3/conf
vi hbase-env.sh
```



修改以下配置

```
export JAVA_HOME=/usr/local/java7
export HBASE_MANAGES_ZK=false
```



## 修改hbase-site.xml

```
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://hacluster/hbase</value>
  </property>

  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>

  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>h102,h103,h104</value>
  </property>
</configuration>
```



## 修改regionservers

>  *conf/regionservers* file, which contains a list of nodes that should run a RegionServer in the cluster. These nodes need HBase installed and they need to use the same contents of the *conf/* directory as the Master server

```
h101
h102
h103
```



## 添加*backup-masters*

> *conf/backup-masters* file, which contains a list of each node that should run a backup Master instance. The backup Master instances will sit idle unless the main Master becomes unavailable.

```
h102
h103
```



## 配置hdfs-site.xml

```
Procedure: HDFS Client Configuration
Of note, if you have made HDFS client configuration changes on your Hadoop cluster, such as configuration directives for HDFS clients, as opposed to server-side configurations, you must use one of the following methods to enable HBase to see and use these configuration changes:

a. Add a pointer to your HADOOP_CONF_DIR to the HBASE_CLASSPATH environment variable in hbase-env.sh.

b. Add a copy of hdfs-site.xml (or hadoop-site.xml) or, better, symlinks, under ${HBASE_HOME}/conf, or

c. if only a small set of HDFS client configurations, add them to hbase-site.xml.

An example of such an HDFS client configuration is dfs.replication. If for example, you want to run with a replication factor of 5, HBase will create files with the default of 3 unless you do the above to make the configuration available to HBase.

```

这里直接采用b方法：

将hdfs-site.xml文件拷贝过来

```
cp /home/hadoop/hadoop-2.7.7/etc/hadoop/hdfs-site.xml /home/hadoop/hbase-1.3.3/conf
```

这里hadoop的jar包用的是2.5.1的。hadoop版本是2.7.7的

官方文档对这个情况有什么说明吗？

没找到，先不管这个情况吧。

```
[hadoop@h101 lib]$ pwd
/home/hadoop/hbase-1.3.3/lib
[hadoop@h101 lib]$ ls|grep hadoop
hadoop-annotations-2.5.1.jar
hadoop-auth-2.5.1.jar
hadoop-client-2.5.1.jar
hadoop-common-2.5.1.jar
hadoop-hdfs-2.5.1.jar
hadoop-mapreduce-client-app-2.5.1.jar
hadoop-mapreduce-client-common-2.5.1.jar
hadoop-mapreduce-client-core-2.5.1.jar
hadoop-mapreduce-client-jobclient-2.5.1.jar
hadoop-mapreduce-client-shuffle-2.5.1.jar
hadoop-yarn-api-2.5.1.jar
hadoop-yarn-client-2.5.1.jar
hadoop-yarn-common-2.5.1.jar
hadoop-yarn-server-common-2.5.1.jar
hbase-hadoop2-compat-1.3.3.jar
hbase-hadoop-compat-1.3.3.jar

```

## 确认服务器的时间是否同步

```
[hadoop@h101 conf]$ cexec date
************************* h *************************
--------- h101---------
2019年 01月 10日 星期四 20:30:14 CST
--------- h102---------
2019年 01月 10日 星期四 20:30:14 CST
--------- h103---------
2019年 01月 10日 星期四 20:30:14 CST
--------- h104---------
2019年 01月 10日 星期四 20:30:14 CST
```



## 拷贝hbase文件到其他机器

```
[hadoop@h101 ~]$ cpush hbase-1.3.3 /home/hadoop/
```

确认拷贝结果：

```
[hadoop@h101 ~]$ cexec ls
************************* h *************************
--------- h101---------
hadoop-2.7.7
hbase-1.3.3
hbase-1.3.3-bin.tar.gz
hdfstest.txt
run
--------- h102---------
hadoop-2.7.7
hbase-1.3.3
run
zkrun
zookeeper-3.4.12
zookeeper.out
--------- h103---------
hadoop-2.7.7
hbase-1.3.3
run
zkrun
zookeeper-3.4.12
zookeeper.out
--------- h104---------
hadoop-2.7.7
hbase-1.3.3
run
zkrun
zookeeper-3.4.12
zookeeper.out
```



## 配置hbase环境变量

```
export HBASE_HOME=/home/hadoop/hbase-1.3.3
export PATH=$HBASE_HOME/bin:$PATH
```



```
[hadoop@h101 ~]$ source .bash_profile
[hadoop@h101 ~]$ which start-hbase.sh
~/hbase-1.3.3/bin/start-hbase.sh
```



## 启动hbase

```
[hadoop@h101 ~]$ start-hbase.sh
starting master, logging to /home/hadoop/hbase-1.3.3/logs/hbase-hadoop-master-h101.out
h102: starting regionserver, logging to /home/hadoop/hbase-1.3.3/bin/../logs/hbase-hadoop-regionserver-h102.out
h103: starting regionserver, logging to /home/hadoop/hbase-1.3.3/bin/../logs/hbase-hadoop-regionserver-h103.out
h101: starting regionserver, logging to /home/hadoop/hbase-1.3.3/bin/../logs/hbase-hadoop-regionserver-h101.out
h103: starting master, logging to /home/hadoop/hbase-1.3.3/bin/../logs/hbase-hadoop-master-h103.out
h102: starting master, logging to /home/hadoop/hbase-1.3.3/bin/../logs/hbase-hadoop-master-h102.out
```



## 检查进程

```
[hadoop@h101 ~]$ cexec jps
************************* h *************************
--------- h101---------
3452 HMaster
1665 JournalNode
1430 NameNode
3964 Jps
1815 DFSZKFailoverController
3582 HRegionServer
--------- h102---------
1718 JournalNode
4521 Jps
1596 NodeManager
4135 HRegionServer
1400 DataNode
4255 HMaster
1327 NameNode
1830 DFSZKFailoverController
1247 QuorumPeerMain
1481 ResourceManager
--------- h103---------
2285 HMaster
2561 Jps
1402 NodeManager
1218 QuorumPeerMain
1299 DataNode
2180 HRegionServer
1514 JournalNode
--------- h104---------
2118 Jps
1191 QuorumPeerMain
1276 DataNode
1379 NodeManager
```



## 查看hbase-web

http://h101:16010/



## 检查hdfs上hbase的文件

```
[hadoop@h101 ~]$ hdfs dfs -ls /
Found 4 items
drwxr-xr-x   - hadoop supergroup          0 2019-01-10 20:35 /hbase
drwxr-xr-x   - hadoop supergroup          0 2019-01-10 19:23 /input
drwxr-xr-x   - hadoop supergroup          0 2019-01-10 19:25 /output
drwx------   - hadoop supergroup          0 2019-01-10 19:25 /tmp
[hadoop@h101 ~]$ hdfs dfs -ls /hbase
Found 8 items
drwxr-xr-x   - hadoop supergroup          0 2019-01-10 20:35 /hbase/.tmp
drwxr-xr-x   - hadoop supergroup          0 2019-01-10 20:36 /hbase/MasterProcWALs
drwxr-xr-x   - hadoop supergroup          0 2019-01-10 20:35 /hbase/WALs
drwxr-xr-x   - hadoop supergroup          0 2019-01-10 20:35 /hbase/data
drwxr-xr-x   - hadoop supergroup          0 2019-01-10 20:35 /hbase/hbase
-rw-r--r--   2 hadoop supergroup         42 2019-01-10 20:35 /hbase/hbase.id
-rw-r--r--   2 hadoop supergroup          7 2019-01-10 20:35 /hbase/hbase.version
drwxr-xr-x   - hadoop supergroup          0 2019-01-10 20:35 /hbase/oldWALs
[hadoop@h101 ~]$ hdfs dfs -ls /hbase/data
Found 2 items
drwxr-xr-x   - hadoop supergroup          0 2019-01-10 20:35 /hbase/data/default
drwxr-xr-x   - hadoop supergroup          0 2019-01-10 20:35 /hbase/data/hbase
```



## 验证高可用

访问3个master，各个页面显示内容（h101为active， h102和h103是backup）

http://h101:16010/

```
Master h101
```

http://h102:16010/

```
Backup Master h102
Current Active Master: h101
```

http://h103:16010/

```
Backup Master h103
Current Active Master: h101
```



kill掉h101的master进程

```
[hadoop@h101 ~]$ jps
3452 HMaster
4183 Jps
1665 JournalNode
1430 NameNode
1815 DFSZKFailoverController
3582 HRegionServer
[hadoop@h101 ~]$ kill 3452
```



各个页面的展示结果：

http://h101:16010/  无法打开

http://h102:16010/

```
Backup Master h102
Current Active Master: h103
```

http://h103:16010/

```
Master h103
```

active已经变味h103了。

再次启动h101的hmaster

```
[hadoop@h101 ~]$ hbase-daemon.sh  start master
starting master, logging to /home/hadoop/hbase-1.3.3/logs/hbase-hadoop-master-h101.out
```

此时访问h101

http://h101:16010/

显示h101为backup, h103为active

```
Backup Master h101
Current Active Master: h103
```



## 参考

http://hbase.apache.org/1.2/book.html#quickstart_fully_distributed

https://www.cnblogs.com/frankdeng/p/9310191.html







