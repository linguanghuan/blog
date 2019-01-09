# zookeeper集群安装



h102 h103 h104三台上安装zookeeper



下载上传zookeeper-3.4.12.tar.gz到h102



解压

```
[hadoop@h102 ~]$ tar -zxvf zookeeper-3.4.12.tar.gz 
```



### 配置zookeeper

```
mkdir -p /home/hadoop/zkrun

cd /home/hadoop/zookeeper-3.4.12/conf
[hadoop@h102 conf]$ ls
configuration.xsl  log4j.properties  zoo_sample.cfg
[hadoop@h102 conf]$ cp zoo_sample.cfg zoo.cfg
```



zoo.cfg新增（或修改）以下配置

```
dataDir=/home/hadoop/zkrun/zookeeper
dataLogDir=/home/hadoop/zkrun/logs

server.1=h102:2888:3888
server.2=h103:2888:3888
server.3=h104:2888:3888
```



```
[hadoop@h102 conf]$ mkdir  /home/hadoop/zkrun/zookeeper/
[hadoop@h102 conf]$ echo 1 > /home/hadoop/zkrun/zookeeper/myid
```



#### 配置zookeeper环境变量

vi .bash_profile

```
export ZK_HOME=$HOME/zookeeper-3.4.12
export PATH=$ZK_HOME/bin:$PATH
```

source .bash_profile

验证环境变量生效

```
[hadoop@h102 bin]$ which zkServer.sh
~/zookeeper-3.4.12/bin/zkServer.sh
```



#### zookeeper拷贝到其他机器

```
[hadoop@h102 ~]$ scp -r zookeeper-3.4.12 hadoop@h103:/home/hadoop
[hadoop@h102 ~]$ scp -r zookeeper-3.4.12 hadoop@h104:/home/hadoop

[hadoop@h102 ~]$ scp .bash_profile hadoop@h103:/home/hadoop
[hadoop@h102 ~]$ scp .bash_profile hadoop@h104:/home/hadoop

[hadoop@h102 ~]$ scp -r zkrun hadoop@h103:/home/hadoop
[hadoop@h102 ~]$ scp -r zkrun hadoop@h104:/home/hadoop


```



#### 修改h103 h104 的myid

```
[hadoop@h103 ~]$ echo 2 > /home/hadoop/zkrun/zookeeper/myid
[hadoop@h104 ~]$ echo 3 > /home/hadoop/zkrun/zookeeper/myid 
```



#### 启动zookeeper

```
[hadoop@h102 ~]$ zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /home/hadoop/zookeeper-3.4.12/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED


[hadoop@h103 ~]$ source .bash_profile
[hadoop@h103 ~]$ zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /home/hadoop/zookeeper-3.4.12/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED

[hadoop@h104 ~]$ source .bash_profile
[hadoop@h104 ~]$ zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /home/hadoop/zookeeper-3.4.12/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

#### 检查进程QuorumPeerMain

```
[hadoop@h101 ~]$ cexec /usr/local/java7/bin/jps
************************* h *************************
--------- h101---------
1313 NameNode
1572 Jps
--------- h102---------
1583 Jps
1254 SecondaryNameNode
1550 QuorumPeerMain
1181 DataNode
--------- h103---------
1407 QuorumPeerMain
1185 DataNode
1442 Jps
--------- h104---------
1424 Jps
1394 QuorumPeerMain
1183 DataNode
```



#### 查看集群状态

```
[hadoop@h102 bin]$ zkServer.sh  status
ZooKeeper JMX enabled by default
Using config: /home/hadoop/zookeeper-3.4.12/bin/../conf/zoo.cfg
Mode: follower

[hadoop@h103 ~]$  zkServer.sh  status
ZooKeeper JMX enabled by default
Using config: /home/hadoop/zookeeper-3.4.12/bin/../conf/zoo.cfg
Mode: leader

[hadoop@h104 ~]$  zkServer.sh  status
ZooKeeper JMX enabled by default
Using config: /home/hadoop/zookeeper-3.4.12/bin/../conf/zoo.cfg
Mode: follower
```



#### 操作测试

```
[hadoop@h103 ~]$ zkCli.sh 
Connecting to localhost:2181
[zk: localhost:2181(CONNECTED) 0] ls
[zk: localhost:2181(CONNECTED) 1]

[hadoop@h103 ~]$ zkCli.sh -server h104:2181
...
[zk: h104:2181(CONNECTED) 0] ls
[zk: h104:2181(CONNECTED) 1] exit


```

#### 集群效果测试

在h102上创建一个/project的项目

```
[zk: h102:2181(CONNECTED) 0] create /project  zookeeper_project
Created /project
[zk: h102:2181(CONNECTED) 1] get /project
zookeeper_project
cZxid = 0x100000005
ctime = Wed Jan 09 19:41:47 CST 2019
mZxid = 0x100000005
mtime = Wed Jan 09 19:41:47 CST 2019
pZxid = 0x100000005
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 17
numChildren = 0

```



h103和h104也能直接get到相同的信息

```
[zk: h103:2181(CONNECTED) 0] ls
[zk: h103:2181(CONNECTED) 1] get /project
zookeeper_project
cZxid = 0x100000005
ctime = Wed Jan 09 19:41:47 CST 2019
mZxid = 0x100000005
mtime = Wed Jan 09 19:41:47 CST 2019
pZxid = 0x100000005
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 17
numChildren = 0

[zk: h104:2181(CONNECTED) 2] ls      
[zk: h104:2181(CONNECTED) 3] get /project
zookeeper_project
cZxid = 0x100000005
ctime = Wed Jan 09 19:41:47 CST 2019
mZxid = 0x100000005
mtime = Wed Jan 09 19:41:47 CST 2019
pZxid = 0x100000005
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 17
numChildren = 0
[zk: h104:2181(CONNECTED) 4] 

```







































