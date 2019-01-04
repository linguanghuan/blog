# flume安装测试

官网： http://flume.apache.org/

下载：

http://mirrors.hust.edu.cn/apache/flume/1.8.0/apache-flume-1.8.0-bin.tar.gz

```
[root@h101 soft]# wget http://mirrors.hust.edu.cn/apache/flume/1.8.0/apache-flume-1.8.0-bin.tar.gz
--2019-01-03 09:08:35--  http://mirrors.hust.edu.cn/apache/flume/1.8.0/apache-flume-1.8.0-bin.tar.gz
正在解析主机 mirrors.hust.edu.cn (mirrors.hust.edu.cn)... 202.114.18.160
正在连接 mirrors.hust.edu.cn (mirrors.hust.edu.cn)|202.114.18.160|:80... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度：58688757 (56M) [application/octet-stream]
正在保存至: “apache-flume-1.8.0-bin.tar.gz”

100%[=================================================================================================>] 58,688,757  2.96MB/s 用时 24s    

2019-01-03 09:08:59 (2.35 MB/s) - 已保存 “apache-flume-1.8.0-bin.tar.gz” [58688757/58688757])
```



```
[root@h101 soft]# tar -zxvf apache-flume-1.8.0-bin.tar.gz
[root@h101 soft]# mv apache-flume-1.8.0-bin /opt/
[root@h101 soft]# 

```

设置环境变量

```
export FLUME_HOME=/opt/apache-flume-1.8.0-bin
export PATH=$FLUME_HOME/bin:$PATH

[root@h101 ~]# source .bash_profile
[root@h101 ~]# which flume-ng
/opt/apache-flume-1.8.0-bin/bin/flume-ng
```



```
[root@h101 ~]# cd $FLUME_HOME
[root@h101 apache-flume-1.8.0-bin]# cp conf/flume-conf.properties.template conf/flume.conf
[root@h101 apache-flume-1.8.0-bin]# cp conf/flume-env.sh.template conf/flume-env.sh
```

参考getting started测试

https://cwiki.apache.org//confluence/display/FLUME/Getting+Started



配置flume.conf

> [root@h101 apache-flume-1.8.0-bin]# vi conf/flume.conf

```

# Define a memory channel called ch1 on agent1
agent1.channels.ch1.type = memory
 
# Define an Avro source called avro-source1 on agent1 and tell it
# to bind to 0.0.0.0:41414. Connect it to channel ch1.
agent1.sources.avro-source1.channels = ch1
agent1.sources.avro-source1.type = avro
agent1.sources.avro-source1.bind = 0.0.0.0
agent1.sources.avro-source1.port = 41414
 
# Define a logger sink that simply logs all events it receives
# and connect it to the other end of the same channel.
agent1.sinks.log-sink1.channel = ch1
agent1.sinks.log-sink1.type = logger
 
# Finally, now that we've defined all of our components, tell
# agent1 which ones we want to activate.
agent1.channels = ch1
agent1.sources = avro-source1
agent1.sinks = log-sink1
```

启动

> [root@h101 apache-flume-1.8.0-bin]# bin/flume-ng agent --conf ./conf/ -f conf/flume.conf -Dflume.root.logger=DEBUG,console -n agent1

端口监听

> [root@h101 bin]# netstat  -an|grep 41414
> tcp6       0      0 :::41414                :::*                    LISTEN  



只有ipv6的监听，正常ipv4去连这个端口连不上的。

配置的是全部0.0.0.0,为什么只有ipv6的？

> [root@h101 etc]# sysctl -a|grep ipv6|grep bind

```
sysctl: reading key "net.ipv6.conf.all.stable_secret"
sysctl: reading key "net.ipv6.conf.default.stable_secret"
sysctl: reading key "net.ipv6.conf.enp0s3.stable_secret"
sysctl: reading key "net.ipv6.conf.enp0s8.stable_secret"
sysctl: reading key "net.ipv6.conf.lo.stable_secret"
net.ipv6.bindv6only = 0
net.ipv6.ip_nonlocal_bind = 0
```

> [samba为啥不监听ipv4地址](http://blog.51cto.com/mosquito/1069920)

```
继续查看内核文档
#cat /usr/share/doc/kernel-doc-2.6.32/Documentation/networking/ip-sysctl.txt | grep bindv6only -n5
----
bindv6only - BOOLEAN
        Default value for IPV6_V6ONLY socket option,
        which restricts use of the IPv6 socket to IPv6 communication
        only.
                TRUE: disable IPv4-mapped address feature
                FALSE: enable IPv4-mapped address feature
        Default: FALSE (as specified in RFC2553bis)
内核文档里面对这个参数进行了说明，它是一个bool值开关。
设置为1时，关闭ipv4映射地址的特性
设置为0时，开启ipv4映射地址的特性
默认设置为0。详情需要参考RFC2553。

通过查看RFC2553获得了一些信息：
 RFC2553描述了IPv4映射地址和IPv6通配绑定套字的特殊行为。规格允许：
通过AF_INET6通配绑定套接字接受IPv4连接。
使用特殊形式的地址 (如 ::ffff:10.1.1.1 ) 通过AF_INET6套接字传输IPv4数据包。

       通过RFC2553规定的规则，将linux默认所有来自IPv4地址的访问转换为IPv6地址的格式从而处理来自于IPv4的连接。

     当bindv6only这个内核参数设置为0时，对所有来自于ipv4的请求都绑定到ipv6地址。简单说就是端口可以接收ipv4的包，也可以接收ipv6的包。
      当bindv6only这个内核参数设置为1时，对于来自ipv4的请求就打开多个端口进行监听和处理。Ipv4与ipv6所监听的端口是分开的。

结论：bindv6only参数是linux内核默认开启的参数，并且不会对系统运行产生什么不良影响。应当是ipv4向ipv6过渡时，为了向下兼容所开发的两种端口的运行模式。
```

说明该地址形式是可以直接连接的。可能是防火墙的问题。

systemctl stop firewalld

关闭防火墙之后，就可以连接了。



写代码将日志推送到flume





