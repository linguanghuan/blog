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



## 配置hive-default.xml

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

这里为什么还是提示用derby?

https://blog.csdn.net/seashouwang/article/details/77867134

```
hive2.3初始化mysql不起作用，主要原因是安装包自身有问题。删除该解压文件，使用其他版本。
```



http://mirrors.shu.edu.cn/apache/hive/

选择更低版本的hive，只有

http://mirrors.shu.edu.cn/apache/hive/hive-1.2.2/

那就选择hive-1.2.2试下

```
[hadoop@h101 conf]$ pwd
/home/hadoop/apache-hive-1.2.2-bin/conf
[hadoop@h101 conf]$ cp hive-default.xml.template hive-default.xml

```

```
export JAVA_HOME=/usr/local/java7
export HADOOP_HOME=/home/hadoop/hadoop-2.7.7
export HIVE_CONF_DIR=/home/hadoop/apache-hive-1.2.2-bin/conf
export HIVE_AUX_JARS_PATH=/home/hadoop/apache-hive-1.2.2-bin/lib
```

参考：

http://hive.apache.org/index.html

https://cwiki.apache.org/confluence/display/Hive/GettingStarted

[HIVE 2.3.4 本地安装与部署 （Ubuntu）](https://www.cnblogs.com/standingby/p/10039974.html)

[Hive之 hive-1.2.1 + hadoop 2.7.4 集群安装](https://www.cnblogs.com/andy6/p/7536958.html)

