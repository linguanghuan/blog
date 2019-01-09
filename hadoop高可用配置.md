# hadoop高可用配置

## 安装zookeeper集群

[zookeeper集群安装](zookeeper集群安装.md)



## 修改hadoop配置

### 修改core-site.xml

1. 将原来的fs.defaultFS

   从hdfs://h101:9000改为  hdfs://hacluster

2. 新增zookeeper配置

   ```
   <!-- 指定ZKFC故障自动切换转移 -->
        <property>
          <name>ha.zookeeper.quorum</name>
          <value>h102:2181,h103:2181,h104:2181</value>
        </property>
   ```

   修改后：

```
<configuration>
<!-- 指定HDFS中NameNode的地址 -->
     <property>
       <name>fs.defaultFS</name>
       <value>hdfs://h101:9000</value>
     </property>
<!-- 指定hadoop运行时产生文件的存储目录 -->
     <property>
       <name>hadoop.tmp.dir</name>
       <value>/home/hadoop/run</value>
     </property>
<!-- 指定ZKFC故障自动切换转移 -->
     <property>
       <name>ha.zookeeper.quorum</name>
       <value>h102:2181,h103:2181,h104:2181</value>
     </property>
</configuration>
```

### 修改hdfs-site.xml

```
<configuration>
<!-- 设置dfs副本数，不设置默认是3个   -->
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>

<!-- 完全分布式集群名称 -->
<property>
  <name>dfs.nameservices</name>
  <value>hacluster</value>
</property>

<!-- 集群中NameNode节点都有哪些 -->
<property>
   <name>dfs.ha.namenodes.hacluster</name>
   <value>nn1,nn2</value>
</property>
<!-- nn1的RPC通信地址 -->
<property>
   <name>dfs.namenode.rpc-address.hacluster.nn1</name>
   <value>h101:8020</value>
</property>
<!-- nn2的RPC通信地址 -->
<property>
   <name>dfs.namenode.rpc-address.hacluster.nn2</name>
   <value>h102:8020</value>
</property>
<!-- nn1的http通信地址 -->
<property>
   <name>dfs.namenode.http-address.hacluster.nn1</name>
   <value>h101:50070</value>
</property>
<!-- nn2的http通信地址 -->
<property>
    <name>dfs.namenode.http-address.hacluster.nn2</name>
    <value>h102:50070</value>
</property>
<!-- 指定NameNode元数据在JournalNode上的存放位置 -->
<property>
    <name>dfs.namenode.shared.edits.dir</name>
    <value>qjournal://h101:8485;h102:8485;h103:8485/hacluster</value>
</property>
<!-- 配置隔离机制，即同一时刻只能有一台服务器对外响应 -->
<property>
    <name>dfs.ha.fencing.methods</name>
    <value>sshfence</value>
</property>
<!-- 使用隔离机制时需要ssh无秘钥登录-->
<property>
    <name>dfs.ha.fencing.ssh.private-key-files</name>
    <value>/home/hadoop/.ssh/id_rsa</value>
</property>
<!-- 声明journalnode服务器存储目录-->
<property>
   <name>dfs.journalnode.edits.dir</name>
   <value>/home/hadoop/run/ha/jn</value>
</property>
<!-- 关闭权限检查-->
<property>
   <name>dfs.permissions.enable</name>
   <value>false</value>
</property>
<!-- 访问代理类：client，hacluster，active配置失败自动切换实现方式-->
<property>
   <name>dfs.client.failover.proxy.provider.hacluster</name>
   <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
</property>
<!-- 配置自动故障转移-->
<property>
   <name>dfs.ha.automatic-failover.enabled</name>
   <value>true</value>
</property>　
</configuration>
```



mapred-site.xml

```
<configuration>
    <!-- 指定mr运行在yarn上 -->
    <property>
     <name>mapreduce.framework.name</name>
     <value>yarn</value>
    </property>

<!-- 指定mr历史服务器主机,端口 -->
  <property>   
    <name>mapreduce.jobhistory.address</name>   
    <value>h101:10020</value>   
  </property>   
<!-- 指定mr历史服务器WebUI主机,端口 -->
  <property>   
    <name>mapreduce.jobhistory.webapp.address</name>   
    <value>h101:19888</value>   
  </property>
<!-- 历史服务器的WEB UI上最多显示20000个历史的作业记录信息 -->    
  <property>
    <name>mapreduce.jobhistory.joblist.cache.size</name>
    <value>20000</value>
  </property>
<!--配置作业运行日志 --> 
  <property>
    <name>mapreduce.jobhistory.done-dir</name>
    <value>${yarn.app.mapreduce.am.staging-dir}/history/done</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.intermediate-done-dir</name>
    <value>${yarn.app.mapreduce.am.staging-dir}/history/done_intermediate</value>
  </property>
  <property>
    <name>yarn.app.mapreduce.am.staging-dir</name>
    <value>/tmp/hadoop-yarn/staging</value>
  </property>

</configuration>
```

### 修改yarn-site.xml

```
<configuration>

<!-- Site specific YARN configuration properties -->

<!-- reducer获取数据的方式 -->
     <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
     </property>

   <!--启用resourcemanager ha-->
    <property>
        <name>yarn.resourcemanager.ha.enabled</name>
        <value>true</value>
    </property>
    <!--声明两台resourcemanager的地址-->
    <property>
        <name>yarn.resourcemanager.cluster-id</name>
        <value>rmCluster</value>
    </property>
    <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm1,rm2</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>h102</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname.rm2</name>
        <value>h103</value>
    </property>
    <!--指定zookeeper集群的地址-->
    <property>
        <name>yarn.resourcemanager.zk-address</name>
        <value>h102:2181,h103:2181,h104:2181</value>
    </property>
    <!--启用自动恢复-->
    <property>
        <name>yarn.resourcemanager.recovery.enabled</name>
        <value>true</value>
    </property>
    <!--指定resourcemanager的状态信息存储在zookeeper集群-->
    <property>
        <name>yarn.resourcemanager.store.class</name>    
        <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
    </property>

</configuration>
```



### 拷贝修改后的文件到其他机器

```
[hadoop@h101 ~]$ cpush hadoop-2.7.7 /home/hadoop/
```

### 启动集群



/home/hadoop/run/目录下如果有旧的东西，先删除旧的。



根据hdfs-site.xml的以下配置，在h101 h102 h103上启动Journal集群

```
<!-- 指定NameNode元数据在JournalNode上的存放位置 -->
<property>
    <name>dfs.namenode.shared.edits.dir</name>
    <value>qjournal://h101:8485;h102:8485;h103:8485/hacluster</value>
</property>
```





```
[hadoop@h101 ~]$ hadoop-daemon.sh start journalnode
starting journalnode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-journalnode-h101.out


[hadoop@h102 ha]$ hadoop-daemon.sh start journalnode
starting journalnode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-journalnode-h102.out

[hadoop@h103 ~]$ hadoop-daemon.sh start journalnode
starting journalnode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-journalnode-h103.out
```

查看启动结果：

```
[hadoop@h101 ~]$ cexec /usr/local/java7/bin/jps
************************* h *************************
--------- h101---------
2100 Jps
2013 JournalNode
--------- h102---------
1550 QuorumPeerMain
1950 JournalNode
2015 Jps
--------- h103---------
1407 QuorumPeerMain
1865 Jps
1800 JournalNode
--------- h104---------
1751 Jps
1394 QuorumPeerMain
```

启动后建立了目录/home/hadoop/run/ha/jn (配置了这个目录)

```
[hadoop@h101 ha]$ pwd
/home/hadoop/run/ha
[hadoop@h101 ha]$ find .
.
./jn
```







执行hdfs namenode -format

```
[hadoop@h101 ha]$ hdfs namenode -format
```

执行结果
```
[hadoop@h101 run]$ cexec "find /home/hadoop/run"
************************* h *************************
--------- h101---------
/home/hadoop/run
/home/hadoop/run/ha
/home/hadoop/run/ha/jn
/home/hadoop/run/ha/jn/hacluster
/home/hadoop/run/ha/jn/hacluster/current
/home/hadoop/run/ha/jn/hacluster/current/VERSION
/home/hadoop/run/ha/jn/hacluster/current/paxos
/home/hadoop/run/ha/jn/hacluster/in_use.lock
/home/hadoop/run/dfs
/home/hadoop/run/dfs/name
/home/hadoop/run/dfs/name/current
/home/hadoop/run/dfs/name/current/VERSION
/home/hadoop/run/dfs/name/current/seen_txid
/home/hadoop/run/dfs/name/current/fsimage_0000000000000000000.md5
/home/hadoop/run/dfs/name/current/fsimage_0000000000000000000
--------- h102---------
/home/hadoop/run
/home/hadoop/run/ha
/home/hadoop/run/ha/jn
/home/hadoop/run/ha/jn/hacluster
/home/hadoop/run/ha/jn/hacluster/current
/home/hadoop/run/ha/jn/hacluster/current/VERSION
/home/hadoop/run/ha/jn/hacluster/current/paxos
/home/hadoop/run/ha/jn/hacluster/in_use.lock
--------- h103---------
/home/hadoop/run
/home/hadoop/run/ha
/home/hadoop/run/ha/jn
/home/hadoop/run/ha/jn/hacluster
/home/hadoop/run/ha/jn/hacluster/current
/home/hadoop/run/ha/jn/hacluster/current/VERSION
/home/hadoop/run/ha/jn/hacluster/current/paxos
/home/hadoop/run/ha/jn/hacluster/in_use.lock
--------- h104---------
/home/hadoop/run
```

查看集群id ：clusterID （一台机器上看就可以，每台jn上都有一样的集群id）

```
[hadoop@h101 /]$ cexec "cat /home/hadoop/run/ha/jn/hacluster/current/VERSION"
************************* h *************************
--------- h101---------
#Wed Jan 09 20:25:58 CST 2019
namespaceID=1703055857
clusterID=CID-4d978406-f243-49d3-b941-5a2bbb849d28
cTime=0
storageType=JOURNAL_NODE
layoutVersion=-63
--------- h102---------
#Wed Jan 09 20:25:58 CST 2019
namespaceID=1703055857
clusterID=CID-4d978406-f243-49d3-b941-5a2bbb849d28
cTime=0
storageType=JOURNAL_NODE
layoutVersion=-63
--------- h103---------
#Wed Jan 09 20:25:58 CST 2019
namespaceID=1703055857
clusterID=CID-4d978406-f243-49d3-b941-5a2bbb849d28
cTime=0
storageType=JOURNAL_NODE
layoutVersion=-63
--------- h104---------
cat: /home/hadoop/run/ha/jn/hacluster/current/VERSION: 没有那个文件或目录
```



启动nn1节点（h101）的namenode

```
[hadoop@h101 /]$  hadoop-daemon.sh  start namenode
starting namenode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-namenode-h101.out
```

在nn2上同步nn1的元信息

```
[hadoop@h102 ~]$ hdfs namenode -bootstrapStandby
19/01/09 20:31:24 INFO namenode.NameNode: STARTUP_MSG:
=====================================================
About to bootstrap Standby ID nn2 from:
           Nameservice ID: hacluster
        Other Namenode ID: nn1
  Other NN's HTTP address: http://h101:50070
  Other NN's IPC  address: h101/192.168.56.101:8020
             Namespace ID: 1703055857
            Block pool ID: BP-913785061-192.168.56.101-1547036757957
               Cluster ID: CID-4d978406-f243-49d3-b941-5a2bbb849d28
           Layout version: -63
       isUpgradeFinalized: true
=====================================================
19/01/09 20:31:27 INFO common.Storage: Storage directory /home/hadoop/run/dfs/name has been successfully formatted.
19/01/09 20:31:28 INFO namenode.TransferFsImage: Opening connection to http://h101:50070/imagetransfer?getimage=1&txid=0&storageInfo=-63:1703055857:0:CID-4d978406-f243-49d3-b941-5a2bbb849d28
19/01/09 20:31:28 INFO namenode.TransferFsImage: Image Transfer timeout configured to 60000 milliseconds
19/01/09 20:31:28 INFO namenode.TransferFsImage: Transfer took 0.00s at 0.00 KB/s
19/01/09 20:31:28 INFO namenode.TransferFsImage: Downloaded file fsimage.ckpt_0000000000000000000 size 323 bytes.
19/01/09 20:31:28 INFO util.ExitUtil: Exiting with status 0
```



启动nn2

```
[hadoop@h102 ~]$ hadoop-daemon.sh start namenode
starting namenode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-namenode-h102.out
```



在nn1上启动所有的datanode



```
[hadoop@h101 /]$ hadoop-daemons.sh start datanode
h102: starting datanode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-datanode-h102.out
h104: starting datanode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-datanode-h104.out
h103: starting datanode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-datanode-h103.out
```



目前所有机器上的进程：

```
[hadoop@h101 /]$ cexec /usr/local/java7/bin/jps
************************* h *************************
--------- h101---------
2551 NameNode
2705 Jps
2379 JournalNode
--------- h102---------
2167 JournalNode
1550 QuorumPeerMain
2424 DataNode
2513 Jps
2317 NameNode
--------- h103---------
1407 QuorumPeerMain
2220 Jps
2141 DataNode
2009 JournalNode
--------- h104---------
2027 Jps
1948 DataNode
1394 QuorumPeerMain
```

访问网页

http://h101:50070 或者

http://h102:50070/



两个Namenode都是standby（备用）的状态



手动切换状态，在各个NameNode节点上启动DFSZK Failover Controller，先在哪台机器启动，哪个机器的NameNode就是Active NameNode

```
[hadoop@h101 sbin]$ hadoop-daemon.sh start zkfc
starting zkfc, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-zkfc-h101.out

[hadoop@h102 ~]$ hadoop-daemon.sh start zkfc
starting zkfc, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-zkfc-h102.out
```

执行以上两个命令之后，依然是standby, 先启动的机器并没有变为active

执行以下命令后，nn1即h101 变为了active h102依然为standby

```
[hadoop@h101 sbin]$ hdfs haadmin -transitionToActive nn1 --forcemanual 
You have specified the --forcemanual flag. This flag is dangerous, as it can induce a split-brain scenario that WILL CORRUPT your HDFS namespace, possibly irrecoverably.

It is recommended not to use this flag, but instead to shut down the cluster and disable automatic failover if you prefer to manually manage your HA state.

You may abort safely by answering 'n' or hitting ^C now.

Are you sure you want to continue? (Y or N) Y
19/01/09 20:43:21 WARN ha.HAAdmin: Proceeding with manual HA state management even though
automatic failover is enabled for NameNode at h102/192.168.56.102:8020
19/01/09 20:43:23 WARN ha.HAAdmin: Proceeding with manual HA state management even though
automatic failover is enabled for NameNode at h101/192.168.56.101:8020
```

此时如果要再设置nn2为active就不行了

提示：Node nn1 is already active

```
[hadoop@h102 ~]$ hdfs haadmin -transitionToActive nn2 --forcemanual
You have specified the --forcemanual flag. This flag is dangerous, as it can induce a split-brain scenario that WILL CORRUPT your HDFS namespace, possibly irrecoverably.

It is recommended not to use this flag, but instead to shut down the cluster and disable automatic failover if you prefer to manually manage your HA state.

You may abort safely by answering 'n' or hitting ^C now.

Are you sure you want to continue? (Y or N) Y
19/01/09 20:46:05 WARN ha.HAAdmin: Proceeding with manual HA state management even though
automatic failover is enabled for NameNode at h101/192.168.56.101:8020
transitionToActive: Node nn1 is already active
Usage: haadmin [-transitionToActive [--forceactive] <serviceId>]
```

到这里，手动故障转移已经完成，接下来配置自动故障转移



### 自动故障转移

先把整个集群关闭，zookeeper不关，输入bin/hdfs zkfc –formatZK，格式化ZKFC



h102上观察zookeeper

```
[zk: localhost:2181(CONNECTED) 0] ls
[zk: localhost:2181(CONNECTED) 1] ls /
[project, zookeeper]
```

目前还没有ha相关的信息

执行formatZK之后

```
[hadoop@h101 ~]$ hdfs zkfc -formatZK
```

多了hadoop-ha了

```
[zk: localhost:2181(CONNECTED) 2] ls /
[project, hadoop-ha, zookeeper]
[zk: localhost:2181(CONNECTED) 3] get /hadoop-ha

cZxid = 0x100000014
ctime = Wed Jan 09 20:48:13 CST 2019
mZxid = 0x100000014
mtime = Wed Jan 09 20:48:13 CST 2019
pZxid = 0x100000015
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 1
[zk: localhost:2181(CONNECTED) 4] ls /hadoop-ha
[hacluster]

[zk: localhost:2181(CONNECTED) 5] get /hadoop-ha/hacluster

cZxid = 0x100000015
ctime = Wed Jan 09 20:48:13 CST 2019
mZxid = 0x100000015
mtime = Wed Jan 09 20:48:13 CST 2019
pZxid = 0x100000015
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 0

[zk: localhost:2181(CONNECTED) 6] ls /hadoop-ha/hacluster 
[]
```

启动集群

```
[hadoop@h101 sbin]$ start-dfs.sh 
Starting namenodes on [h101 h102]
h101: starting namenode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-namenode-h101.out
h102: starting namenode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-namenode-h102.out
h103: starting datanode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-datanode-h103.out
h104: starting datanode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-datanode-h104.out
h102: starting datanode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-datanode-h102.out
Starting journal nodes [h101 h102 h103]
h103: starting journalnode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-journalnode-h103.out
h102: starting journalnode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-journalnode-h102.out
h101: starting journalnode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-journalnode-h101.out
Starting ZK Failover Controllers on NN hosts [h101 h102]
h101: starting zkfc, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-zkfc-h101.out
h102: starting zkfc, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-zkfc-h102.out
```



启动后打开页面查看nn1为active，nn2为standby



### 验证ha

https://www.cnblogs.com/share23/p/9708473.html



kill掉nn1的NameNode

```
[hadoop@h101 sbin]$ jps
3831 JournalNode
3623 NameNode
4008 DFSZKFailoverController
4078 Jps
[hadoop@h101 sbin]$ kill 3623
[hadoop@h101 sbin]$ kill 3623
-bash: kill: (3623) - 没有那个进程
[hadoop@h101 sbin]$ jps
3831 JournalNode
4105 Jps
4008 DFSZKFailoverController
```

http://h101:50070/  访问不了了

http://h102:50070/  还是standby



```
[zk: localhost:2181(CONNECTED) 10] ls /hadoop-ha/hacluster
[ActiveBreadCrumb]

[zk: localhost:2181(CONNECTED) 9] ls /hadoop-ha
[hacluster]
[zk: localhost:2181(CONNECTED) 10] ls /hadoop-ha/hacluster
[ActiveBreadCrumb]
[zk: localhost:2181(CONNECTED) 11] ls /hadoop-ha/hacluster/ActiveBreadCrumb
[]
[zk: localhost:2181(CONNECTED) 12] get /hadoop-ha/hacluster/ActiveBreadCrumb

        haclusternn1h101 �>(�>
cZxid = 0x100000019
ctime = Wed Jan 09 21:14:16 CST 2019
mZxid = 0x100000019
mtime = Wed Jan 09 21:14:16 CST 2019
pZxid = 0x100000019
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 28
numChildren = 0
```

配置正确的话，h102应该变为active了吧？



参考：

[ha切换不成功原因](ha切换不成功原因.md)

```
[hadoop@h102 logs]$ pwd
/home/hadoop/hadoop-2.7.7/logs
```

报错日志：

> 2019-01-09 21:16:08,290 WARN org.apache.hadoop.ha.SshFenceByTcpPort: PATH=$PATH:/sbin:/usr/sbin **fuser** -v -k -n tcp 8020 via ssh: bash: **fuse**
> **r: 未找到命令**

```
2019-01-09 21:16:08,046 INFO org.apache.hadoop.ha.SshFenceByTcpPort: Looking for process running on port 8020
2019-01-09 21:16:08,290 WARN org.apache.hadoop.ha.SshFenceByTcpPort: PATH=$PATH:/sbin:/usr/sbin fuser -v -k -n tcp 8020 via ssh: bash: fuse
r: 未找到命令
2019-01-09 21:16:08,290 INFO org.apache.hadoop.ha.SshFenceByTcpPort: rc: 127
2019-01-09 21:16:08,290 INFO org.apache.hadoop.ha.SshFenceByTcpPort.jsch: Disconnecting from h101 port 22
2019-01-09 21:16:08,295 WARN org.apache.hadoop.ha.NodeFencer: Fencing method org.apache.hadoop.ha.SshFenceByTcpPort(null) was unsuccessful.
2019-01-09 21:16:08,295 ERROR org.apache.hadoop.ha.NodeFencer: Unable to fence service by any configured method.
2019-01-09 21:16:08,296 WARN org.apache.hadoop.ha.ActiveStandbyElector: Exception handling the winning of election
java.lang.RuntimeException: Unable to fence NameNode at h101/192.168.56.101:8020
        at org.apache.hadoop.ha.ZKFailoverController.doFence(ZKFailoverController.java:533)
        at org.apache.hadoop.ha.ZKFailoverController.fenceOldActive(ZKFailoverController.java:505)
        at org.apache.hadoop.ha.ZKFailoverController.access$1100(ZKFailoverController.java:61)
        at org.apache.hadoop.ha.ZKFailoverController$ElectorCallbacks.fenceOldActive(ZKFailoverController.java:892)
        at org.apache.hadoop.ha.ActiveStandbyElector.fenceOldActive(ActiveStandbyElector.java:921)
        at org.apache.hadoop.ha.ActiveStandbyElector.becomeActive(ActiveStandbyElector.java:820)
        at org.apache.hadoop.ha.ActiveStandbyElector.processResult(ActiveStandbyElector.java:418)
        at org.apache.zookeeper.ClientCnxn$EventThread.processEvent(ClientCnxn.java:599)
        at org.apache.zookeeper.ClientCnxn$EventThread.run(ClientCnxn.java:498)
2019-01-09 21:16:08,297 INFO org.apache.hadoop.ha.ActiveStandbyElector: Trying to re-establish ZK session
```



```
在zkfc的日志里面，有一个warn：PATH=$PATH:/sbin:/usr/sbin fuser -v -k -n tcp 8090 via ssh: bash: fuser: 未找到命令
原因是最小化安装centos的时候，没有fuser这个命令，导致无法fence
yum install psmisc就行了
```



解决

```
[root@h101 ~]# cexec mount /dev/cdrom /media/cdrom
************************* h *************************
--------- h101---------
mount: /dev/sr0 写保护，将以只读方式挂载
--------- h102---------
mount: /dev/sr0 写保护，将以只读方式挂载
--------- h103---------
mount: 在 /dev/sr0 上找不到媒体
--------- h104---------
mount: /dev/sr0 写保护，将以只读方式挂载
[root@h101 ~]# cexec mount /dev/cdrom /media/cdrom
************************* h *************************
--------- h101---------
mount: /dev/sr0 写保护，将以只读方式挂载
mount: /dev/sr0 已经挂载或 /media/cdrom 忙
       /dev/sr0 已经挂载到 /media/cdrom 上
--------- h102---------
mount: /dev/sr0 写保护，将以只读方式挂载
mount: /dev/sr0 已经挂载或 /media/cdrom 忙
       /dev/sr0 已经挂载到 /media/cdrom 上
--------- h103---------
mount: /dev/sr0 写保护，将以只读方式挂载
--------- h104---------
mount: /dev/sr0 写保护，将以只读方式挂载
mount: /dev/sr0 已经挂载或 /media/cdrom 忙
       /dev/sr0 已经挂载到 /media/cdrom 上
[root@h101 ~]# cexec yum install psmisc -y
************************* h *************************
--------- h101---------
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * c7-media: 
正在解决依赖关系
--> 正在检查事务
---> 软件包 psmisc.x86_64.0.22.20-15.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

================================================================================
 Package         架构            版本                   源                 大小
================================================================================
正在安装:
 psmisc          x86_64          22.20-15.el7           c7-media          141 k

事务概要
================================================================================
安装  1 软件包

总下载量：141 k
安装大小：475 k
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : psmisc-22.20-15.el7.x86_64                                  1/1 
  验证中      : psmisc-22.20-15.el7.x86_64                                  1/1 

已安装:
  psmisc.x86_64 0:22.20-15.el7                                                  

完毕！
--------- h102---------
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * c7-media: 
正在解决依赖关系
--> 正在检查事务
---> 软件包 psmisc.x86_64.0.22.20-15.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

================================================================================
 Package         架构            版本                   源                 大小
================================================================================
正在安装:
 psmisc          x86_64          22.20-15.el7           c7-media          141 k

事务概要
================================================================================
安装  1 软件包

总下载量：141 k
安装大小：475 k
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : psmisc-22.20-15.el7.x86_64                                  1/1 
  验证中      : psmisc-22.20-15.el7.x86_64                                  1/1 

已安装:
  psmisc.x86_64 0:22.20-15.el7                                                  

完毕！
--------- h103---------
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * c7-media: 
正在解决依赖关系
--> 正在检查事务
---> 软件包 psmisc.x86_64.0.22.20-15.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

================================================================================
 Package         架构            版本                   源                 大小
================================================================================
正在安装:
 psmisc          x86_64          22.20-15.el7           c7-media          141 k

事务概要
================================================================================
安装  1 软件包

总下载量：141 k
安装大小：475 k
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : psmisc-22.20-15.el7.x86_64                                  1/1 
  验证中      : psmisc-22.20-15.el7.x86_64                                  1/1 

已安装:
  psmisc.x86_64 0:22.20-15.el7                                                  

完毕！
--------- h104---------
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * c7-media: 
正在解决依赖关系
--> 正在检查事务
---> 软件包 psmisc.x86_64.0.22.20-15.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

================================================================================
 Package         架构            版本                   源                 大小
================================================================================
正在安装:
 psmisc          x86_64          22.20-15.el7           c7-media          141 k

事务概要
================================================================================
安装  1 软件包

总下载量：141 k
安装大小：475 k
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : psmisc-22.20-15.el7.x86_64                                  1/1 
  验证中      : psmisc-22.20-15.el7.x86_64                                  1/1 

已安装:
  psmisc.x86_64 0:22.20-15.el7                                                  

完毕！
[root@h101 ~]# cexec which fuser
************************* h *************************
--------- h101---------
/usr/sbin/fuser
--------- h102---------
/usr/sbin/fuser
--------- h103---------
/usr/sbin/fuser
--------- h104---------
/usr/sbin/fuser
```



处理完之后，就可以自动切换了。

启动后nn1为active，nn2为standby

kill掉nn1的namenode进程后nn2的namenode变为active



日志中还有一个报错，虽然这里没有影响ha切换，但是还是要解决掉好，肯定其他地方有用到了

2019-01-09 21:37:28,476 **WARN** org.apache.hadoop.ha.SshFenceByTcpPort: nc -z h101 8020 via ssh: bash: nc: 未找到命令

```
[root@h101 ~]# cexec yum install nc -y
************************* h *************************
--------- h101---------
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * c7-media: 
软件包 2:nmap-ncat-6.40-13.el7.x86_64 已安装并且是最新版本
无须任何处理
--------- h102---------
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * c7-media: 
正在解决依赖关系
--> 正在检查事务
---> 软件包 nmap-ncat.x86_64.2.6.40-13.el7 将被 安装
--> 正在处理依赖关系 libpcap.so.1()(64bit)，它被软件包 2:nmap-ncat-6.40-13.el7.x86_64 需要
--> 正在检查事务
---> 软件包 libpcap.x86_64.14.1.5.3-11.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

================================================================================
 Package          架构          版本                      源               大小
================================================================================
正在安装:
 nmap-ncat        x86_64        2:6.40-13.el7             c7-media        205 k
为依赖而安装:
 libpcap          x86_64        14:1.5.3-11.el7           c7-media        138 k

事务概要
================================================================================
安装  1 软件包 (+1 依赖软件包)

总下载量：343 k
安装大小：740 k
Downloading packages:
--------------------------------------------------------------------------------
总计                                                21 MB/s | 343 kB  00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : 14:libpcap-1.5.3-11.el7.x86_64                              1/2 
  正在安装    : 2:nmap-ncat-6.40-13.el7.x86_64                              2/2 
  验证中      : 14:libpcap-1.5.3-11.el7.x86_64                              1/2 
  验证中      : 2:nmap-ncat-6.40-13.el7.x86_64                              2/2 

已安装:
  nmap-ncat.x86_64 2:6.40-13.el7                                                

作为依赖被安装:
  libpcap.x86_64 14:1.5.3-11.el7                                                

完毕！
--------- h103---------
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * c7-media: 
正在解决依赖关系
--> 正在检查事务
---> 软件包 nmap-ncat.x86_64.2.6.40-13.el7 将被 安装
--> 正在处理依赖关系 libpcap.so.1()(64bit)，它被软件包 2:nmap-ncat-6.40-13.el7.x86_64 需要
--> 正在检查事务
---> 软件包 libpcap.x86_64.14.1.5.3-11.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

================================================================================
 Package          架构          版本                      源               大小
================================================================================
正在安装:
 nmap-ncat        x86_64        2:6.40-13.el7             c7-media        205 k
为依赖而安装:
 libpcap          x86_64        14:1.5.3-11.el7           c7-media        138 k

事务概要
================================================================================
安装  1 软件包 (+1 依赖软件包)

总下载量：343 k
安装大小：740 k
Downloading packages:
--------------------------------------------------------------------------------
总计                                                25 MB/s | 343 kB  00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : 14:libpcap-1.5.3-11.el7.x86_64                              1/2 
  正在安装    : 2:nmap-ncat-6.40-13.el7.x86_64                              2/2 
  验证中      : 14:libpcap-1.5.3-11.el7.x86_64                              1/2 
  验证中      : 2:nmap-ncat-6.40-13.el7.x86_64                              2/2 

已安装:
  nmap-ncat.x86_64 2:6.40-13.el7                                                

作为依赖被安装:
  libpcap.x86_64 14:1.5.3-11.el7                                                

完毕！
--------- h104---------
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * c7-media: 
正在解决依赖关系
--> 正在检查事务
---> 软件包 nmap-ncat.x86_64.2.6.40-13.el7 将被 安装
--> 正在处理依赖关系 libpcap.so.1()(64bit)，它被软件包 2:nmap-ncat-6.40-13.el7.x86_64 需要
--> 正在检查事务
---> 软件包 libpcap.x86_64.14.1.5.3-11.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

================================================================================
 Package          架构          版本                      源               大小
================================================================================
正在安装:
 nmap-ncat        x86_64        2:6.40-13.el7             c7-media        205 k
为依赖而安装:
 libpcap          x86_64        14:1.5.3-11.el7           c7-media        138 k

事务概要
================================================================================
安装  1 软件包 (+1 依赖软件包)

总下载量：343 k
安装大小：740 k
Downloading packages:
--------------------------------------------------------------------------------
总计                                                22 MB/s | 343 kB  00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : 14:libpcap-1.5.3-11.el7.x86_64                              1/2 
  正在安装    : 2:nmap-ncat-6.40-13.el7.x86_64                              2/2 
  验证中      : 14:libpcap-1.5.3-11.el7.x86_64                              1/2 
  验证中      : 2:nmap-ncat-6.40-13.el7.x86_64                              2/2 

已安装:
  nmap-ncat.x86_64 2:6.40-13.el7                                                

作为依赖被安装:
  libpcap.x86_64 14:1.5.3-11.el7                                                

完毕！


[root@h101 ~]# cexec which nc
************************* h *************************
--------- h101---------
/usr/bin/nc
--------- h102---------
/usr/bin/nc
--------- h103---------
/usr/bin/nc
--------- h104---------
/usr/bin/nc
```



重新启动h101的namenode

```
cd /home/hadoop/hadoop-2.7.7/sbin
[hadoop@h101 sbin]$ ./hadoop-daemon.sh start namenode
starting namenode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-namenode-h101.out
```

这时候h102还是active, 新启动的h101变为standby了





### yarn高可用

```
[hadoop@h101 sbin]$ start-yarn.sh
starting yarn daemons
starting resourcemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-resourcemanager-h101.out
h103: starting nodemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-nodemanager-h103.out
h102: starting nodemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-nodemanager-h102.out
h104: starting nodemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-nodemanager-h104.out
[hadoop@h101 sbin]$ 
[hadoop@h101 sbin]$ 
[hadoop@h101 sbin]$ cexec /usr/local/java7/bin/jps
************************* h *************************
--------- h101---------
9892 NameNode
10733 Jps
9458 DFSZKFailoverController
9282 JournalNode
--------- h102---------
1550 QuorumPeerMain
6709 NodeManager
6070 DFSZKFailoverController
5947 JournalNode
5848 DataNode
5775 NameNode
6816 Jps
--------- h103---------
3353 NodeManager
2820 DataNode
1407 QuorumPeerMain
3454 Jps
2916 JournalNode
--------- h104---------
3037 Jps
2936 NodeManager
2485 DataNode
1394 QuorumPeerMain
```



发现resourcemanager没有启动成功

查看日志：

```
[hadoop@h101 logs]$ pwd
/home/hadoop/hadoop-2.7.7/logs
[hadoop@h101 logs]$ more yarn-hadoop-resourcemanager-h101.log
```

报错：

```
2019-01-09 22:29:35,927 INFO org.apache.hadoop.service.AbstractService: Service ResourceManager failed in state INITED; cause: org.apache.h
adoop.yarn.exceptions.YarnRuntimeException: Invalid configuration! Can not find valid RM_HA_ID. None of yarn.resourcemanager.address.rm1 ya
rn.resourcemanager.address.rm2  are matching the local address OR yarn.resourcemanager.ha.id is not specified in HA Configuration
org.apache.hadoop.yarn.exceptions.YarnRuntimeException: Invalid configuration! Can not find valid RM_HA_ID. None of yarn.resourcemanager.ad
dress.rm1 yarn.resourcemanager.address.rm2  are matching the local address OR yarn.resourcemanager.ha.id is not specified in HA Configurati
on
        at org.apache.hadoop.yarn.conf.HAUtil.throwBadConfigurationException(HAUtil.java:43)
        at org.apache.hadoop.yarn.conf.HAUtil.verifyAndSetCurrentRMHAId(HAUtil.java:125)
        at org.apache.hadoop.yarn.conf.HAUtil.verifyAndSetConfiguration(HAUtil.java:81)
        at org.apache.hadoop.yarn.server.resourcemanager.ResourceManager.serviceInit(ResourceManager.java:228)
        at org.apache.hadoop.service.AbstractService.init(AbstractService.java:163)
        at org.apache.hadoop.yarn.server.resourcemanager.ResourceManager.main(ResourceManager.java:1187)
2019-01-09 22:29:35,933 INFO org.apache.hadoop.yarn.server.resourcemanager.ResourceManager: Transitioning to standby state
```



因为yarn resourcemanager是配置在h102和h103的，所以不能再h101上启动yarn，要在h102和h103上启动

```
    <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm1,rm2</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>h102</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname.rm2</name>
        <value>h103</value>
    </property>
```

[hadoop@h102 sbin]$ start-yarn.sh

```
[hadoop@h102 sbin]$ start-yarn.sh
starting yarn daemons
starting resourcemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-resourcemanager-h102.out
h103: starting nodemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-nodemanager-h103.out
h104: starting nodemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-nodemanager-h104.out
h102: starting nodemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-nodemanager-h102.out
```



验证进程

```
[hadoop@h101 hadoop]$ cexec /usr/local/java7/bin/jps
************************* h *************************
--------- h101---------
9892 NameNode
9458 DFSZKFailoverController
10977 Jps
9282 JournalNode
--------- h102---------
7103 ResourceManager
1550 QuorumPeerMain
6070 DFSZKFailoverController
7214 NodeManager
5947 JournalNode
5848 DataNode
7577 Jps
5775 NameNode
--------- h103---------
2820 DataNode
1407 QuorumPeerMain
2916 JournalNode
3758 Jps
3604 NodeManager
--------- h104---------
3313 Jps
3187 NodeManager
2485 DataNode
1394 QuorumPeerMain
```

只有h102上有ResourceManager，怎么体现高可用呢？

zookeeper到时多了 yarn-leader-election

```
[zk: localhost:2181(CONNECTED) 0] ls /
[project, rmstore, yarn-leader-election, hadoop-ha, zookeeper]
```

访问

http://h102:8088/

点击about

http://h102:8088/cluster/cluster

```
Cluster ID:	1547044705991
ResourceManager state:	STARTED
ResourceManager HA state:	active
ResourceManager HA zookeeper connection state:	CONNECTED
ResourceManager RMStateStore:	org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore
ResourceManager started on:	星期三 一月 09 22:38:25 +0800 2019
ResourceManager version:	2.7.7 from c1aad84bd27cd79c3d1a7dd58202a8c3ee1ed3ac by stevel source checksum d0c780b3552e7bd9462fffca3f9fc51d on 2018-07-19T00:39Z
Hadoop version:	2.7.7 from c1aad84bd27cd79c3d1a7dd58202a8c3ee1ed3ac by stevel source checksum 792e15d20b12c74bd6f19a1fb886490 on 2018-07-18T22:47Z
```

页面上有ha和cluster的信息, h102状态为active

参考：

https://blog.csdn.net/qq_33161208/article/details/80925232

另外一个节点h103需要手动启动

```
[hadoop@h103 ~]$ yarn-daemon.sh start resourcemanager
starting resourcemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-resourcemanager-h103.out
[hadoop@h103 ~]$ 
```

启动后访问h103, 状态为standby

http://h103:8088/cluster/cluster

```
Cluster ID:	1547045222504
ResourceManager state:	STARTED
ResourceManager HA state:	standby
ResourceManager HA zookeeper connection state:	CONNECTED
ResourceManager RMStateStore:	org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore
ResourceManager started on:	星期三 一月 09 22:47:02 +0800 2019
ResourceManager version:	2.7.7 from c1aad84bd27cd79c3d1a7dd58202a8c3ee1ed3ac by stevel source checksum d0c780b3552e7bd9462fffca3f9fc51d on 2018-07-19T00:39Z
Hadoop version:	2.7.7 from c1aad84bd27cd79c3d1a7dd58202a8c3ee1ed3ac by stevel source checksum 792e15d20b12c74bd6f19a1fb886490 on 2018-07-18T22:47Z
```

停止h102的resourcemanager（kill -9 ）

```
[hadoop@h102 hadoop]$ jps
7653 Jps
7103 ResourceManager
1550 QuorumPeerMain
6070 DFSZKFailoverController
7214 NodeManager
5947 JournalNode
5848 DataNode
5775 NameNode
[hadoop@h102 hadoop]$ kill -9 7103
```



h103的resourcemanager就变为active了

```
http://h103:8088/cluster/cluster
Cluster ID:	1547045347557
ResourceManager state:	STARTED
ResourceManager HA state:	active
ResourceManager HA zookeeper connection state:	CONNECTED
ResourceManager RMStateStore:	org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore
ResourceManager started on:	星期三 一月 09 22:49:07 +0800 2019
ResourceManager version:	2.7.7 from c1aad84bd27cd79c3d1a7dd58202a8c3ee1ed3ac by stevel source checksum d0c780b3552e7bd9462fffca3f9fc51d on 2018-07-19T00:39Z
Hadoop version:	2.7.7 from c1aad84bd27cd79c3d1a7dd58202a8c3ee1ed3ac by stevel source checksum 792e15d20b12c74bd6f19a1fb886490 on 2018-07-18T22:47Z
```

再启动h102的rm

```
[hadoop@h102 hadoop]$ yarn-daemon.sh start resourcemanager
starting resourcemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-resourcemanager-h102.out
```

h102变standby了

```
Cluster ID:	1547045407119
ResourceManager state:	STARTED
ResourceManager HA state:	standby
ResourceManager HA zookeeper connection state:	CONNECTED
ResourceManager RMStateStore:	org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore
ResourceManager started on:	星期三 一月 09 22:50:07 +0800 2019
ResourceManager version:	2.7.7 from c1aad84bd27cd79c3d1a7dd58202a8c3ee1ed3ac by stevel source checksum d0c780b3552e7bd9462fffca3f9fc51d on 2018-07-19T00:39Z
Hadoop version:	2.7.7 from c1aad84bd27cd79c3d1a7dd58202a8c3ee1ed3ac by stevel source checksum 792e15d20b12c74bd6f19a1fb886490 on 2018-07-18T22:47Z
```

也可以命令查看状态

```
[hadoop@h102 hadoop]$ yarn rmadmin -getServiceState rm1
standby
[hadoop@h102 hadoop]$ yarn rmadmin -getServiceState rm2
active
```

https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/ResourceManagerHA.html



### 关闭

#### 关yarn

```
[hadoop@h102 ~]$ stop-yarn.sh
stopping yarn daemons
stopping resourcemanager
h103: stopping nodemanager
h102: stopping nodemanager
h104: stopping nodemanager
no proxyserver to stop
```



h103也要单独关

```
[hadoop@h103 sbin]$ ./yarn-daemon.sh  stop resourcemanager
```



#### 关hdfs

```
[hadoop@h101 ~]$ stop-dfs.sh
Stopping namenodes on [h101 h102]
h101: stopping namenode
h102: stopping namenode
h104: stopping datanode
h103: stopping datanode
h102: stopping datanode
Stopping journal nodes [h101 h102 h103]
h101: stopping journalnode
h103: stopping journalnode
h102: stopping journalnode
Stopping ZK Failover Controllers on NN hosts [h101 h102]
h101: stopping zkfc
h102: stopping zkfc
```



#### 关zookeeper

```
[hadoop@h102 ~]$ zkServer.sh stop
ZooKeeper JMX enabled by default
Using config: /home/hadoop/zookeeper-3.4.12/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED

[hadoop@h104 ~]$ zkServer.sh stop
ZooKeeper JMX enabled by default
Using config: /home/hadoop/zookeeper-3.4.12/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED

[hadoop@h103 ~]$ zkServer.sh stop
ZooKeeper JMX enabled by default
Using config: /home/hadoop/zookeeper-3.4.12/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
```



#### 都关闭检查进程

```
[hadoop@h101 ~]$ cexec /usr/local/java7/bin/jps
************************* h *************************
--------- h101---------
11650 Jps
--------- h102---------
8224 Jps
--------- h103---------
4412 Jps
--------- h104---------
3561 Jps
```



### 启动

#### 启动zookeeper

```
[hadoop@h102 ~]$ zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /home/hadoop/zookeeper-3.4.12/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED

[hadoop@h103 ~]$ zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /home/hadoop/zookeeper-3.4.12/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED

[hadoop@h104 ~]$ zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /home/hadoop/zookeeper-3.4.12/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED

```

#### 启动hdfs

```
[hadoop@h101 ~]$ start-dfs.sh
Starting namenodes on [h101 h102]
h101: starting namenode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-namenode-h101.out
h102: starting namenode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-namenode-h102.out
h103: starting datanode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-datanode-h103.out
h104: starting datanode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-datanode-h104.out
h102: starting datanode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-datanode-h102.out
Starting journal nodes [h101 h102 h103]
h101: starting journalnode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-journalnode-h101.out
h103: starting journalnode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-journalnode-h103.out
h102: starting journalnode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-journalnode-h102.out
Starting ZK Failover Controllers on NN hosts [h101 h102]
h101: starting zkfc, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-zkfc-h101.out
h102: starting zkfc, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-zkfc-h102.out
```

#### 启动yarn

```
[hadoop@h102 ~]$ start-yarn.sh
starting yarn daemons
starting resourcemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-resourcemanager-h102.out
h103: starting nodemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-nodemanager-h103.out
h104: starting nodemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-nodemanager-h104.out
h102: starting nodemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-nodemanager-h102.out
```

h103上的yarn单独启动

```
cd ~/hadoop-2.7.7/sbin/
[hadoop@h103 ~]$ ./yarn-daemon.sh start resourcemanager
```



#### 启动后进程

```
[hadoop@h101 ~]$ cexec /usr/local/java7/bin/jps
************************* h *************************
--------- h101---------
12167 DFSZKFailoverController
11989 JournalNode
12329 Jps
11779 NameNode
--------- h102---------
8725 ResourceManager
8332 NameNode
8406 DataNode
8837 NodeManager
8624 DFSZKFailoverController
8505 JournalNode
8251 QuorumPeerMain
9080 Jps
--------- h103---------
4438 QuorumPeerMain
5283 Jps
5007 NodeManager
4600 JournalNode
4503 DataNode
4703 ResourceManager
--------- h104---------
3646 DataNode
3764 NodeManager
3951 Jps
3587 QuorumPeerMain
```



### cexec环境变量问题

```
[hadoop@h101 ~]$ cexec env
************************* h *************************
--------- h101---------
XDG_SESSION_ID=371
SELINUX_ROLE_REQUESTED=
SHELL=/bin/bash
SSH_CLIENT=192.168.56.101 41054 22
SELINUX_USE_CURRENT_RANGE=
USER=hadoop
MAIL=/var/mail/hadoop
PATH=/usr/local/bin:/usr/bin
PWD=/home/hadoop
LANG=zh_CN.UTF-8
SELINUX_LEVEL_REQUESTED=
SHLVL=1
HOME=/home/hadoop
LOGNAME=hadoop
SSH_CONNECTION=192.168.56.101 41054 192.168.56.101 22
LESSOPEN=||/usr/bin/lesspipe.sh %s
XDG_RUNTIME_DIR=/run/user/1001
_=/usr/bin/env
--------- h102---------
XDG_SESSION_ID=70
SELINUX_ROLE_REQUESTED=
SHELL=/bin/bash
SSH_CLIENT=192.168.56.101 47504 22
SELINUX_USE_CURRENT_RANGE=
USER=hadoop
MAIL=/var/mail/hadoop
PATH=/usr/local/bin:/usr/bin
PWD=/home/hadoop
LANG=zh_CN.UTF-8
SELINUX_LEVEL_REQUESTED=
SHLVL=1
HOME=/home/hadoop
LOGNAME=hadoop
SSH_CONNECTION=192.168.56.101 47504 192.168.56.102 22
LESSOPEN=||/usr/bin/lesspipe.sh %s
XDG_RUNTIME_DIR=/run/user/1001
_=/usr/bin/env
--------- h103---------
XDG_SESSION_ID=57
SELINUX_ROLE_REQUESTED=
SHELL=/bin/bash
SSH_CLIENT=192.168.56.101 36882 22
SELINUX_USE_CURRENT_RANGE=
USER=hadoop
MAIL=/var/mail/hadoop
PATH=/usr/local/bin:/usr/bin
PWD=/home/hadoop
LANG=zh_CN.UTF-8
SELINUX_LEVEL_REQUESTED=
SHLVL=1
HOME=/home/hadoop
LOGNAME=hadoop
SSH_CONNECTION=192.168.56.101 36882 192.168.56.103 22
LESSOPEN=||/usr/bin/lesspipe.sh %s
XDG_RUNTIME_DIR=/run/user/1000
_=/usr/bin/env
--------- h104---------
XDG_SESSION_ID=51
SELINUX_ROLE_REQUESTED=
SHELL=/bin/bash
SSH_CLIENT=192.168.56.101 54978 22
SELINUX_USE_CURRENT_RANGE=
USER=hadoop
MAIL=/var/mail/hadoop
PATH=/usr/local/bin:/usr/bin
PWD=/home/hadoop
LANG=zh_CN.UTF-8
SELINUX_LEVEL_REQUESTED=
SHLVL=1
HOME=/home/hadoop
LOGNAME=hadoop
SSH_CONNECTION=192.168.56.101 54978 192.168.56.104 22
LESSOPEN=||/usr/bin/lesspipe.sh %s
XDG_RUNTIME_DIR=/run/user/1000
_=/usr/bin/env
```

这里PATH都只有/usr/local/bin:/usr/bin



这是因为ssh的环境变量就是这样

```
[hadoop@h101 ~]$ ssh h102 "env"
XDG_SESSION_ID=76
SELINUX_ROLE_REQUESTED=
SHELL=/bin/bash
SSH_CLIENT=192.168.56.101 47558 22
SELINUX_USE_CURRENT_RANGE=
USER=hadoop
MAIL=/var/mail/hadoop
PATH=/usr/local/bin:/usr/bin
PWD=/home/hadoop
LANG=zh_CN.UTF-8
SELINUX_LEVEL_REQUESTED=
SHLVL=1
HOME=/home/hadoop
LOGNAME=hadoop
SSH_CONNECTION=192.168.56.101 47558 192.168.56.102 22
LESSOPEN=||/usr/bin/lesspipe.sh %s
XDG_RUNTIME_DIR=/run/user/1001
_=/usr/bin/env
```



参考

 [解决SSH远程执行命令找不到环境变量的问题](https://www.cnblogs.com/zhenyuyaodidiao/p/9287497.html)

```
1. 通过SSH登录后再执行命令和脚本
这种方式会使用Bash的interactive + login shell模式，这里面有两个概念需要解释：interactive和login。

login故名思义，即登陆，login shell是指用户以非图形化界面或者以ssh登陆到机器上时获得的第一个shell，简单些说就是需要输入用户名和密码的shell。因此通常不管以何种方式登陆机器后用户获得的第一个shell就是login shell。

interactive意为交互式，这也很好理解，interactive shell会有一个输入提示符，并且它的标准输入、输出和错误输出都会显示在控制台上。所以一般来说只要是需要用户交互的，即一个命令一个命令的输入的shell都是interactive shell。而如果无需用户交互，它便是non-interactive shell。通常来说如bash script.sh此类执行脚本的命令就会启动一个non-interactive shell，它不需要与用户进行交互，执行完后它便会退出创建的Shell。

在interactive + login shell模式中，Shell首先会加载/etc/profile文件，然后再尝试依次去加载下列三个配置文件之一，一旦找到其中一个便不再接着寻找：

~/.bash_profile
~/.bash_login
~/.profile
2. 通过SSH直接执行远程命令和脚本
这种方式会使用Bash的non-interactive + non-login shell模式，它会创建一个shell，执行完脚本之后便退出，不再需要与用户交互。

no-login shell，顾名思义就是不是在登录Linux系统时启动的（比如你在命令行提示符上输入bash启动）。它不会去执行/etc/profile文件，而会去用户的HOME目录检查.bashrc并加载。

系统执行Shell脚本的时候，就是属于这种non-interactive shell。Bash通过BASH_ENV环境变量来记录要加载的文件，默认情况下这个环境变量并没有设置。如果有指定文件，那么Shell会先去加载这个文件里面的内容，然后再开始执行Shell脚本。

3. 结论
由此可见，如果要解决SSH远程执行命令时找不到自定义环境变量的问题，那么可以在登录用户的HOME目录的.bashrc中添加需要的环境变量。
```

解决，将需要的$PATH环境变量添加到.bashrc文件中去

```
[hadoop@h101 ~]$ vi .bashrc
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi

# Uncomment the following line if you don't like systemctl's auto-paging feature:
# export SYSTEMD_PAGER=

# User specific aliases and functions
PATH=$PATH:$HOME/.local/bin:$HOME/bin

export PATH
export C3_HOME=/opt/c3
export PATH=$C3_HOME/usr/bin:$PATH
export MANPATH=$MANPATH:$C3_HOME/doc/man

export JAVA_HOME=/usr/local/java7
export PATH=$JAVA_HOME/bin:$PATH

export HADOOP_HOME=/home/hadoop/hadoop-2.7.7
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH

export ZK_HOME=$HOME/zookeeper-3.4.12
export PATH=$ZK_HOME/bin:$PATH
```

```
[hadoop@h101 ~]$ cpush .bashrc /home/hadoop/.bashrc
```

果然环境变量就有了

```
[hadoop@h101 ~]$ ssh h102 env
MANPATH=:/opt/c3/doc/man
XDG_SESSION_ID=85
SELINUX_ROLE_REQUESTED=
SHELL=/bin/bash
HADOOP_HOME=/home/hadoop/hadoop-2.7.7
SSH_CLIENT=192.168.56.101 47636 22
SELINUX_USE_CURRENT_RANGE=
ZK_HOME=/home/hadoop/zookeeper-3.4.12
USER=hadoop
MAIL=/var/mail/hadoop
PATH=/home/hadoop/zookeeper-3.4.12/bin:/home/hadoop/hadoop-2.7.7/bin:/home/hadoop/hadoop-2.7.7/sbin:/usr/local/java7/bin:/opt/c3/usr/bin:/usr/local/bin:/usr/bin:/home/hadoop/.local/bin:/home/hadoop/bin
C3_HOME=/opt/c3
PWD=/home/hadoop
JAVA_HOME=/usr/local/java7
LANG=zh_CN.UTF-8
SELINUX_LEVEL_REQUESTED=
SHLVL=1
HOME=/home/hadoop
LOGNAME=hadoop
SSH_CONNECTION=192.168.56.101 47636 192.168.56.102 22
LESSOPEN=||/usr/bin/lesspipe.sh %s
XDG_RUNTIME_DIR=/run/user/1001
_=/usr/bin/env
```



命令也能找到了

```
[hadoop@h101 ~]$ cexec jps
************************* h *************************
--------- h101---------
12167 DFSZKFailoverController
11989 JournalNode
28898 Jps
11779 NameNode
--------- h102---------
8725 ResourceManager
8332 NameNode
17059 Jps
8406 DataNode
8837 NodeManager
8624 DFSZKFailoverController
8505 JournalNode
8251 QuorumPeerMain
--------- h103---------
4438 QuorumPeerMain
5007 NodeManager
5450 Jps
4600 JournalNode
4503 DataNode
4703 ResourceManager
--------- h104---------
4146 Jps
3646 DataNode
3764 NodeManager
3587 QuorumPeerMain
```

