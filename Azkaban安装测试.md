# azkaban安装测试

## 从源码编译

https://azkaban.readthedocs.io/en/latest/getStarted.html#building-from-source

```
Azkaban builds use Gradle (downloads automatically when run using gradlew which is the Gradle wrapper) and requires Java 8 or higher.

The following commands run on *nix platforms like Linux, OS X.

# Build Azkaban
./gradlew build

# Clean the build
./gradlew clean

# Build and install distributions
./gradlew installDist

# Run tests
./gradlew test

# Build without running tests
./gradlew build -x test
These are all standard Gradle commands. Please look at Gradle documentation for more info.
```

1. 需要java8以上的环境，这里就用java8
2. 安装gradle



### 下载源码

github找到最新的发布版本（2018.9.26）

![1550022036858](C:\Users\gh\AppData\Roaming\Typora\typora-user-images\1550022036858.png)

h101新建编译的用户，编译的工作运行在这个用户上

```
[root@h101 ~]# useradd compile

```

设置密码，用compile用户登录h101

上传: azkaban-3.57.0.tar.gz

### 准备环境

设置java8环境

```
[compile@h101 ~]$ vi .bash_profile
export JAVA_HOME=/usr/local/java8
export PATH=$JAVA_HOME/bin:$PATH

[compile@h101 ~]$ source .bash_profile
[compile@h101 ~]$ which java
/usr/local/java8/bin/java

[compile@h101 ~]$ java -version
java version "1.8.0_171"
Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)

```

验证私服环境访问正常：

```
[compile@h101 azkaban-3.57.0]$ curl http://192.168.56.1:8081/nexus/content/groups/public/
```

### 编译

解压

进入目录, 修改build.gradle文件，修改maven配置为私服

```
     17 buildscript {
     18     repositories {
     19         maven {
     20             url 'http://192.168.56.1:8081/nexus/content/groups/public/'
     21         }
     22     }

```



```
     48 allprojects {
     49     apply plugin: 'jacoco'
     50 
     51     repositories {
     52         maven {
     53             url 'http://192.168.56.1:8081/nexus/content/groups/public/'
     54         }
     55         mavenLocal()
     56     }
     57 }

```



```
Build Azkaban and create an installation package:

cd azkaban; ./gradlew build installDist
```

进入解压开的目录，执行

```
 ./gradlew build installDist -x test
```



报错，显示没安装git命令

```
Download https://plugins.gradle.org/m2/commons-codec/commons-codec/1.6/commons-codec-1.6.jar
Download https://plugins.gradle.org/m2/net/ltgt/gradle/gradle-errorprone-plugin/0.0.14/gradle-errorprone-plugin-0.0.14.jar

FAILURE: Build failed with an exception.

* Where:
Build file '/home/compile/azkaban-3.57.0/build.gradle' line: 40

* What went wrong:
A problem occurred evaluating root project 'azkaban'.
> Failed to apply plugin [id 'com.cinnober.gradle.semver-git']
   > Cannot run program "git" (in directory "/home/compile/azkaban-3.57.0"): error=2, 没有那个文件或目录

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.

* Get more help at https://help.gradle.org

BUILD FAILED in 5m 20s

```



安装git

```
[root@h101 ~]# yum install git
......
[root@h101 ~]# git --version
git version 1.8.3.1

```



![1550024906118](C:\Users\gh\AppData\Roaming\Typora\typora-user-images\1550024906118.png)

碰到这类错误，直接重新运行

```
 ./gradlew build installDist -x test
```



```
> Task :azkaban-common:compileJava 
错误: 读取/home/compile/.gradle/caches/modules-2/files-2.1/mysql/mysql-connector-java/5.1.28/63638386ed42c57e52a57cd488b998ff1b9726e9/mysql-connector-java-5.1.28.jar时出错; invalid LOC header (bad signature)
注: 某些输入文件使用或覆盖了已过时的 API。

```

maven私服上的该文件有问题，删除maven私服上的这个版本的mysql central（仓库），再删除

```
rm -rf /home/compile/.gradle/caches/modules-2/files-2.1/mysql/mysql-connector-java/5.1.28/
```

再重跑，直到没有报错最后提示成功

```
[compile@h101 distributions]$ pwd
/home/compile/azkaban-3.57.0/azkaban-solo-server/build/distributions
[compile@h101 distributions]$ ll
总用量 46752
-rw-rw-r--. 1 compile compile 23868898 2月  13 10:51 azkaban-solo-server-0.1.0-SNAPSHOT.tar.gz
-rw-rw-r--. 1 compile compile 24001205 2月  13 10:51 azkaban-solo-server-0.1.0-SNAPSHOT.zip

```

编译完成后，生成以下文件

```
[compile@h101 azkaban-3.57.0]$ find .|grep distributions|grep zip
./az-crypto/build/distributions/az-crypto-0.1.0-SNAPSHOT.zip
./az-hadoop-jobtype-plugin/build/distributions/az-hadoop-jobtype-plugin-0.1.0-SNAPSHOT.zip
./az-hdfs-viewer/build/distributions/az-hdfs-viewer-0.1.0-SNAPSHOT.zip
./az-jobsummary/build/distributions/az-jobsummary-0.1.0-SNAPSHOT.zip
./az-reportal/build/distributions/az-reportal-0.1.0-SNAPSHOT.zip
./azkaban-db/build/distributions/azkaban-db-0.1.0-SNAPSHOT.zip
./azkaban-exec-server/build/distributions/azkaban-exec-server-0.1.0-SNAPSHOT.zip
./azkaban-hadoop-security-plugin/build/distributions/azkaban-hadoop-security-plugin-0.1.0-SNAPSHOT.zip
./azkaban-solo-server/build/distributions/azkaban-solo-server-0.1.0-SNAPSHOT.zip
./azkaban-web-server/build/distributions/azkaban-web-server-0.1.0-SNAPSHOT.zip

```

```
[compile@h101 azkaban-3.57.0]$ find .|grep distributions|grep tar.gz
./az-crypto/build/distributions/az-crypto-0.1.0-SNAPSHOT.tar.gz
./az-hadoop-jobtype-plugin/build/distributions/az-hadoop-jobtype-plugin-0.1.0-SNAPSHOT.tar.gz
./az-hdfs-viewer/build/distributions/az-hdfs-viewer-0.1.0-SNAPSHOT.tar.gz
./az-jobsummary/build/distributions/az-jobsummary-0.1.0-SNAPSHOT.tar.gz
./az-reportal/build/distributions/az-reportal-0.1.0-SNAPSHOT.tar.gz
./azkaban-db/build/distributions/azkaban-db-0.1.0-SNAPSHOT.tar.gz
./azkaban-exec-server/build/distributions/azkaban-exec-server-0.1.0-SNAPSHOT.tar.gz
./azkaban-hadoop-security-plugin/build/distributions/azkaban-hadoop-security-plugin-0.1.0-SNAPSHOT.tar.gz
./azkaban-solo-server/build/distributions/azkaban-solo-server-0.1.0-SNAPSHOT.tar.gz
./azkaban-web-server/build/distributions/azkaban-web-server-0.1.0-SNAPSHOT.tar.gz

```

这里下载所有的tar.gz吧。

```
[compile@h101 azkaban-3.57.0]$ sz ./az-hadoop-jobtype-plugin/build/distributions/az-hadoop-jobtype-plugin-0.1.0-SNAPSHOT.tar.gz
[compile@h101 azkaban-3.57.0]$ sz ./az-hdfs-viewer/build/distributions/az-hdfs-viewer-0.1.0-SNAPSHOT.tar.gz
[compile@h101 azkaban-3.57.0]$ sz ./az-jobsummary/build/distributions/az-jobsummary-0.1.0-SNAPSHOT.tar.gz
[compile@h101 azkaban-3.57.0]$ sz ./az-reportal/build/distributions/az-reportal-0.1.0-SNAPSHOT.tar.gz
[compile@h101 azkaban-3.57.0]$ sz ./azkaban-db/build/distributions/azkaban-db-0.1.0-SNAPSHOT.tar.gz
[compile@h101 azkaban-3.57.0]$ sz ./azkaban-exec-server/build/distributions/azkaban-exec-server-0.1.0-SNAPSHOT.tar.gz
[compile@h101 azkaban-3.57.0]$ sz ./azkaban-hadoop-security-plugin/build/distributions/azkaban-hadoop-security-plugin-0.1.0-SNAPSHOT.tar.gz
[compile@h101 azkaban-3.57.0]$ sz ./azkaban-solo-server/build/distributions/azkaban-solo-server-0.1.0-SNAPSHOT.tar.gz
[compile@h101 azkaban-3.57.0]$ sz ./azkaban-web-server/build/distributions/azkaban-web-server-0.1.0-SNAPSHOT.tar.gz

```

修改0.1.0-SNAPSHOT 为 3.57.0

将gradle的依赖也打包

```
[compile@h101 ~]$ tar -cvf gradle.azkaban-3.57.0.tar .gradle
```



## 安装





## 参考

https://azkaban.readthedocs.io/en/latest/



