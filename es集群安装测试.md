# es集群安装测试

## 集群安装

es版本选择

elasticsearch-6.4.0.tar.gz



```
[app@h101 ~]$ mkdir -p run/es

[app@h101 ~]$ tar -zxvf elasticsearch-6.4.0.tar.gz 

[app@h101 ~]$ ln -s elasticsearch-6.4.0 es

[app@h101 ~]$ cd es
[app@h101 es]$ vi config/elasticsearch.yml
```



```
cluster.name: log_es_cluster
path.data: /home/app/run/es/data
path.logs: /home/app/run/logs
node.name: h101

http.cors.enabled: true
http.cors.allow-origin: "*"
node.master: true
node.data: true
# 配置白名单 0.0.0.0表示其他机器都可访问
network.host: 0.0.0.0
transport.tcp.port: 9300
# tcp 传输压缩
transport.tcp.compress: true
http.port: 9200
discovery.zen.ping.unicast.hosts: ["h101","h102","h103"]



```



拷贝到h102 h103

```
[app@h101 ~]$ scp -r elasticsearch-6.4.0 app@h102:/home/app/elasticsearch-6.4.0
[app@h101 ~]$ scp -r elasticsearch-6.4.0 app@h103:/home/app/elasticsearch-6.4.0

```

创建软连接es

```
[app@h102 ~]$ ln -s elasticsearch-6.4.0 es
[app@h103 ~]$ ln -s elasticsearch-6.4.0 es

```



配置h102节点

> [app@h102 es]$ mkdir -p /home/app/run/es/data
>
> [app@h102 es]$ mkdir -p /home/app/run/es/logs
>
> [app@h102 es]$ vi config/elasticsearch.yml 

```
node.name: h102
node.master: false
```

配置h103节点

> [app@h103 es]$ mkdir -p /home/app/run/es/data
>
> [app@h103 es]$ mkdir -p /home/app/run/es/logs
>
> [app@h103 es]$ vi config/elasticsearch.yml 

```
node.name: h103
node.master: false

```



h101也执行创建以下配置文件里面配置的目录

```
[app@h101 ~]$ mkdir -p /home/app/run/es/data
[app@h101 ~]$ mkdir -p /home/app/run/es/logs

```



到h101 es的bin目录之下启动es

```
[app@h101 bin]$ ./elasticsearch

```

访问

http://h101:9200/

返回

```
{
  "name" : "h101",
  "cluster_name" : "log_es_cluster",
  "cluster_uuid" : "cbUqtNYKS1WzbaILbqblpQ",
  "version" : {
    "number" : "6.4.0",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "595516e",
    "build_date" : "2018-08-17T23:18:47.308994Z",
    "build_snapshot" : false,
    "lucene_version" : "7.4.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```



同样再启动h102

报错

```
[2019-01-17T20:50:34,512][WARN ][o.e.d.z.ZenDiscovery     ] [h102] failed to connect to master [{h101}{OIrBL2E8Q9euJ8k8id-Mww}{41H7dZL-SCOyDdJB5prEuw}{10.0.2.15}{10.0.2.15:9300}{ml.machine_memory=2096844800, ml.max_open_jobs=20, xpack.installed=true, ml.enabled=true}], retrying...
org.elasticsearch.transport.ConnectTransportException: [h101][10.0.2.15:9300] handshake failed. unexpected remote node {h102}{aG_ZKSWhT12rOx5IF5XweQ}{ElORmtWeSYC3eURsu80aow}{10.0.2.15}{10.0.2.15:9300}{ml.machine_memory=2096844800, ml.max_open_jobs=20, xpack.installed=true, ml.enabled=true}
	at org.elasticsearch.transport.TransportService.lambda$connectToNode$4(TransportService.java:333) ~[elasticsearch-6.4.0.jar:6.4.0]
	at org.elasticsearch.transport.TcpTransport.connectToNode(TcpTransport.java:543) ~[elasticsearch-6.4.0.jar:6.4.0]
	at org.elasticsearch.transport.TransportService.connectToNode(TransportService.java:329) ~[elasticsearch-6.4.0.jar:6.4.0]
	at org.elasticsearch.transport.TransportService.connectToNode(TransportService.java:316) ~[elasticsearch-6.4.0.jar:6.4.0]
	at org.elasticsearch.discovery.zen.ZenDiscovery.joinElectedMaster(ZenDiscovery.java:507) [elasticsearch-6.4.0.jar:6.4.0]
	at org.elasticsearch.discovery.zen.ZenDiscovery.innerJoinCluster(ZenDiscovery.java:475) [elasticsearch-6.4.0.jar:6.4.0]
	at org.elasticsearch.discovery.zen.ZenDiscovery.access$2500(ZenDiscovery.java:88) [elasticsearch-6.4.0.jar:6.4.0]
	at org.elasticsearch.discovery.zen.ZenDiscovery$JoinThreadControl$1.run(ZenDiscovery.java:1245) [elasticsearch-6.4.0.jar:6.4.0]
	at org.elasticsearch.common.util.concurrent.ThreadContext$ContextPreservingRunnable.run(ThreadContext.java:624) [elasticsearch-6.4.0.jar:6.4.0]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [?:1.8.0_171]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [?:1.8.0_171]
	at java.lang.Thread.run(Thread.java:748) [?:1.8.0_171]

```

有以下这段内容

```
 [h101][10.0.2.15:9300] 
```

这里应该是virtualbox内部ip导致的。是第一个网卡的地址。是互不相同的。

所以尝试修改以下配置

修改h101 es配置

```
network.host: 192.168.56.101
```

h102 es 配置

```
network.host: 192.168.56.102
```

h103 es 配置

```
network.host: 192.168.56.103
```



h103报错

```
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
[2019-01-17T20:50:26,237][INFO ][o.e.n.Node               ] [h103] stopping ...
[2019-01-17T20:50:26,274][INFO ][o.e.n.Node               ] [h103] stopped
[2019-01-17T20:50:26,275][INFO ][o.e.n.Node               ] [h103] closing ...
[2019-01-17T20:50:26,302][INFO ][o.e.n.Node               ] [h103] closed

```



切换到root用户下

> [root@h103 ~]# vi /etc/security/limits.conf
>
>

```
app             hard    nofile          65536
app             soft    nofile          65536
```



设置完成之后，重新登录app用户的shell，验证是否生效

> [app@h103 ~]$ ulimit -Hn
> 65536



> [root@h103 ~]# vi /etc/sysctl.conf

```
vm.max_map_count=262144
```

> [root@h103 ~]# sysctl -p
>
> vm.max_map_count = 262144



修改后，es启动成功

http://h102:9200/

```
{
  "name" : "h102",
  "cluster_name" : "log_es_cluster",
  "cluster_uuid" : "cbUqtNYKS1WzbaILbqblpQ",
  "version" : {
    "number" : "6.4.0",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "595516e",
    "build_date" : "2018-08-17T23:18:47.308994Z",
    "build_snapshot" : false,
    "lucene_version" : "7.4.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

http://h103:9200/

```
{
  "name" : "h103",
  "cluster_name" : "log_es_cluster",
  "cluster_uuid" : "cbUqtNYKS1WzbaILbqblpQ",
  "version" : {
    "number" : "6.4.0",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "595516e",
    "build_date" : "2018-08-17T23:18:47.308994Z",
    "build_snapshot" : false,
    "lucene_version" : "7.4.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```



可以看出集群id都是一样的

主节点访问

> http://h101:9200/_nodes/process?pretty

得到集群节点信息

```
{
  "_nodes" : {
    "total" : 3,
    "successful" : 3,
    "failed" : 0
  },
  "cluster_name" : "log_es_cluster",
  "nodes" : {
    "OIrBL2E8Q9euJ8k8id-Mww" : {
      "name" : "h101",
      "transport_address" : "192.168.56.101:9300",
      "host" : "192.168.56.101",
      "ip" : "192.168.56.101",
      "version" : "6.4.0",
      "build_flavor" : "default",
      "build_type" : "tar",
      "build_hash" : "595516e",
      "roles" : [
        "master",
        "data",
        "ingest"
      ],
      "attributes" : {
        "ml.machine_memory" : "2096844800",
        "xpack.installed" : "true",
        "ml.max_open_jobs" : "20",
        "ml.enabled" : "true"
      },
      "process" : {
        "refresh_interval_in_millis" : 1000,
        "id" : 11804,
        "mlockall" : false
      }
    },
    "Se7BEJR6RMa2Piy3k4OcHQ" : {
      "name" : "h103",
      "transport_address" : "192.168.56.103:9300",
      "host" : "192.168.56.103",
      "ip" : "192.168.56.103",
      "version" : "6.4.0",
      "build_flavor" : "default",
      "build_type" : "tar",
      "build_hash" : "595516e",
      "roles" : [
        "data",
        "ingest"
      ],
      "attributes" : {
        "ml.machine_memory" : "2096844800",
        "ml.max_open_jobs" : "20",
        "xpack.installed" : "true",
        "ml.enabled" : "true"
      },
      "process" : {
        "refresh_interval_in_millis" : 1000,
        "id" : 5407,
        "mlockall" : false
      }
    },
    "aG_ZKSWhT12rOx5IF5XweQ" : {
      "name" : "h102",
      "transport_address" : "192.168.56.102:9300",
      "host" : "192.168.56.102",
      "ip" : "192.168.56.102",
      "version" : "6.4.0",
      "build_flavor" : "default",
      "build_type" : "tar",
      "build_hash" : "595516e",
      "roles" : [
        "data",
        "ingest"
      ],
      "attributes" : {
        "ml.machine_memory" : "2096844800",
        "ml.max_open_jobs" : "20",
        "xpack.installed" : "true",
        "ml.enabled" : "true"
      },
      "process" : {
        "refresh_interval_in_millis" : 1000,
        "id" : 4816,
        "mlockall" : false
      }
    }
  }
}
```

http://h102:9200/_nodes/process?pretty

http://h103:9200/_nodes/process?pretty

也都能返回，只是节点排序不一样

主节点h101的roles

```
      "roles" : [
        "master",
        "data",
        "ingest"
      ],
```

其他节点h102和h103的roles

```
     "roles" : [
        "data",
        "ingest"
      ],
```



## 参考

[ELK 安装部署实战 (最新6.4.0版本)](https://www.cnblogs.com/woodylau/p/elk_latest.html)

https://www.elastic.co/guide/en/elasticsearch/reference/current/zip-targz.html





