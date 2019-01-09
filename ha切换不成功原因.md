# ha切换不成功原因

http://f.dataguru.cn/hadoop-707120-1-1.html



系统环境：centos7（最小化安装）  Hadoop版本：hadoop-2.5.2

在学习过程中，配置完HA进行验证：手动kill主namennode进程，但是备用namenode始终是standby状态，一直找不到原因，很是郁闷，还以为是hadoop版本跟老师将的不一样，进而需要不同的配置导致的。

今天，终于找到问题原因了。记录如下，希望对遇到此问题的朋友有所帮助。

解决办法：

在备用namenode上查看 hadoop-grid-zkfc-server102.log日志，发现异常如下

2016-10-16 00:09:32,465 WARN org.apache.hadoop.ha.SshFenceByTcpPort: <font color=#DC143C>**PATH=$PATH:/sbin:/usr/sbin fuser -v -k -n tcp 53310 via ssh: bash: fuser: command not found**</font>

<font color=#DC143C>2016-10-16 00:09:32,465 WARN org.apache.hadoop.ha.NodeFencer: Fencing method org.apache.hadoop.ha.SshFenceByTcpPort(null) was unsuccessful.</font>
2016-10-16 00:09:32,465 WARN org.apache.hadoop.ha.ActiveStandbyElector: Exception handling the winning of election
2016-10-16 00:09:34,552 WARN org.apache.hadoop.ha.FailoverController: Unable to gracefully make NameNode at server101.hadoop.com/192.168.1.101:53310 standby (unable to connect)
2016-10-16 00:09:34,592 WARN org.apache.hadoop.ha.SshFenceByTcpPort.jsch: Permanently added 'server101.hadoop.com' (RSA) to the list of known hosts.


由上面的警告可知：fuser: command not found，在做主备切换时执行fuser命令失败了。

查看hdfs-site.xml配置文件，

```
<property>
<name>dfs.ha.fencing.methods</name>
<value>sshfence</value>
</property>
```

hdfs-site.xml通过参数dfs.ha.fencing.methods来实现，在出现故障时通过哪种方式登录到另一个namenode上进行接管工作。

====》》》dfs.ha.fencing.methods参数解释

系统在任何时候只有一个namenode节点处于active状态。
在主备切换的时候，standby namenode会变成active状态，原来的active namenode就不能再处于active状态了，
否则两个namenode同时处于active状态会有问题。
所以在failover的时候要设置防止2个namenode都处于active状态的方法，可以是Java类或者脚本。

fencing的方法目前有两种，sshfence和shell

sshfence方法是指通过ssh登陆到active namenode节点杀掉namenode进程，所以你需要设置ssh无密码登陆，还要保证有杀掉namenode进程的权限。


以下是具体解决步骤：

查找fuser

```
[root@server101 ~]# yum provides "*/fuser"
Loaded plugins: fastestmirror
Determining fastest mirrors

epel: mirrors.tuna.tsinghua.edu.cn
  psmisc-22.20-9.el7.x86_64 : Utilities for managing processes on your system
  Repo        : base
  Matched from:
  Filename    : /usr/sbin/fuser
```



在namenode主、备节点上安装fuser(datanode节点不用安装)

```
[root@server101 ~]# yum -y install psmisc
[root@server102 ~]#  yum -y install psmisc
```

安装完成后，再次测试，namenode自动切换成功！！！





