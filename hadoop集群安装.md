# hadoop集群安装


# 机器规划
```
192.168.56.101 h101   namenode
192.168.56.102 h102   yarn datanode
192.168.56.103 h103   datanode
192.168.56.104 h104   datanode  （用于后续新加入节点）
```

## 防火墙

关闭防火墙

 cexec systemctl stop firewalld

开机禁用防火墙

 cexec systemctl disable firewalld



## 新建hadoop用户

[root@h101 ~]# cexec "useradd hadoop"
```
************************* h *************************
--------- h101---------
useradd：用户“hadoop”已存在
--------- h102---------
--------- h103---------
--------- h104---------
[root@h101 ~]# cexec "cat /etc/passwd|grep hadoop"
```

```
************************* h *************************
--------- h101---------
hadoop:x:1001:1001::/home/hadoop:/bin/bash
--------- h102---------
hadoop:x:1001:1001::/home/hadoop:/bin/bash
--------- h103---------
hadoop:x:1000:1000::/home/hadoop:/bin/bash
--------- h104---------
hadoop:x:1000:1000::/home/hadoop:/bin/bash

```



## 设置hadoop用户的密码

(echo -e "hadoop\nhadoop")|passwd hadoop

[root@h101 ~]# cexec "(echo -e \"hadoop\nhadoop\")|passwd hadoop"
```
************************* h *************************
--------- h101---------
新的 密码：无效的密码： 密码少于 8 个字符
重新输入新的 密码：更改用户 hadoop 的密码 。
passwd：所有的身份验证令牌已经成功更新。
--------- h102---------
新的 密码：无效的密码： 密码少于 8 个字符
重新输入新的 密码：更改用户 hadoop 的密码 。
passwd：所有的身份验证令牌已经成功更新。
--------- h103---------
新的 密码：无效的密码： 密码少于 8 个字符
重新输入新的 密码：更改用户 hadoop 的密码 。
passwd：所有的身份验证令牌已经成功更新。
--------- h104---------
新的 密码：无效的密码： 密码少于 8 个字符
重新输入新的 密码：更改用户 hadoop 的密码 。
passwd：所有的身份验证令牌已经成功更新。
```

## hadoop用户免密登录配置

[hadoop@h101 ~]$ ssh-keygen -t rsa

```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/hadoop/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/hadoop/.ssh/id_rsa.
Your public key has been saved in /home/hadoop/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:K7LlFmaCusZOsADPoVRwz70MPauIiFThpwrwLu2s3zI hadoop@h101
The key's randomart image is:
+---[RSA 2048]----+
| ..+             |
|  + + o          |
|o..o = +         |
|+=..o o +        |
|=.+o   +S        |
|B++ o =  .       |
|*=o..=o..        |
|o*E. =..         |
|==+oo..          |
+----[SHA256]-----+
```

ssh-copy-id 将h101的pub文件内容放到h101-104四台机器上

```
[hadoop@h101 ~]$ ssh-copy-id -i .ssh/id_rsa.pub h101
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: ".ssh/id_rsa.pub"
The authenticity of host 'h101 (192.168.56.101)' can't be established.
ECDSA key fingerprint is SHA256:epP1OFtThjD3D1WojGrAnYkmT09fmRQWs2pdRe/BzcA.
ECDSA key fingerprint is MD5:b8:0f:97:6b:a4:97:d5:a0:1a:dc:6d:2a:dd:26:53:a5.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
hadoop@h101's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'h101'"
and check to make sure that only the key(s) you wanted were added.

[hadoop@h101 ~]$ ssh-copy-id -i .ssh/id_rsa.pub h102
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: ".ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
hadoop@h102's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'h102'"
and check to make sure that only the key(s) you wanted were added.

[hadoop@h101 ~]$ ssh-copy-id -i .ssh/id_rsa.pub h103
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: ".ssh/id_rsa.pub"
The authenticity of host 'h103 (192.168.56.103)' can't be established.
ECDSA key fingerprint is SHA256:epP1OFtThjD3D1WojGrAnYkmT09fmRQWs2pdRe/BzcA.
ECDSA key fingerprint is MD5:b8:0f:97:6b:a4:97:d5:a0:1a:dc:6d:2a:dd:26:53:a5.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
hadoop@h103's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'h103'"
and check to make sure that only the key(s) you wanted were added.

[hadoop@h101 ~]$ ssh-copy-id -i .ssh/id_rsa.pub h104
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: ".ssh/id_rsa.pub"
The authenticity of host 'h104 (192.168.56.104)' can't be established.
ECDSA key fingerprint is SHA256:epP1OFtThjD3D1WojGrAnYkmT09fmRQWs2pdRe/BzcA.
ECDSA key fingerprint is MD5:b8:0f:97:6b:a4:97:d5:a0:1a:dc:6d:2a:dd:26:53:a5.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
hadoop@h104's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'h104'"
and check to make sure that only the key(s) you wanted were added.
```

验证免密

```
[hadoop@h101 ~]$ ssh h101
Last login: Tue Jan  8 20:49:33 2019 from h1
[hadoop@h101 ~]$ exit
登出
Connection to h101 closed.
[hadoop@h101 ~]$ ssh h102
Last login: Tue Jan  8 20:49:34 2019 from h1
[hadoop@h101 ~]$ ssh h103
Last login: Tue Jan  8 20:49:34 2019 from h1
[hadoop@h103 ~]$ exit
登出
Connection to h103 closed.
[hadoop@h101 ~]$ ssh h104
Last login: Tue Jan  8 20:49:34 2019 from h1
[hadoop@h104 ~]$ exit
登出
Connection to h104 closed.
```

## java版本选择和安装

http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html

```
Required software for Linux include:

Java™ must be installed. Recommended Java versions are described at HadoopJavaVersions.

ssh must be installed and sshd must be running to use the Hadoop scripts that manage remote Hadoop daemons.
```



HadoopJavaVersions

https://wiki.apache.org/hadoop/HadoopJavaVersions

```
Hadoop Java Versions
Version 2.7 and later of Apache Hadoop requires Java 7. It is built and tested on both OpenJDK and Oracle (HotSpot)'s JDK/JRE.

Earlier versions (2.6 and earlier) support Java 6.
```

这里hadoop的版本选择的是hadoop-2.7.7，根据以上描述需要java7

### Tested JDK

Here are the known JDKs in use or which have been tested:



- | **Version**                                                  | **Status** | **Reported By**       |
  | ------------------------------------------------------------ | ---------- | --------------------- |
  | **oracle 1.7.0_15**                                          | **Good**   | **Cloudera**          |
  | oracle 1.7.0_21                                              | Good (4)   | Hortonworks           |
  | oracle 1.7.0_45                                              | Good       | Pivotal               |
  | openjdk 1.7.0_09-icedtea                                     | Good (5)   | Hortonworks           |
  | oracle 1.6.0_16                                              | Avoid (1)  | Cloudera              |
  | oracle 1.6.0_18                                              | Avoid      | Many                  |
  | oracle 1.6.0_19                                              | Avoid      | Many                  |
  | oracle 1.6.0_20                                              | Good (2)   | LinkedIn, Cloudera    |
  | oracle [1.6.0_21](http://www.oracle.com/technetwork/java/javase/downloads/jdk6-jsp-136632.html) | Good (2)   | Yahoo!, Cloudera      |
  | oracle 1.6.0_24                                              | Good       | Cloudera              |
  | oracle 1.6.0_26                                              | Good(2)    | Hortonworks, Cloudera |
  | oracle 1.6.0_28                                              | Good       | LinkedIn              |
  | oracle 1.6.0_31                                              | Good(3, 4) | Cloudera, Hortonworks |

这里就根据这个测试报告，选择1.7.0_15这个jdk吧。

1.7.0_15

https://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html

解压到/usr/local目录之下

推送到其他机器

> cpush /usr/local/jdk1.7.0_15/ /usr/local/jdk1.7.0_15/

其他机器建立软连接

```
[root@h101 bin]# cexec "ln -s /usr/local/jdk1.7.0_15/ /usr/local/java7"
************************* h *************************
--------- h101---------
--------- h102---------
--------- h103---------
--------- h104---------
```



## h101上安装配置hadoop

hadoop用户上传hadoop-2.7.7.tar.gz 到h101

解压

> tar -zxvf hadoop-2.7.7.tar.gz 



### 配置环境变量

```
export HADOOP_HOME=/home/hadoop/hadoop-2.7.7
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
```







### 查看版本检查环境配置

```
[hadoop@h101 ~]$ hadoop version
Hadoop 2.7.7
Subversion Unknown -r c1aad84bd27cd79c3d1a7dd58202a8c3ee1ed3ac
Compiled by stevel on 2018-07-18T22:47Z
Compiled with protoc 2.5.0
From source with checksum 792e15d20b12c74bd6f19a1fb886490
This command was run using /home/hadoop/hadoop-2.7.7/share/hadoop/common/hadoop-common-2.7.7.jar
```



## 集群安装配置



参考：

https://blog.csdn.net/meepomiracle/article/details/81667653  (有官网文档为依据，但是启动过程没描述)

https://www.cnblogs.com/frankdeng/p/9047698.html  （过程比较完善，版本是2.7.6，主要参考这里的步骤）



### 配置JAVA_HOME

参考：https://hadoop.apache.org/docs/r2.7.6/hadoop-project-dist/hadoop-common/SingleCluster.html

```
cd /home/hadoop/hadoop-2.7.7/etc/hadoop
```

修改以下3个文件

```
hadoop-env.sh
mapred-env.sh 
yarn-env.sh
```

找到JAVA_HOME的位置，修改为以下内容

```
export JAVA_HOME=/usr/local/java7/
```

必须配置，不配置启动会报错找不到JAVA_HOME



### 配置core-site.xml

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
</configuration>
```



### 配置hdfs-site.xml

```
<configuration>
<!-- 设置dfs副本数，不设置默认是3个   -->
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
<!-- 设置secondname的端口   -->
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>h102:50090</value>
    </property>
</configuration>
```



### 配置mapred-site.xml

> cp mapred-site.xml.template mapred-site.xml

```
<configuration>
    <!-- 指定mr运行在yarn上 -->
    <property>
     <name>mapreduce.framework.name</name>
     <value>yarn</value>
    </property>
</configuration>
```



### 配置yarn-site.xml

```
<configuration>

<!-- Site specific YARN configuration properties -->

<!-- reducer获取数据的方式 -->
     <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
     </property>
<!-- 指定YARN的ResourceManager的地址 -->
     <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>h102</value>
     </property>
</configuration>
```



### 配置slaves

```
h102
h103
h104
```





### 拷贝文件到其他机器

> cpush hadoop-2.7.7 /home/hadoop/



躺坑：

> ~~cpush hadoop-2.7.7 /home/hadoop/hadoop-2.7.7~~

拷贝结果变成了：

```
/home/hadoop/hadoop-2.7.7/hadoop-2.7.7
```



环境变量配置覆盖

```
cpush .bash_profile /home/hadoop/.bash_profile 
```



## h101上格式化namenode

```
[hadoop@h101 hadoop]$ hdfs namenode -format
```

日志

```
19/01/08 23:10:11 INFO common.Storage: Storage directory /home/hadoop/run/dfs/name has been successfully formatted.
```



run目录生成一些文件

```
[hadoop@h101 ~]$ cd run
[hadoop@h101 run]$ find .
.
./dfs
./dfs/name
./dfs/name/current
./dfs/name/current/VERSION
./dfs/name/current/seen_txid
./dfs/name/current/fsimage_0000000000000000000.md5
./dfs/name/current/fsimage_0000000000000000000
```



## 启动hdfs

```
[hadoop@h101 hadoop]$ start-dfs.sh
Starting namenodes on [h101]
h101: starting namenode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-namenode-h101.out
h103: starting datanode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-datanode-h103.out
h104: starting datanode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-datanode-h104.out
h102: starting datanode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-datanode-h102.out
Starting secondary namenodes [h102]
h102: starting secondarynamenode, logging to /home/hadoop/hadoop-2.7.7/logs/hadoop-hadoop-secondarynamenode-h102.out

```

### 查看hdfs进程

```
[hadoop@h101 hadoop]$ which jps
/usr/local/java7/bin/jps
[hadoop@h101 hadoop]$ cexec /usr/local/java7/bin/jps
************************* h *************************
--------- h101---------
4228 NameNode
4475 Jps
--------- h102---------
2794 SecondaryNameNode
2844 Jps
2691 DataNode
--------- h103---------
2573 Jps
2469 DataNode
--------- h104---------
2565 Jps
2487 DataNode
```

### hdfs网页访问

http://h101:50070/





### 启动yarn

```
[hadoop@h101 hadoop]$ start-yarn.sh
starting yarn daemons
starting resourcemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-resourcemanager-h101.out
h103: starting nodemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-nodemanager-h103.out
h104: starting nodemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-nodemanager-h104.out
h102: starting nodemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-nodemanager-h102.out
```

发现ResoureManager启动失败，进程没有起来

> no resourcemanager to stop

```
[hadoop@h101 hadoop]$ stop-yarn.sh
stopping yarn daemons
no resourcemanager to stop
h102: stopping nodemanager
h104: stopping nodemanager
h103: stopping nodemanager
```



**启动Yarn：**　注意：Namenode和ResourceManger如果不是同一台机器，不能在NameNode上启动 yarn，应该在ResouceManager所在的机器上启动yarn。



因为yarn是配置在h102，所以要在h102上启动yarn

那h102也要配置到其他机器的免密登录

```
[hadoop@h102 ~]$ ssh-copy-id -i .ssh/id_rsa.pub h101
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: ".ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
hadoop@h101's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'h101'"
and check to make sure that only the key(s) you wanted were added.

[hadoop@h102 ~]$ ssh-copy-id -i .ssh/id_rsa.pub h102
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: ".ssh/id_rsa.pub"
The authenticity of host 'h102 (192.168.56.102)' can't be established.
ECDSA key fingerprint is SHA256:epP1OFtThjD3D1WojGrAnYkmT09fmRQWs2pdRe/BzcA.
ECDSA key fingerprint is MD5:b8:0f:97:6b:a4:97:d5:a0:1a:dc:6d:2a:dd:26:53:a5.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
hadoop@h102's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'h102'"
and check to make sure that only the key(s) you wanted were added.

[hadoop@h102 ~]$ ssh-copy-id -i .ssh/id_rsa.pub h103
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: ".ssh/id_rsa.pub"
The authenticity of host 'h103 (192.168.56.103)' can't be established.
ECDSA key fingerprint is SHA256:epP1OFtThjD3D1WojGrAnYkmT09fmRQWs2pdRe/BzcA.
ECDSA key fingerprint is MD5:b8:0f:97:6b:a4:97:d5:a0:1a:dc:6d:2a:dd:26:53:a5.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
hadoop@h103's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'h103'"
and check to make sure that only the key(s) you wanted were added.

[hadoop@h102 ~]$ ssh-copy-id -i .ssh/id_rsa.pub h104
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: ".ssh/id_rsa.pub"
The authenticity of host 'h104 (192.168.56.104)' can't be established.
ECDSA key fingerprint is SHA256:epP1OFtThjD3D1WojGrAnYkmT09fmRQWs2pdRe/BzcA.
ECDSA key fingerprint is MD5:b8:0f:97:6b:a4:97:d5:a0:1a:dc:6d:2a:dd:26:53:a5.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
hadoop@h104's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'h104'"
and check to make sure that only the key(s) you wanted were added.

```



验证免密登录

```
[hadoop@h102 ~]$ ssh h101
Last login: Tue Jan  8 23:59:39 2019 from h1
[hadoop@h101 ~]$ exit
登出
Connection to h101 closed.
[hadoop@h102 ~]$ ssh h102
Last login: Wed Jan  9 00:11:54 2019 from h101
[hadoop@h102 ~]$ exit
登出
Connection to h102 closed.
[hadoop@h102 ~]$ ssh h103
Last login: Tue Jan  8 22:04:30 2019 from h1
[hadoop@h103 ~]$ exit
登出
Connection to h103 closed.
[hadoop@h102 ~]$ ssh h104
Last login: Tue Jan  8 22:04:31 2019 from h1
[hadoop@h104 ~]$ exit
登出
Connection to h104 closed.
```



h102上启动yarn

```
[hadoop@h102 ~]$ start-yarn.sh
starting yarn daemons
starting resourcemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-resourcemanager-h102.out
h104: starting nodemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-nodemanager-h104.out
h103: starting nodemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-nodemanager-h103.out
h102: starting nodemanager, logging to /home/hadoop/hadoop-2.7.7/logs/yarn-hadoop-nodemanager-h102.out
```



### h101上cexec查看yarn和hdfs进程

hdfs和yarn都启动成功了

```
[hadoop@h101 ~]$ cexec /usr/local/java7/bin/jps
************************* h *************************
--------- h101---------
4824 Jps
4228 NameNode
--------- h102---------
2794 SecondaryNameNode
4067 Jps
3644 ResourceManager
3755 NodeManager
2691 DataNode
--------- h103---------
2882 NodeManager
3007 Jps
2469 DataNode
--------- h104---------
2900 NodeManager
3025 Jps
2487 DataNode
```



### yarn网页访问

http://h102:8088/cluster/nodes











































