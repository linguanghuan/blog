https://blog.csdn.net/weixin_40803329/article/details/80801216



http://hbase.apache.org/book.html#arch.overview

搜索:Not tested

找到：

| HBase-1.2.x       | HBase-1.3.x | HBase-1.5.x | HBase-2.0.x | HBase-2.1.x |      |
| ----------------- | ----------- | ----------- | ----------- | ----------- | ---- |
| Hadoop-2.4.x      | S           | S           | X           | X           | X    |
| Hadoop-2.5.x      | S           | S           | X           | X           | X    |
| Hadoop-2.6.0      | X           | X           | X           | X           | X    |
| Hadoop-2.6.1+     | S           | S           | X           | S           | X    |
| Hadoop-2.7.0      | X           | X           | X           | X           | X    |
| **Hadoop-2.7.1+** | S           | S           | S           | S           | S    |
| Hadoop-2.8.[0-1]  | X           | X           | X           | X           | X    |
| Hadoop-2.8.2      | NT          | NT          | NT          | NT          | NT   |
| Hadoop-2.8.3+     | NT          | NT          | NT          | S           | S    |
| Hadoop-2.9.0      | X           | X           | X           | X           | X    |
| Hadoop-2.9.1+     | NT          | NT          | NT          | NT          | NT   |
| Hadoop-3.0.x      | X           | X           | X           | X           | X    |
| Hadoop-3.1.0      | X           | X           | X           | X           | X    |



- "S" = supported
- "X" = not supported
- "NT" = Not tested



这里hadoop当然选2.7.1+版本，兼容性最好，选2.7.x中版本最高的吧。

https://hadoop.apache.org/releases.html

当前最高的是2.7.7

| Version   | Release date    | Source download                                              | Binary download                                              | Release notes                                                |
| --------- | --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 2.9.2     | 2018 Nov 19     | [source](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-2.9.2/hadoop-2.9.2-src.tar.gz) ([checksum](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-2.9.2/hadoop-2.9.2-src.tar.gz.mds) [signature](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-2.9.2/hadoop-2.9.2-src.tar.gz.asc)) | [binary](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-2.9.2/hadoop-2.9.2.tar.gz) ([checksum](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-2.9.2/hadoop-2.9.2.tar.gz.mds) [signature](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-2.9.2/hadoop-2.9.2.tar.gz.asc)) | [Announcement](https://hadoop.apache.org/release/2.9.2.html) |
| 2.8.5     | 2018 Sep 15     | [source](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-2.8.5/hadoop-2.8.5-src.tar.gz) ([checksum](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-2.8.5/hadoop-2.8.5-src.tar.gz.mds) [signature](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-2.8.5/hadoop-2.8.5-src.tar.gz.asc)) | [binary](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-2.8.5/hadoop-2.8.5.tar.gz) ([checksum](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-2.8.5/hadoop-2.8.5.tar.gz.mds) [signature](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-2.8.5/hadoop-2.8.5.tar.gz.asc)) | [Announcement](https://hadoop.apache.org/release/2.8.5.html) |
| 3.1.1     | 2018 Aug 8      | [source](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.1.1/hadoop-3.1.1-src.tar.gz) ([checksum](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-3.1.1/hadoop-3.1.1-src.tar.gz.mds) [signature](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-3.1.1/hadoop-3.1.1-src.tar.gz.asc)) | [binary](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.1.1/hadoop-3.1.1.tar.gz) ([checksum](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-3.1.1/hadoop-3.1.1.tar.gz.mds) [signature](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-3.1.1/hadoop-3.1.1.tar.gz.asc)) | [Announcement](https://hadoop.apache.org/release/3.1.1.html) |
| **2.7.7** | **2018 May 31** | [source](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-2.7.7/hadoop-2.7.7-src.tar.gz) ([checksum](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-2.7.7/hadoop-2.7.7-src.tar.gz.mds) [signature](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-2.7.7/hadoop-2.7.7-src.tar.gz.asc)) | [binary](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-2.7.7/hadoop-2.7.7.tar.gz) ([checksum](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-2.7.7/hadoop-2.7.7.tar.gz.mds) [signature](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-2.7.7/hadoop-2.7.7.tar.gz.asc)) | [Announcement](https://hadoop.apache.org/release/2.7.7.html) |
| 3.0.3     | 2018 May 31     | [source](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.0.3/hadoop-3.0.3-src.tar.gz) ([checksum](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-3.0.3/hadoop-3.0.3-src.tar.gz.mds) [signature](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-3.0.3/hadoop-3.0.3-src.tar.gz.asc)) | [binary](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.0.3/hadoop-3.0.3.tar.gz) ([checksum](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-3.0.3/hadoop-3.0.3.tar.gz.mds) [signature](https://dist.apache.org/repos/dist/release/hadoop/common/hadoop-3.0.3/hadoop-3.0.3.tar.gz.asc)) | [Announcement](https://hadoop.apache.org/release/3.0.3.html) |

那就选择这个版本了。

hbase就选择：

HBase-2.1.x



https://hbase.apache.org/book.html

 ZooKeeper Requirements

ZooKeeper 3.4.x is required.



http://hbase.apache.org/book.html#zookeeper

What version of ZooKeeper should I use?

The newer version, the better. ZooKeeper 3.4.x is required as of HBase 1.0.0



https://archive.apache.org/dist/zookeeper/stable/

看到3.4.12

那当然是选择这个了



https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-Requirements

Requirements

> - Java 1.7
>   *Note:*  Hive versions [1.2](https://issues.apache.org/jira/browse/HIVE/fixforversion/12329345/?selectedTab=com.atlassian.jira.jira-projects-plugin:version-summary-panel) onward require Java 1.7 or newer. Hive versions 0.14 to 1.1 work with Java 1.6 as well. Users are strongly advised to start moving to **Java 1.8** (see [HIVE-8607](https://issues.apache.org/jira/browse/HIVE-8607)).  
> - **Hadoop 2.x (preferred)**, 1.x (not supported by Hive 2.0.0 onward).
>   Hive versions up to 0.13 also supported Hadoop 0.20.x, 0.23.x.
> - Hive is commonly used in production Linux and Windows environment. Mac is a commonly used development environment. The instructions in this document are applicable to Linux and Mac. Using it on Windows would require slightly different steps.  



https://cwiki.apache.org/confluence/display/Hive/HBaseIntegration

> Version information
>
>
>
> Hive 1.x will remain compatible with HBase 0.98.x and lower versions. **Hive 2.x will be compatible with HBase 1.x and higher.** (See [HIVE-10990](https://issues.apache.org/jira/browse/HIVE-10990) for details.) Consumers wanting to work with HBase 1.x using Hive 1.x will need to compile Hive 1.x stream code themselves.



hive2.x与hbase1.x及比hbase1.x更高版本兼容

基于这个原因，这里hbase就选择1.5.x

hbase页面上直接提供下载的几个版本没有1.5.x的

https://hbase.apache.org/downloads.html

Releases

| Version | Release Date | Compatiblity Report                                          | Changes                                                      | Release Notes                                                | Download                                                     |
| ------- | ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 2.1.1   | 2018/10/31   | [2.1.0 vs 2.1.1](https://apache.org/dist/hbase/2.1.1/compatibility_report_2.1.0_vs_2.1.1.html) | [Changes](https://apache.org/dist/hbase/2.1.1/CHANGES.md)    | [Release Notes](https://apache.org/dist/hbase/2.1.1/RELEASENOTES.md) | [src](https://www.apache.org/dyn/closer.lua/hbase/2.1.1/hbase-2.1.1-src.tar.gz) ([sha512](https://apache.org/dist/hbase/2.1.1/hbase-2.1.1-src.tar.gz.sha512) [asc](https://apache.org/dist/hbase/2.1.1/hbase-2.1.1-src.tar.gz.asc))  [bin](https://www.apache.org/dyn/closer.lua/hbase/2.1.1/hbase-2.1.1-bin.tar.gz) ([sha512](https://apache.org/dist/hbase/2.1.1/hbase-2.1.1-bin.tar.gz.sha512) [asc](https://apache.org/dist/hbase/2.1.1/hbase-2.1.1-bin.tar.gz.asc))  [client-bin](https://www.apache.org/dyn/closer.lua/hbase/2.1.1/hbase-2.1.1-client-bin.tar.gz) ([sha512](https://apache.org/dist/hbase/2.1.1/hbase-2.1.1-client-bin.tar.gz.sha512) [asc](https://apache.org/dist/hbase/2.1.1/hbase-2.1.1-client-bin.tar.gz.asc)) |
| 2.0.4   | 2018/12/02   | [2.0.3 vs 2.0.4](https://apache.org/dist/hbase/2.0.4/compatibility_report_2.0.3vs2.0.4.html) | [Changes](https://apache.org/dist/hbase/2.0.4/CHANGES.md)    | [Release Notes](https://apache.org/dist/hbase/2.0.4/RELEASENOTES.md) | [src](https://apache.org/dist/hbase/2.0.4/hbase-2.0.4-src.tar.gz) ([sha512](https://apache.org/dist/hbase/2.0.4/hbase-2.0.4-src.tar.gz.sha512) [asc](https://apache.org/dist/hbase/2.0.4/hbase-2.0.4-src.tar.gz.asc))  [bin](https://apache.org/dist/hbase/2.0.4/hbase-2.0.4-bin.tar.gz) ([sha512](https://apache.org/dist/hbase/2.0.4/hbase-2.0.4-src.tar.gz.sha512) [asc](https://apache.org/dist/hbase/2.0.4/hbase-2.0.4-src.tar.gz.asc)) |
| 1.4.9   | 2018/12/14   | [1.4.8 vs 1.4.9](https://apache.org/dist/hbase/1.4.9/compat-check-report.html) | [Changes](https://github.com/apache/hbase/blob/rel/1.4.9/CHANGES.txt) | [Release Notes](https://s.apache.org/hbase-1.4.9-jira-release-notes) | [src](https://www.apache.org/dyn/closer.lua/hbase/1.4.9/hbase-1.4.9-src.tar.gz) ([sha512](https://apache.org/dist/hbase/1.4.9/hbase-1.4.9-src.tar.gz.sha512) [asc](https://apache.org/dist/hbase/1.4.9/hbase-1.4.9-src.tar.gz.asc))  [bin](https://www.apache.org/dyn/closer.lua/hbase/1.4.9/hbase-1.4.9-bin.tar.gz) ([sha512](https://apache.org/dist/hbase/1.4.9/hbase-1.4.9-bin.tar.gz.sha512) [asc](https://apache.org/dist/hbase/1.4.9/hbase-1.4.9-bin.tar.gz.asc)) |
| 1.3.3   | 2018/12/21   | [1.3.2 vs 1.3.3](https://apache.org/dist/hbase/1.3.3/compat-check-report.html) | [Changes](https://github.com/apache/hbase/blob/rel/1.3.3/CHANGES.txt) | [Release Notes](https://s.apache.org/hbase-1.3.3-jira-release-notes) | [src](https://www.apache.org/dyn/closer.lua/hbase/1.3.3/hbase-1.3.3-src.tar.gz) ([sha512](https://apache.org/dist/hbase/1.3.3/hbase-1.3.3-src.tar.gz.sha512) [asc](https://apache.org/dist/hbase/1.3.3/hbase-1.3.3-src.tar.gz.asc))  [bin](https://www.apache.org/dyn/closer.lua/hbase/1.3.3/hbase-1.3.3-bin.tar.gz) ([sha512](https://apache.org/dist/hbase/1.3.3/hbase-1.3.3-bin.tar.gz.sha512) [asc](https://apache.org/dist/hbase/1.3.3/hbase-1.3.3-bin.tar.gz.asc)) |
| 1.2.9   | 2018/11/27   | [1.2.8 vs 1.2.9](https://apache.org/dist/hbase/hbase-1.2.9/compat-check-report.html) | [Changes](https://github.com/apache/hbase/blob/rel/1.2.9/CHANGES.txt) | [Release Notes](https://s.apache.org/hbase-1.2.9-jira-release-notes) | [src](https://www.apache.org/dyn/closer.lua/hbase/hbase-1.2.9/hbase-1.2.9-src.tar.gz) ([sha512](https://apache.org/dist/hbase/hbase-1.2.9/hbase-1.2.9-src.tar.gz.sha512) [asc](https://apache.org/dist/hbase/hbase-1.2.9/hbase-1.2.9-src.tar.gz.asc))  [bin](https://www.apache.org/dyn/closer.lua/hbase/hbase-1.2.9/hbase-1.2.9-bin.tar.gz) ([sha512](https://apache.org/dist/hbase/hbase-1.2.9/hbase-1.2.9-bin.tar.gz.sha512) [asc](https://apache.org/dist/hbase/hbase-1.2.9/hbase-1.2.9-bin.tar.gz.asc)) |

这里有和hadoop兼容的就是1.3.3



所以hbase就选择1.3.3了



## 最终选择版本

```
apache-hive-2.3.4-bin.tar.gz
hadoop-2.7.7.tar.gz
hbase-1.3.3-bin.tar.gz
zookeeper-3.4.12.tar.gz
```





