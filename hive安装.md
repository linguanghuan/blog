# hive安装

安装到h101上

## 安装mysql

因为Hive的默认元数据是Mysql，所以先要安装Mysql

将mysql安装在h104上吧

[mysql安装](mysql安装.md)



### mysql建hive库和授权

新建hive数据库 

```
CREATE DATABASE IF NOT EXISTS hive default charset utf8 COLLATE utf8_general_ci; 
```

新建用户和授权

```
CREATE USER 'hive'@'%' IDENTIFIED BY 'hive';
GRANT ALL ON `hive`.* TO 'hive'@'%';
```



### only_full_group_by处理

执行成功后，一直报这个错误

```
[Err] 1055 - Expression #1 of ORDER BY clause is not in GROUP BY clause and contains nonaggregated column 'information_schema.PROFILING.SEQ' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
```



```
select @@sql_mode
```

结果

```
ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

去除ONLY_FULL_GROUP_BY,

```
STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

```
set @@sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION'
```



> [MySql版本问题sql_mode=only_full_group_by的完美解决方案](https://blog.csdn.net/liuyunshengsir/article/details/79525031)

```
1、查看sql_mode
select @@sql_mode
查询出来的值为：
ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

2、去掉ONLY_FULL_GROUP_BY，重新设置值。
set @@sql_mode ='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';

3、上面是改变了全局sql_mode，对于新建的数据库有效。对于已存在的数据库，则需要在对应的数据下执行：
set sql_mode ='
STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER
,NO_ENGINE_SUBSTITUTION'
--------------------- 
作者：liuyunshengsir 
来源：CSDN 
原文：https://blog.csdn.net/liuyunshengsir/article/details/79525031 
版权声明：本文为博主原创文章，转载请附上博文链接！
```

因为已经先建了hive库了，所以还需要连上hive数据库，然后执行语句：

```
set sql_mode ='
STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER
,NO_ENGINE_SUBSTITUTION'
```



## 下载 上传 解压

```
tar -zxvf apache-hive-2.3.4-bin.tar.gz
```



```
[hadoop@h101 ~]$ cd apache-hive-2.3.4-bin
```



## 配置环境变量

```
export HIVE_HOME=$HOME/apache-hive-2.3.4-bin
export PATH=$HIVE_HOME/bin:$PATH
```

验证环境变量

```
[hadoop@h101 ~]$ source .bash_profile
[hadoop@h101 ~]$ which hive
~/apache-hive-2.3.4-bin/bin/hive
```



## 配置hive-site.xml

```
cd /home/hadoop/apache-hive-2.3.4-bin/conf
cp hive-default.xml.template hive-default.xml
vi hive-default.xml
```

查找：javax.jdo.option.ConnectionURL

修改:

```
jdbc:derby:;databaseName=metastore_db;create=true
```

改为：

```
jdbc:mysql://h104:3306/hive?useUnicode=true&amp;characterEncoding=UTF-8&amp;createDatabaseIfNotExist=true;useSSL=false
```



查找：javax.jdo.option.ConnectionDriverName

修改:

```
org.apache.derby.jdbc.EmbeddedDriver
```

改为：

```
com.mysql.jdbc.Driver
```

查找:

```
javax.jdo.option.ConnectionUserName

javax.jdo.option.ConnectionPassword
```

相应的改为mysql数据库连接的用户名密码



其实hive-defalut.xml中的配置是默认配置不要动。要覆盖的话，新建一个hive-site.xml即可。

参考:[HIVE 2.3.4 本地安装与部署 （Ubuntu）](https://www.cnblogs.com/standingby/p/10039974.html)

> 进入hive配置目录：
> `cd /usr/local/hive/conf`
> 将hive-default.xml.template重命名为hive-default.xml
> `sudo mv hive-default.xml.template hive-default.xml`
> 新建一个配置文件hive-site.xml并编辑
> `sudo vim hive-site.xml`
> 在hive-site.xml中写入MySQL配置信息（虽然我们还没有安装MySQL）：
>



## 拷贝mysql驱动jar包

下载：

https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-java-5.1.46.tar.gz

解压开，将mysql-connector-java-5.1.46-bin.jar 上传到

/home/hadoop/apache-hive-2.3.4-bin/lib

## 配置hive-env.sh

```
 cp hive-env.sh.template hive-env.sh
```

将：

```
# Set HADOOP_HOME to point to a specific hadoop install directory
# HADOOP_HOME=${bin}/../../hadoop

# Hive Configuration Directory can be controlled by:
# export HIVE_CONF_DIR=

# Folder containing extra libraries required for hive compilation/execution can be controlled by:
# export HIVE_AUX_JARS_PATH=

```

配置为：

```
export HADOOP_HOME=/home/hadoop/hadoop-2.7.7
export HIVE_CONF_DIR=/home/hadoop/apache-hive-2.3.4-bin/conf
export HIVE_AUX_JARS_PATH=/home/hadoop/apache-hive-2.3.4-bin/lib
```



## hdfs上创建相关库和配置权限

```
hadoop fs -mkdir -p /user/hive/warehouse
hadoop fs -mkdir -p /user/hive/tmp
hadoop fs -mkdir -p /user/hive/log
hadoop fs -chmod -R 777 /user/hive/warehouse
hadoop fs -chmod -R 777 /user/hive/tmp
hadoop fs -chmod -R 777 /user/hive/log
```

这些目录应该是hive默认的或者hive-default.xml

hive-defalult.xml中搜到了/user/hive/warehouse

## 初始化 initSchema

报错了。

```
[hadoop@h101 ~]$ which schematool 
~/apache-hive-2.3.4-bin/bin/schematool
[hadoop@h101 ~]$ schematool -initSchema -dbType mysql
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/hadoop/apache-hive-2.3.4-bin/lib/log4j-slf4j-impl-2.6.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/hadoop/hadoop-2.7.7/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Metastore connection URL:        jdbc:derby:;databaseName=metastore_db;create=true
Metastore Connection Driver :    org.apache.derby.jdbc.EmbeddedDriver
Metastore connection User:       APP
Starting metastore schema initialization to 2.3.0
Initialization script hive-schema-2.3.0.mysql.sql
Error: Syntax error: Encountered "<EOF>" at line 1, column 64. (state=42X01,code=30000)
org.apache.hadoop.hive.metastore.HiveMetaException: Schema initialization FAILED! Metastore state would be inconsistent !!
Underlying cause: java.io.IOException : Schema script failed, errorcode 2
Use --verbose for detailed stacktrace.
*** schemaTool failed ***
```



## 失败 --verbose

```
[hadoop@h101 ~]$  schematool -initSchema -dbType mysql --verbose
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/hadoop/apache-hive-2.3.4-bin/lib/log4j-slf4j-impl-2.6.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/hadoop/hadoop-2.7.7/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Metastore connection URL:        jdbc:derby:;databaseName=metastore_db;create=true
Metastore Connection Driver :    org.apache.derby.jdbc.EmbeddedDriver
Metastore connection User:       APP
Starting metastore schema initialization to 2.3.0
Initialization script hive-schema-2.3.0.mysql.sql
Connecting to jdbc:derby:;databaseName=metastore_db;create=true
Connected to: Apache Derby (version 10.10.2.0 - (1582446))
Driver: Apache Derby Embedded JDBC Driver (version 10.10.2.0 - (1582446))
Transaction isolation: TRANSACTION_READ_COMMITTED
0: jdbc:derby:> !autocommit on
Autocommit status: true
0: jdbc:derby:> /*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */
Error: Syntax error: Encountered "<EOF>" at line 1, column 64. (state=42X01,code=30000)
Closing: 0: jdbc:derby:;databaseName=metastore_db;create=true
org.apache.hadoop.hive.metastore.HiveMetaException: Schema initialization FAILED! Metastore state would be inconsistent !!
Underlying cause: java.io.IOException : Schema script failed, errorcode 2
org.apache.hadoop.hive.metastore.HiveMetaException: Schema initialization FAILED! Metastore state would be inconsistent !!
        at org.apache.hive.beeline.HiveSchemaTool.doInit(HiveSchemaTool.java:590)
        at org.apache.hive.beeline.HiveSchemaTool.doInit(HiveSchemaTool.java:563)
        at org.apache.hive.beeline.HiveSchemaTool.main(HiveSchemaTool.java:1145)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:601)
        at org.apache.hadoop.util.RunJar.run(RunJar.java:226)
        at org.apache.hadoop.util.RunJar.main(RunJar.java:141)
Caused by: java.io.IOException: Schema script failed, errorcode 2
        at org.apache.hive.beeline.HiveSchemaTool.runBeeLine(HiveSchemaTool.java:980)
        at org.apache.hive.beeline.HiveSchemaTool.runBeeLine(HiveSchemaTool.java:959)
        at org.apache.hive.beeline.HiveSchemaTool.doInit(HiveSchemaTool.java:586)
        ... 8 more
*** schemaTool failed ***
```

[hadoop@h101 apache-hive-2.3.4-bin]$ grep -rn "jdbc:derby:;databaseName=metastore_db" *
conf/hive-default.xml.template:545:    <value>jdbc:derby:;databaseName=metastore_db;create=true</value>



这里为什么还是提示用derby?

https://blog.csdn.net/seashouwang/article/details/77867134

```
hive2.3初始化mysql不起作用，主要原因是安装包自身有问题。删除该解压文件，使用其他版本。
```

尝试了用旧的版本，也没用。



## 问题解决

后面再搜索相关的资料

https://blog.csdn.net/java060515/article/details/84582381

发现这里用的是hive-site.xml而不是hive-default.xml，所以讲hive-defalult.xml重命名为hive-site.xml，就成功了

```
[hadoop@h101 conf]$ mv hive-default.xml hive-site.xml
```

```
[hadoop@h101 conf]$ schematool -initSchema -dbType mysql
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/hadoop/apache-hive-2.3.4-bin/lib/log4j-slf4j-impl-2.6.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/hadoop/hadoop-2.7.7/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Metastore connection URL:        jdbc:mysql://h104:3306/hive?createDatabaseIfNotExist=true&useSSL=false
Metastore Connection Driver :    com.mysql.jdbc.Driver
Metastore connection User:       hive
Starting metastore schema initialization to 2.3.0
Initialization script hive-schema-2.3.0.mysql.sql
Initialization script completed
schemaTool completed
```



# hive测试

输入hive

报错:

```
[hadoop@h101 ~]$ hive
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/hadoop/apache-hive-2.3.4-bin/lib/log4j-slf4j-impl-2.6.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/hadoop/hadoop-2.7.7/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]

Logging initialized using configuration in jar:file:/home/hadoop/apache-hive-2.3.4-bin/lib/hive-common-2.3.4.jar!/hive-log4j2.properties Async: true

Exception in thread "main" java.lang.RuntimeException: org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.ipc.StandbyException): Operation category READ is not supported in state standby
        at org.apache.hadoop.hdfs.server.namenode.ha.StandbyState.checkOperation(StandbyState.java:87)
        at org.apache.hadoop.hdfs.server.namenode.NameNode$NameNodeHAContext.checkOperation(NameNode.java:1802)
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.checkOperation(FSNamesystem.java:1321)
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.getFileInfo(FSNamesystem.java:3829)
        at org.apache.hadoop.hdfs.server.namenode.NameNodeRpcServer.getFileInfo(NameNodeRpcServer.java:1012)
        at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolServerSideTranslatorPB.getFileInfo(ClientNamenodeProtocolServerSideTranslatorPB.java:855)
        at org.apache.hadoop.hdfs.protocol.proto.ClientNamenodeProtocolProtos$ClientNamenodeProtocol$2.callBlockingMethod(ClientNamenodeProtocolProtos.java)
        at org.apache.hadoop.ipc.ProtobufRpcEngine$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine.java:616)
        at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:982)
        at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2217)
        at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2213)
        at java.security.AccessController.doPrivileged(Native Method)
```

搜索

```
Operation category READ is not supported in state standby
```

输入:

http://h101:50070/ namenode web访问不了（代理问题）

重启hdfs看下



发现h101和h102两个namenode都是standby

重启观察hadoop，发现后面有一个namenode为active了

再执行，有新的报错

```
[hadoop@h101 ~]$ hive
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/hadoop/apache-hive-2.3.4-bin/lib/log4j-slf4j-impl-2.6.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/hadoop/hadoop-2.7.7/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]

Logging initialized using configuration in jar:file:/home/hadoop/apache-hive-2.3.4-bin/lib/hive-common-2.3.4.jar!/hive-log4j2.properties Async: true
Exception in thread "main" java.lang.IllegalArgumentException: java.net.URISyntaxException: Relative path in absolute URI: ${system:java.io.tmpdir%7D/$%7Bsystem:user.name%7D
        at org.apache.hadoop.fs.Path.initialize(Path.java:205)
        at org.apache.hadoop.fs.Path.<init>(Path.java:171)
        at org.apache.hadoop.hive.ql.session.SessionState.createSessionDirs(SessionState.java:659)
        at org.apache.hadoop.hive.ql.session.SessionState.start(SessionState.java:582)
        at org.apache.hadoop.hive.ql.session.SessionState.beginStart(SessionState.java:549)
        at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:750)
        at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:686)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:601)
        at org.apache.hadoop.util.RunJar.run(RunJar.java:226)
        at org.apache.hadoop.util.RunJar.main(RunJar.java:141)
Caused by: java.net.URISyntaxException: Relative path in absolute URI: ${system:java.io.tmpdir%7D/$%7Bsystem:user.name%7D
        at java.net.URI.checkPath(URI.java:1804)
        at java.net.URI.<init>(URI.java:752)
        at org.apache.hadoop.fs.Path.initialize(Path.java:202)
        ... 12 more
```



```
[hadoop@h101 conf]$ cat hive-site.xml |grep tmpdir
    <value>${system:java.io.tmpdir}/${system:user.name}</value>
    <value>${system:java.io.tmpdir}/${hive.session.id}_resources</value>
    <value>${system:java.io.tmpdir}/${system:user.name}</value>
    <description>Local temporary directory used to persist intermediate indexing state, will default to JVM system property java.io.tmpdir.</description>
    <value>${system:java.io.tmpdir}/${system:user.name}/operation_logs</value>
```

https://stackoverflow.com/questions/27099898/java-net-urisyntaxexception-when-starting-hive

```

Put the following at the beginning of hive-site.xml

  <property>
    <name>system:java.io.tmpdir</name>
    <value>/tmp/hive/java</value>
  </property>
  <property>
    <name>system:user.name</name>
    <value>${user.name}</value>
  </property>
See also question
```

这里参考stackoverflow的做法。

再hive-site.xml的最前面，加上配置:

```
  <property>
    <name>system:java.io.tmpdir</name>
    <value>/tmp/hive/java</value>
  </property>
  <property>
    <name>system:user.name</name>
    <value>${user.name}</value>
  </property>
```



启动hive就成功了，配置目录的文件树结构为

```
[hadoop@h101 tmp]$ pwd
/tmp
[hadoop@h101 tmp]$ find hive
hive
hive/java
hive/java/b2b92117-92e3-4f4a-85e8-77d4ea68f7d5_resources
hive/java/hadoop
hive/java/hadoop/b2b92117-92e3-4f4a-85e8-77d4ea68f7d5
hive/java/hadoop/b2b92117-92e3-4f4a-85e8-77d4ea68f7d56766837216493829083.pipeout
hive/java/hadoop/b2b92117-92e3-4f4a-85e8-77d4ea68f7d52350230507362198461.pipeout
```

可见：${user.name} 代表当前用户

成功到Hive命令行了

```
[hadoop@h101 ~]$ hive
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/hadoop/apache-hive-2.3.4-bin/lib/log4j-slf4j-impl-2.6.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/hadoop/hadoop-2.7.7/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]

Logging initialized using configuration in jar:file:/home/hadoop/apache-hive-2.3.4-bin/lib/hive-common-2.3.4.jar!/hive-log4j2.properties Async: true
Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. tez, spark) or using Hive 1.X releases.
hive> 
```



```
hive> show databases;
OK
default
Time taken: 9.057 seconds, Fetched: 1 row(s)
hive> create table test1(tid int, tname string);
OK
Time taken: 1.296 seconds
hive> show tables;
OK
test1
Time taken: 0.083 seconds, Fetched: 1 row(s)
hive> drop table test1;
OK
Time taken: 3.172 seconds
hive> quit;

```

新建表的时候，mysql中tbl表也有一条记录

| TBL_ID | CREATE_TIME | DB_ID | LAST_ACCESS_TIME | OWNER  | RETENTION | SD_ID | TBL_NAME | TBL_TYPE      | VIEW_EXPANDED_TEXT | VIEW_ORIGINAL_TEXT | IS_REWRITE_ENABLED |      |
| ------ | ----------- | ----- | ---------------- | ------ | --------- | ----- | -------- | ------------- | ------------------ | ------------------ | ------------------ | ---- |
| 6      | 1547298122  | 1     | 0                | hadoop | 0         | 6     | test1    | MANAGED_TABLE |                    |                    | 0                  |      |

hdfs上也有这样的目录层级

`/user/hive/warehouse/test1`



新建数据库和表，并且查询数据

```
hive> create database db2;
OK
Time taken: 0.206 seconds
hive> use db2;
OK
```

```
hive> create table test_hive(id int, name string)
    > row format delimited fields terminated by '\t'
    >  stored as textfile; 
OK
Time taken: 0.382 seconds
```

| TBL_ID | CREATE_TIME | DB_ID | LAST_ACCESS_TIME | OWNER  | RETENTION | SD_ID | TBL_NAME      | TBL_TYPE      | VIEW_EXPANDED_TEXT | VIEW_ORIGINAL_TEXT | IS_REWRITE_ENABLED |      |
| ------ | ----------- | ----- | ---------------- | ------ | --------- | ----- | ------------- | ------------- | ------------------ | ------------------ | ------------------ | ---- |
| 6      | 1547298122  | 1     | 0                | hadoop | 0         | 6     | test1         | MANAGED_TABLE |                    |                    | 0                  |      |
| 7      | 1547298521  | **6** | 0                | hadoop | 0         | 7     | **test_hive** | MANAGED_TABLE |                    |                    | 0                  |      |

hdfs路径: `/user/hive/warehouse/db2.db/test_hive`



加载本地数据

构造:

```
[hadoop@h101 ~]$ mkdir data
[hadoop@h101 ~]$ cd data
[hadoop@h101 data]$ ls
[hadoop@h101 data]$ vi test_tb.txt
[hadoop@h101 data]$ cat test_tb.txt 
101     aa
102     bb
103     cc
```

加载:

```
hive> load data local inpath '/home/hadoop/data/test_tb.txt' into table test_hive;
Loading data to table db2.test_hive
OK
Time taken: 2.686 seconds
```

查询：

```
hive> select * from test_hive;
OK
101     aa
102     bb
103     cc
Time taken: 3.64 seconds, Fetched: 3 row(s)

hive> select * from test_hive where id > 101;
OK
102     bb
103     cc
Time taken: 1.075 seconds, Fetched: 2 row(s)
```

hdfs上就有test_tb.txt文件了
```
/user/hive/warehouse/db2.db/test_hive/test_tb.txt
```

说明load data local inpath 会把本地文件上传到hdfs上去



## 参考

http://hive.apache.org/index.html

https://cwiki.apache.org/confluence/display/Hive/GettingStarted

[HIVE 2.3.4 本地安装与部署 （Ubuntu）](https://www.cnblogs.com/standingby/p/10039974.html)

[滴滴云部署 Hadoop2.7.7+Hive2.3.4](https://blog.csdn.net/java060515/article/details/84582381)

[[HIVE 2.3.4 本地安装与部署 （Ubuntu）](https://www.cnblogs.com/standingby/p/10039974.html)](https://www.cnblogs.com/standingby/p/10039974.html)



