# sqoop安装测试



http://sqoop.apache.org/

https://www.cnblogs.com/qingyunzong/p/8807252.html





下载：

sqoop-1.4.7.bin__hadoop-2.6.0.tar

```
[hadoop@h101 conf]$ pwd
/home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/conf

[hadoop@h101 conf]$ cp sqoop-env-template.sh sqoop-env.sh



```

```

export HADOOP_COMMON_HOME=/home/hadoop/hadoop-2.7.7
export HADOOP_MAPRED_HOME=/home/hadoop/hadoop-2.7.7
export HBASE_HOME=/home/hadoop/hbase-1.3.3
export HIVE_HOME=/home/hadoop/apache-hive-2.3.4-bin
export ZOOCFGDIR=/home/hadoop/zookeeper-3.4.12/conf
export ZOOKEEPER_HOME=/home/hadoop/zookeeper-3.4.12


```

上传mysql jar包驱动到：

```
[hadoop@h101 lib]$ pwd
/home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/lib
```



配置环境变量.bashrc

```
[hadoop@h101 ~]$ vi .bashrc
[hadoop@h101 ~]$ source .bash_profile
[hadoop@h101 ~]$ which sqoop
~/sqoop-1.4.7.bin__hadoop-2.6.0/bin/sqoop

```

```
export SQOOP_HOME=$HOME/sqoop-1.4.7.bin__hadoop-2.6.0
export PATH=$SQOOP_HOME/bin:$PATH

```



报错：

```
[hadoop@h101 ~]$ sqoop -version
/home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/bin/sqoop:行79: dirname: 未找到命令
/home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/bin/sqoop:行81: basename: 未找到命令
/home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/bin/sqoop:行97: dirname: 未找到命令
/home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/bin/sqoop:行100: /home/hadoop/configure-sqoop: 没有那个文件或目录
/home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/bin/sqoop:行101: /bin/hadoop: 没有那个文件或目录

```



```
[root@h101 ~]# yum whatprovides dirname
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * c7-media: 
c7-media                                                                                                                            | 3.6 kB  00:00:00     
c7-media/filelists_db                                                                                                               | 6.9 MB  00:00:00     
coreutils-8.22-21.el7.x86_64 : A set of basic GNU tools commonly used in shell scripts
源    ：c7-media
匹配来源：
文件名    ：/usr/bin/dirname



coreutils-8.22-21.el7.x86_64 : A set of basic GNU tools commonly used in shell scripts
源    ：@anaconda
匹配来源：
文件名    ：/usr/bin/dirname


[root@h101 ~]# yum whatprovides basename
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * c7-media: 
coreutils-8.22-21.el7.x86_64 : A set of basic GNU tools commonly used in shell scripts
源    ：c7-media
匹配来源：
文件名    ：/usr/bin/basename



coreutils-8.22-21.el7.x86_64 : A set of basic GNU tools commonly used in shell scripts
源    ：@anaconda
匹配来源：
文件名    ：/usr/bin/basename


```



其实有安装的，因为配置环境变量PATH配置坏了导致的

export PATH=$SQOOP_HOME/bin

改为：

export PATH=$SQOOP_HOME/bin:$PATH



设置完成后就正常执行了。

```
[hadoop@h101 ~]$ sqoop-version
Warning: /home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/../hcatalog does not exist! HCatalog jobs will fail.
Please set $HCAT_HOME to the root of your HCatalog installation.
Warning: /home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/../accumulo does not exist! Accumulo imports will fail.
Please set $ACCUMULO_HOME to the root of your Accumulo installation.
Warning: /home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/../zookeeper does not exist! Accumulo imports will fail.
Please set $ZOOKEEPER_HOME to the root of your Zookeeper installation.
19/02/19 15:19:11 INFO sqoop.Sqoop: Running Sqoop version: 1.4.7
Sqoop 1.4.7
git commit id 2328971411f57f0cb683dfb79d19d4d19d185dd8
Compiled by maugli on Thu Dec 21 15:59:58 STD 2017

```



查看mysql数据库

```
[hadoop@h101 conf]$ sqoop list-databases --connect jdbc:mysql://h104:3306/ --username azkaban --password azkaban
Warning: /home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/../hcatalog does not exist! HCatalog jobs will fail.
Please set $HCAT_HOME to the root of your HCatalog installation.
Warning: /home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/../accumulo does not exist! Accumulo imports will fail.
Please set $ACCUMULO_HOME to the root of your Accumulo installation.
19/02/19 15:25:27 INFO sqoop.Sqoop: Running Sqoop version: 1.4.7
19/02/19 15:25:27 WARN tool.BaseSqoopTool: Setting your password on the command-line is insecure. Consider using -P instead.
19/02/19 15:25:28 INFO manager.MySQLManager: Preparing to use a MySQL streaming resultset.
Tue Feb 19 15:25:29 CST 2019 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
information_schema
azkaban

```



从mysql导入hive

```
[hadoop@h101 conf]$ sqoop create-hive-table --connect jdbc:mysql://h104:3306/azkaban --username azkaban --password azkaban --table projects --hive-table hive_projects
```



```

Warning: /home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/../hcatalog does not exist! HCatalog jobs will fail.
Please set $HCAT_HOME to the root of your HCatalog installation.
Warning: /home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/../accumulo does not exist! Accumulo imports will fail.
Please set $ACCUMULO_HOME to the root of your Accumulo installation.
19/02/19 15:30:08 INFO sqoop.Sqoop: Running Sqoop version: 1.4.7
19/02/19 15:30:08 WARN tool.BaseSqoopTool: Setting your password on the command-line is insecure. Consider using -P instead.
19/02/19 15:30:08 INFO tool.BaseSqoopTool: Using Hive-specific delimiters for output. You can override
19/02/19 15:30:08 INFO tool.BaseSqoopTool: delimiters with --fields-terminated-by, etc.
19/02/19 15:30:09 INFO manager.MySQLManager: Preparing to use a MySQL streaming resultset.
Tue Feb 19 15:30:10 CST 2019 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
19/02/19 15:30:10 INFO manager.SqlManager: Executing SQL statement: SELECT t.* FROM `projects` AS t LIMIT 1
19/02/19 15:30:10 INFO manager.SqlManager: Executing SQL statement: SELECT t.* FROM `projects` AS t LIMIT 1
19/02/19 15:30:10 ERROR tool.CreateHiveTableTool: Encountered IOException running create table job: java.io.IOException: Hive does not support the SQL type for column settings_blob
	at org.apache.sqoop.hive.TableDefWriter.getCreateTableStmt(TableDefWriter.java:191)
	at org.apache.sqoop.hive.HiveImport.importTable(HiveImport.java:189)
	at org.apache.sqoop.tool.CreateHiveTableTool.run(CreateHiveTableTool.java:57)
	at org.apache.sqoop.Sqoop.run(Sqoop.java:147)
	at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:70)
	at org.apache.sqoop.Sqoop.runSqoop(Sqoop.java:183)
	at org.apache.sqoop.Sqoop.runTool(Sqoop.java:234)
	at org.apache.sqoop.Sqoop.runTool(Sqoop.java:243)
	at org.apache.sqoop.Sqoop.main(Sqoop.java:252)

```

```
Hive does not support the SQL type for column settings_blob
```



```
[hadoop@h101 conf]$ sqoop create-hive-table --connect jdbc:mysql://h104:3306/azkaban --username azkaban --password azkaban --table project_permissions --hive-table hive_projects
Warning: /home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/../hcatalog does not exist! HCatalog jobs will fail.
Please set $HCAT_HOME to the root of your HCatalog installation.
Warning: /home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/../accumulo does not exist! Accumulo imports will fail.
Please set $ACCUMULO_HOME to the root of your Accumulo installation.
19/02/19 15:32:39 INFO sqoop.Sqoop: Running Sqoop version: 1.4.7
19/02/19 15:32:39 WARN tool.BaseSqoopTool: Setting your password on the command-line is insecure. Consider using -P instead.
19/02/19 15:32:39 INFO tool.BaseSqoopTool: Using Hive-specific delimiters for output. You can override
19/02/19 15:32:39 INFO tool.BaseSqoopTool: delimiters with --fields-terminated-by, etc.
19/02/19 15:32:40 INFO manager.MySQLManager: Preparing to use a MySQL streaming resultset.
Tue Feb 19 15:32:41 CST 2019 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
19/02/19 15:32:41 INFO manager.SqlManager: Executing SQL statement: SELECT t.* FROM `project_permissions` AS t LIMIT 1
19/02/19 15:32:41 INFO manager.SqlManager: Executing SQL statement: SELECT t.* FROM `project_permissions` AS t LIMIT 1
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/hadoop/hadoop-2.7.7/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/hadoop/hbase-1.3.3/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
19/02/19 15:32:44 INFO retry.RetryInvocationHandler: Exception while invoking getFileInfo of class ClientNamenodeProtocolTranslatorPB over h102/192.168.56.102:8020 after 1 fail over attempts. Trying to fail over after sleeping for 1267ms.
org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.ipc.StandbyException): Operation category READ is not supported in state standby
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
	at javax.security.auth.Subject.doAs(Subject.java:415)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1762)
	at org.apache.hadoop.ipc.Server$Handler.run(Server.java:2211)

	at org.apache.hadoop.ipc.Client.call(Client.java:1476)
	at org.apache.hadoop.ipc.Client.call(Client.java:1413)
	at org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(ProtobufRpcEngine.java:229)
	at com.sun.proxy.$Proxy16.getFileInfo(Unknown Source)
	at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolTranslatorPB.getFileInfo(ClientNamenodeProtocolTranslatorPB.java:776)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:601)
	at org.apache.hadoop.io.retry.RetryInvocationHandler.invokeMethod(RetryInvocationHandler.java:191)
	at org.apache.hadoop.io.retry.RetryInvocationHandler.invoke(RetryInvocationHandler.java:102)
	at com.sun.proxy.$Proxy17.getFileInfo(Unknown Source)
	at org.apache.hadoop.hdfs.DFSClient.getFileInfo(DFSClient.java:2117)
	at org.apache.hadoop.hdfs.DistributedFileSystem$22.doCall(DistributedFileSystem.java:1305)
	at org.apache.hadoop.hdfs.DistributedFileSystem$22.doCall(DistributedFileSystem.java:1301)
	at org.apache.hadoop.fs.FileSystemLinkResolver.resolve(FileSystemLinkResolver.java:81)
	at org.apache.hadoop.hdfs.DistributedFileSystem.getFileStatus(DistributedFileSystem.java:1301)
	at org.apache.hadoop.fs.FileSystem.exists(FileSystem.java:1425)
	at org.apache.sqoop.hive.HiveImport.removeTempLogs(HiveImport.java:120)
	at org.apache.sqoop.hive.HiveImport.importTable(HiveImport.java:194)
	at org.apache.sqoop.tool.CreateHiveTableTool.run(CreateHiveTableTool.java:57)
	at org.apache.sqoop.Sqoop.run(Sqoop.java:147)
	at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:70)
	at org.apache.sqoop.Sqoop.runSqoop(Sqoop.java:183)
	at org.apache.sqoop.Sqoop.runTool(Sqoop.java:234)
	at org.apache.sqoop.Sqoop.runTool(Sqoop.java:243)
	at org.apache.sqoop.Sqoop.main(Sqoop.java:252)
19/02/19 15:32:45 INFO retry.RetryInvocationHandler: Exception while invoking getFileInfo of class ClientNamenodeProtocolTranslatorPB over h101/192.168.56.101:8020 after 2 fail over attempts. Trying to fail over after sleeping for 2226ms.

```





```
19/02/19 15:32:44 INFO retry.RetryInvocationHandler: Exception while invoking getFileInfo of class ClientNamenodeProtocolTranslatorPB over h102/192.168.56.102:8020 after 1 fail over attempts. Trying to fail over after sleeping for 1267ms.
org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.ipc.StandbyException): Operation category READ is not supported in state standby
```

namenode状态问题

```
http://h101:50070
http://h102:50070
```

状态都是standby



应该是应为zookeeper没启动。

启动zookeeper后，重新启动dfs

```
[hadoop@h101 ~]$ stop-dfs.sh
Stopping namenodes on [h101 h102]
h101: stopping namenode
h102: stopping namenode
h103: stopping datanode
h102: stopping datanode
h104: stopping datanode
Stopping journal nodes [h101 h102 h103]
h101: stopping journalnode
h102: stopping journalnode
h103: stopping journalnode
Stopping ZK Failover Controllers on NN hosts [h101 h102]
h102: no zkfc to stop
h101: no zkfc to stop
[hadoop@h101 ~]$ start-dfs.sh
Starting namenodes on [h101 h102]
h102: starting namenode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-namenode-h102.out
h101: starting namenode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-namenode-h101.out
h104: starting datanode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-datanode-h104.out
h103: starting datanode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-datanode-h103.out
h102: starting datanode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-datanode-h102.out
Starting journal nodes [h101 h102 h103]
h103: starting journalnode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-journalnode-h103.out
h102: starting journalnode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-journalnode-h102.out
h101: starting journalnode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-journalnode-h101.out
Starting ZK Failover Controllers on NN hosts [h101 h102]
h101: starting zkfc, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-zkfc-h101.out
h102: starting zkfc, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-zkfc-h102.out

```

重启后再检查，一个namenode是active了

再次执行：

```
[hadoop@h101 ~]$  sqoop create-hive-table --connect jdbc:mysql://h104:3306/azkaban --username azkaban --password azkaban --table project_permissions --hive-table hive_projects
Warning: /home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/../hcatalog does not exist! HCatalog jobs will fail.
Please set $HCAT_HOME to the root of your HCatalog installation.
Warning: /home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/../accumulo does not exist! Accumulo imports will fail.
Please set $ACCUMULO_HOME to the root of your Accumulo installation.
19/02/19 15:49:41 INFO sqoop.Sqoop: Running Sqoop version: 1.4.7
19/02/19 15:49:41 WARN tool.BaseSqoopTool: Setting your password on the command-line is insecure. Consider using -P instead.
19/02/19 15:49:41 INFO tool.BaseSqoopTool: Using Hive-specific delimiters for output. You can override
19/02/19 15:49:41 INFO tool.BaseSqoopTool: delimiters with --fields-terminated-by, etc.
19/02/19 15:49:41 INFO manager.MySQLManager: Preparing to use a MySQL streaming resultset.
Tue Feb 19 15:49:42 CST 2019 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
19/02/19 15:49:43 INFO manager.SqlManager: Executing SQL statement: SELECT t.* FROM `project_permissions` AS t LIMIT 1
19/02/19 15:49:43 INFO manager.SqlManager: Executing SQL statement: SELECT t.* FROM `project_permissions` AS t LIMIT 1
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/hadoop/hadoop-2.7.7/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/hadoop/hbase-1.3.3/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
19/02/19 15:49:45 INFO hive.HiveImport: Loading uploaded data into Hive
19/02/19 15:49:45 ERROR hive.HiveConfig: Could not load org.apache.hadoop.hive.conf.HiveConf. Make sure HIVE_CONF_DIR is set correctly.
19/02/19 15:49:45 ERROR tool.CreateHiveTableTool: Encountered IOException running create table job: java.io.IOException: java.lang.ClassNotFoundException: org.apache.hadoop.hive.conf.HiveConf
	at org.apache.sqoop.hive.HiveConfig.getHiveConf(HiveConfig.java:50)
	at org.apache.sqoop.hive.HiveImport.getHiveArgs(HiveImport.java:392)
	at org.apache.sqoop.hive.HiveImport.executeExternalHiveScript(HiveImport.java:379)
	at org.apache.sqoop.hive.HiveImport.executeScript(HiveImport.java:337)
	at org.apache.sqoop.hive.HiveImport.importTable(HiveImport.java:241)
	at org.apache.sqoop.tool.CreateHiveTableTool.run(CreateHiveTableTool.java:57)
	at org.apache.sqoop.Sqoop.run(Sqoop.java:147)
	at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:70)
	at org.apache.sqoop.Sqoop.runSqoop(Sqoop.java:183)
	at org.apache.sqoop.Sqoop.runTool(Sqoop.java:234)
	at org.apache.sqoop.Sqoop.runTool(Sqoop.java:243)
	at org.apache.sqoop.Sqoop.main(Sqoop.java:252)
Caused by: java.lang.ClassNotFoundException: org.apache.hadoop.hive.conf.HiveConf
	at java.net.URLClassLoader$1.run(URLClassLoader.java:366)
	at java.net.URLClassLoader$1.run(URLClassLoader.java:355)
	at java.security.AccessController.doPrivileged(Native Method)
	at java.net.URLClassLoader.findClass(URLClassLoader.java:354)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:423)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:308)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:356)
	at java.lang.Class.forName0(Native Method)
	at java.lang.Class.forName(Class.java:188)
	at org.apache.sqoop.hive.HiveConfig.getHiveConf(HiveConfig.java:44)
	... 11 more

```



```
19/02/19 15:49:45 ERROR hive.HiveConfig: Could not load org.apache.hadoop.hive.conf.HiveConf. Make sure HIVE_CONF_DIR is set correctly.

Caused by: java.lang.ClassNotFoundException: org.apache.hadoop.hive.conf.HiveConf
```



https://www.cnblogs.com/abcdwxc/p/8981543.html

```
 解决方法：

往/etc/profile最后加入 export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:$HIVE_HOME/lib/*
然后刷新配置，source /etc/profile
```



处理后

```
19/02/19 15:58:28 WARN ql.Driver: Caught exception attempting to write metadata call information org.apache.hadoop.hive.ql.metadata.HiveException: java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
org.apache.hadoop.hive.ql.metadata.HiveException: java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
	at org.apache.hadoop.hive.ql.metadata.Hive.registerAllFunctionsOnce(Hive.java:236)
	at org.apache.hadoop.hive.ql.metadata.Hive.<init>(Hive.java:388)
	at org.apache.hadoop.hive.ql.metadata.Hive.create(Hive.java:332)
	at org.apache.hadoop.hive.ql.metadata.Hive.getInternal(Hive.java:312)
	at org.apache.hadoop.hive.ql.metadata.Hive.get(Hive.java:354)
	at org.apache.hadoop.hive.ql.metadata.Hive.get(Hive.java:350)
	at org.apache.hadoop.hive.ql.Driver.dumpMetaCallTimingWithoutEx(Driver.java:683)
	at org.apache.hadoop.hive.ql.Driver.compile(Driver.java:621)
	at org.apache.hadoop.hive.ql.Driver.compileInternal(Driver.java:1317)
	at org.apache.hadoop.hive.ql.Driver.runInternal(Driver.java:1457)
	at org.apache.hadoop.hive.ql.Driver.run(Driver.java:1237)
	at org.apache.hadoop.hive.ql.Driver.run(Driver.java:1227)
	at org.apache.hadoop.hive.cli.CliDriver.processLocalCmd(CliDriver.java:233)
	at org.apache.hadoop.hive.cli.CliDriver.processCmd(CliDriver.java:184)
	at org.apache.hadoop.hive.cli.CliDriver.processLine(CliDriver.java:403)
	at org.apache.hadoop.hive.cli.CliDriver.processLine(CliDriver.java:336)
	at org.apache.hadoop.hive.cli.CliDriver.processReader(CliDriver.java:474)
	at org.apache.hadoop.hive.cli.CliDriver.processFile(CliDriver.java:490)
	at org.apache.hadoop.hive.cli.CliDriver.executeDriver(CliDriver.java:793)
	at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:759)
	at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:686)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:601)
	at org.apache.sqoop.hive.HiveImport.executeScript(HiveImport.java:331)
	at org.apache.sqoop.hive.HiveImport.importTable(HiveImport.java:241)
	at org.apache.sqoop.tool.CreateHiveTableTool.run(CreateHiveTableTool.java:57)
	at org.apache.sqoop.Sqoop.run(Sqoop.java:147)
	at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:70)
	at org.apache.sqoop.Sqoop.runSqoop(Sqoop.java:183)
	at org.apache.sqoop.Sqoop.runTool(Sqoop.java:234)
	at org.apache.sqoop.Sqoop.runTool(Sqoop.java:243)
	at org.apache.sqoop.Sqoop.main(Sqoop.java:252)
Caused by: java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
	at org.apache.hadoop.hive.metastore.MetaStoreUtils.newInstance(MetaStoreUtils.java:1708)
	at org.apache.hadoop.hive.metastore.RetryingMetaStoreClient.<init>(RetryingMetaStoreClient.java:83)
	at org.apache.hadoop.hive.metastore.RetryingMetaStoreClient.getProxy(RetryingMetaStoreClient.java:133)
	at org.apache.hadoop.hive.metastore.RetryingMetaStoreClient.getProxy(RetryingMetaStoreClient.java:104)
	at org.apache.hadoop.hive.ql.metadata.Hive.createMetaStoreClient(Hive.java:3600)
	at org.apache.hadoop.hive.ql.metadata.Hive.getMSC(Hive.java:3652)
	at org.apache.hadoop.hive.ql.metadata.Hive.getMSC(Hive.java:3632)
	at org.apache.hadoop.hive.ql.metadata.Hive.getAllFunctions(Hive.java:3894)
	at org.apache.hadoop.hive.ql.metadata.Hive.reloadFunctions(Hive.java:248)
	at org.apache.hadoop.hive.ql.metadata.Hive.registerAllFunctionsOnce(Hive.java:231)
	... 33 more
Caused by: java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:57)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:525)
	at org.apache.hadoop.hive.metastore.MetaStoreUtils.newInstance(MetaStoreUtils.java:1706)
	... 42 more
Caused by: MetaException(message:Version information not found in metastore. )
	at org.apache.hadoop.hive.metastore.RetryingHMSHandler.<init>(RetryingHMSHandler.java:83)
	at org.apache.hadoop.hive.metastore.RetryingHMSHandler.getProxy(RetryingHMSHandler.java:92)
	at org.apache.hadoop.hive.metastore.HiveMetaStore.newRetryingHMSHandler(HiveMetaStore.java:6891)
	at org.apache.hadoop.hive.metastore.HiveMetaStoreClient.<init>(HiveMetaStoreClient.java:164)
	at org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient.<init>(SessionHiveMetaStoreClient.java:70)
	... 47 more
Caused by: MetaException(message:Version information not found in metastore. )
	at org.apache.hadoop.hive.metastore.ObjectStore.checkSchema(ObjectStore.java:7564)
	at org.apache.hadoop.hive.metastore.ObjectStore.verifySchema(ObjectStore.java:7542)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:601)
	at org.apache.hadoop.hive.metastore.RawStoreProxy.invoke(RawStoreProxy.java:101)
	at com.sun.proxy.$Proxy37.verifySchema(Unknown Source)
	at org.apache.hadoop.hive.metastore.HiveMetaStore$HMSHandler.getMSForConf(HiveMetaStore.java:595)
	at org.apache.hadoop.hive.metastore.HiveMetaStore$HMSHandler.getMS(HiveMetaStore.java:588)
	at org.apache.hadoop.hive.metastore.HiveMetaStore$HMSHandler.createDefaultDB(HiveMetaStore.java:655)
	at org.apache.hadoop.hive.metastore.HiveMetaStore$HMSHandler.init(HiveMetaStore.java:431)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:601)
	at org.apache.hadoop.hive.metastore.RetryingHMSHandler.invokeInternal(RetryingHMSHandler.java:148)
	at org.apache.hadoop.hive.metastore.RetryingHMSHandler.invoke(RetryingHMSHandler.java:107)
	at org.apache.hadoop.hive.metastore.RetryingHMSHandler.<init>(RetryingHMSHandler.java:79)
	... 51 more
19/02/19 15:58:28 INFO ql.Driver: Completed compiling command(queryId=hadoop_20190219155809_760befee-c3aa-476a-843f-2adb21b6e04f); Time taken: 18.486 seconds
19/02/19 15:58:28 INFO conf.HiveConf: Using the default value passed in for log id: 45859556-85af-4577-9eb7-2c099e8bcd81
19/02/19 15:58:28 INFO session.SessionState: Resetting thread name to  main
19/02/19 15:58:28 INFO conf.HiveConf: Using the default value passed in for log id: 45859556-85af-4577-9eb7-2c099e8bcd81
19/02/19 15:58:28 INFO session.SessionState: Deleted directory: /tmp/hive/hadoop/45859556-85af-4577-9eb7-2c099e8bcd81 on fs with scheme hdfs
19/02/19 15:58:28 INFO session.SessionState: Deleted directory: /tmp/hadoop/45859556-85af-4577-9eb7-2c099e8bcd81 on fs with scheme file
19/02/19 15:58:28 INFO metastore.HiveMetaStore: 0: Opening raw store with implementation class:org.apache.hadoop.hive.metastore.ObjectStore
19/02/19 15:58:28 INFO metastore.ObjectStore: ObjectStore, initialize called
19/02/19 15:58:28 WARN DataNucleus.Query: Query for candidates of org.apache.hadoop.hive.metastore.model.MDatabase and subclasses resulted in no possible candidates
Required table missing : "DBS" in Catalog "" Schema "". DataNucleus requires this table to perform its persistence operations. Either your MetaData is incorrect, or you need to enable "datanucleus.schema.autoCreateTables"

```



根本的原因是应该没有读取到hive-site.xml的配置导致访问不到元数据

```
19/02/19 16:13:01 WARN common.LogUtils: hive-site.xml not found on CLASSPATH

```

https://stackoverflow.com/questions/14353394/hive-site-xml-not-found-on-classpath

```
Add export HADOOP_CLASSPATH= $HIVE_HOME/conf:$HIVE_HOME/lib in bash_rc or bash_profile
```



```
export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:$HIVE_HOME/lib/*:$HIVE_HOME/conf

```

修改后，找到了hive-site的配置文件了

```
19/02/19 16:18:33 INFO conf.HiveConf: Found configuration file file:/home/hadoop/apache-hive-2.3.4-bin/conf/hive-site.xml

```



继续报错：

```
19/02/19 16:22:09 ERROR exec.DDLTask: java.lang.NoSuchMethodError: com.fasterxml.jackson.databind.ObjectMapper.readerFor(Ljava/lang/Class;)Lcom/fasterxml/jackson/databind/ObjectReader


```

[Sqoop 把表导入 hive 出现的问题解决汇总](https://blog.csdn.net/zhangbcn/article/details/83014437)

```
[hadoop@h101 lib]$ rm jackson-*
[hadoop@h101 lib]$ pwd
/home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/lib

```



```
[hadoop@h101 lib]$ cp jackson-* /home/hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/lib
[hadoop@h101 lib]$ pwd
/home/hadoop/apache-hive-2.3.4-bin/lib

```

这么处理后，表是建成功了，但是数据没有过来。

```
hive> select * from hive_project_permissions;
OK
Time taken: 0.355 seconds
hive> desc hive_project_permissions;
OK
project_id          	string              	                    
modified_time       	bigint              	                    
name                	string              	                    
permissions         	int                 	                    
isgroup             	boolean             	                    
Time taken: 0.131 seconds, Fetched: 5 row(s)


hive> drop table hive_project_permissions;
OK
Time taken: 2.283 seconds
hive> 


```

是因为本来就是执行

```
sqoop create-hive-table ...
```



```
ail over after sleeping for 33503ms.
java.net.ConnectException: Call From h101/192.168.56.101 to h102:8032 failed on connection exception: java.net.ConnectException: 拒绝连接; For more details see:  http://wiki.apache.org/hadoop/ConnectionRefused

```

8032是yarn用的端口

重启yarn

```
[hadoop@h101 ~]$ stop-yarn.sh
stopping yarn daemons
no resourcemanager to stop
h103: no nodemanager to stop
h102: no nodemanager to stop
h104: no nodemanager to stop
no proxyserver to stop
[hadoop@h101 ~]$ start-yarn.sh
starting yarn daemons
starting resourcemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-resourcemanager-h101.out
h103: starting nodemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-nodemanager-h103.out
h104: starting nodemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-nodemanager-h104.out
h102: starting nodemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-nodemanager-h102.out

```



报错变成

```
19/02/19 17:12:10 ERROR tool.ImportTool: Import failed: java.io.IOException: Generating splits for a textual index column allowed only in case of "-Dorg.apache.sqoop.splitter.allow_text_splitter=true" property passed as a parameter
	at org.apache.sqoop.mapreduce.db.DataDrivenDBInputFormat.getSplits(DataDrivenDBInputFormat.java:204)
	at org.apache.hadoop.mapreduce.JobSubmitter.writeNewSplits(JobSubmitter.java:301)
	at org.apache.hadoop.mapreduce.JobSubmitter.writeSplits(JobSubmitter.java:318)
	at org.apache.hadoop.mapreduce.JobSubmitter.submitJobInternal(JobSubmitter.java:196)
	at org.apache.hadoop.mapreduce.Job$10.run(Job.java:1290)
	at org.apache.hadoop.mapreduce.Job$10.run(Job.java:1287)
	at java.security.AccessController.doPrivileged(Native Method)

```

http://www.cnblogs.com/harrymore/p/9057096.html



```
报错信息:"Generating splits for a textual index column allowed only in case of "-Dorg.apache.sqoop.splitter.allow_text_splitter=true" property passed as a parameter"。

主要问题是“--split-by id”这个参数指定的id是一个文本格式，所以需要在命令中加入选项"-Dorg.apache.sqoop.splitter.allow_text_splitter=true"，补齐命令：

sqoop import "-Dorg.apache.sqoop.splitter.allow_text_splitter=true" --connect jdbc:mysql://localhost:3306/test --username root -P --split-by id --columns id,name --table customer  --target-dir hdfs://harry.com:9000/user/cloudera/ingest/raw/customers --fields-terminated-by "," --hive-import --create-hive-table --hive-table sqoop_workspace.customers
```



加入该参数

```
[hadoop@h101 ~]$ sqoop import "-Dorg.apache.sqoop.splitter.allow_text_splitter=true" --connect jdbc:mysql://h104:3306/azkaban --username azkaban --password azkaban --table project_permissions --hive-table hive_project_permissions

```

```
9/02/19 17:16:28 WARN db.BigDecimalSplitter: Set BigDecimal splitSize to MIN_INCREMENT
19/02/19 17:16:28 INFO mapreduce.JobSubmitter: number of splits:1
19/02/19 17:16:29 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1550567508633_0002
19/02/19 17:16:30 INFO impl.YarnClientImpl: Submitted application application_1550567508633_0002
19/02/19 17:16:30 INFO mapreduce.Job: The url to track the job: http://h102:8088/proxy/application_1550567508633_0002/
19/02/19 17:16:30 INFO mapreduce.Job: Running job: job_1550567508633_0002
19/02/19 17:16:48 INFO mapreduce.Job: Job job_1550567508633_0002 running in uber mode : false
19/02/19 17:16:48 INFO mapreduce.Job:  map 0% reduce 0%
19/02/19 17:17:01 INFO mapreduce.Job:  map 100% reduce 0%
19/02/19 17:17:02 INFO mapreduce.Job: Job job_1550567508633_0002 completed successfully
19/02/19 17:17:02 INFO mapreduce.Job: Counters: 30
	File System Counters
		FILE: Number of bytes read=0
		FILE: Number of bytes written=147910
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=119
		HDFS: Number of bytes written=40
		HDFS: Number of read operations=4
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=2
	Job Counters 
		Launched map tasks=1
		Other local map tasks=1
		Total time spent by all maps in occupied slots (ms)=9559
		Total time spent by all reduces in occupied slots (ms)=0
		Total time spent by all map tasks (ms)=9559
		Total vcore-milliseconds taken by all map tasks=9559
		Total megabyte-milliseconds taken by all map tasks=9788416
	Map-Reduce Framework
		Map input records=1
		Map output records=1
		Input split bytes=119
		Spilled Records=0
		Failed Shuffles=0
		Merged Map outputs=0
		GC time elapsed (ms)=111
		CPU time spent (ms)=1620
		Physical memory (bytes) snapshot=142270464
		Virtual memory (bytes) snapshot=907366400
		Total committed heap usage (bytes)=31850496
	File Input Format Counters 
		Bytes Read=0
	File Output Format Counters 
		Bytes Written=40
19/02/19 17:17:02 INFO mapreduce.ImportJobBase: Transferred 40 bytes in 41.1364 seconds (0.9724 bytes/sec)
19/02/19 17:17:02 INFO mapreduce.ImportJobBase: Retrieved 1 records.
[hadoop@h101 ~]$ hive
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/hadoop/apache-hive-2.3.4-bin/lib/log4j-slf4j-impl-2.6.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/hadoop/hadoop-2.7.7/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]

Logging initialized using configuration in jar:file:/home/hadoop/apache-hive-2.3.4-bin/lib/hive-common-2.3.4.jar!/hive-log4j2.properties Async: true
Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. tez, spark) or using Hive 1.X releases.
hive> show tables;
OK
hive_project_permissions
test1
Time taken: 9.835 seconds, Fetched: 2 row(s)
hive> select * from hive_project_permissions;
OK
Time taken: 4.527 seconds

```

成功提交mapreduce，但是没有数据。

