---
layout: post
title: ZooKeeper的入门指南
---
说明：本文翻译自ZooKeeper3.3.6版本的官网入门指南。由于自己水平有限，难免有不当之处，请包容和指出。官方英文文档地址：<http://zookeeper.apache.org/doc/r3.3.6/zookeeperStarted.html>

###入门：使用ZooKeeper协调分布式应用
本文档包含的信息用来帮助你快速入门ZooKeeper。主要针对开发人员希望能够尝试一下ZooKeeper，包含了安装一个ZooKeeper服务器的简单安装说明，几个验证它是否正在运行的命令，和一个简单的编程示例。最后，为了方便，还有几节是关于复杂安装的，例如运行复制的部署和优化事务日志。然而，对于商业部署的完整说明，请参阅的[ZooKeeper管理员指南](http://zookeeper.apache.org/doc/r3.3.6/zookeeperAdmin.html)。

###先决条件
见管理员指南中的[系统要求](http://zookeeper.apache.org/doc/r3.3.6/zookeeperAdmin.html#sc_systemReq)。

###下载
获取ZooKeeper的发布包，从Apache下载镜像下载最近的[稳定版本](http://hadoop.apache.org/zookeeper/releases.html)。

###单机操作
设置一个在单机模式下ZooKeeper服务器很简单。服务器被包含在一个独立的JAR文件中，所以安装只需要创建配置。
一旦你下载了一个稳定ZooKeeper版本，释放它然后cd到root目录
启动的ZooKeeper你需要一个配置文件。下面是一个例子，创建conf/zoo.cfg：
```
tickTime=2000
dataDir=/var/zookeeper
clientPort=2181
```
这个文件可以叫任意名字, 但为了便于描述称之为conf/zoo.cfg。改变dataDir值指定一个现有的空目录。下面是每个字段的意义：
**tickTime**
ZooKeeper使用的基本时间单元，以毫秒为单位。它被用来做心跳监测，最小会话超时时间是2陪的tickTime。
**dataDir**
存储内存数据库快照的位置，除非特殊说明，也包括事务的更新的日志。
**clientPort**
监听客户端连接的端口。

现在您创建好了配置文件，可以启动ZooKeeper：
```
bin/zkServer.sh start
```
ZooKeeperd日志信息使用log4j -- 更多的详细信息在程序员指南的[日志](http://zookeeper.apache.org/doc/r3.3.6/zookeeperProgrammers.html#Logging)章节。你会看到log4j配置控制台日志和(或)日志文件。

这里的步骤描述了在单机模式下运行ZooKeeper。没有复制，所以如果ZooKeeper进程失败，服务将会停止。这在大多数的开发情况下是允许的，运行复制模式下的ZooKeeper，请参阅后面的运行复制的ZooKeeper。

###管理ZooKeeper存储
对于长时间运行的生产系统，ZooKeeper的存储必须外部管理(dataDir and logs)。请参[阅维](http://zookeeper.apache.org/doc/r3.3.6/zookeeperAdmin.html#sc_maintenance)护部分获取更多细节。

###链接ZooKeeper
一旦ZooKeeper在运行，你可以有几个选项来链接它：
* java:使用
```
bin/zkCli.sh -server 127.0.0.1:2181
```
* C:在ZooKeeper源码里的src/c子目录，运行`make cli_mt`或`make cli_st`来编译cli_mt(多线程)或cli_st（单线程）。参阅参阅src/c中的README获取全部细节。你可以从src/c目录中运行程序：
```
LD_LIBRARY_PATH=. cli_mt 127.0.0.1:2181
```
或者
```
LD_LIBRARY_PATH=. cli_st 127.0.0.1:2181
```
这可以提供一下简单的shell，像是在ZooKeeper上操作文件系统。

一旦你链接上了，你会看到类似这样的信息：
```
Connecting to localhost:2181
log4j:WARN No appenders could be found for logger (org.apache.zookeeper.ZooKeeper).
log4j:WARN Please initialize the log4j system properly.
Welcome to ZooKeeper!
JLine support is enabled
[zkshell: 0]
```
在shell里，键入help可以获取可以在客户端执行的命令清单，就像这样：
```
[zk: 127.0.0.1:2181(CONNECTED) 0] help
ZooKeeper -server host:port cmd args
        stat path [watch]
        set path data [version]
        ls path [watch]
        delquota [-n|-b] path
        ls2 path [watch]
        setAcl path acl
        setquota -n|-b val path
        history 
        redo cmdno
        printwatches on|off
        delete path [version]
        sync path
        listquota path
        get path [watch]
        create [-s] [-e] path data acl
        addauth scheme auth
        quit 
        getAcl path
        close 
        connect host:port
```
在这里，你可以尝试一些简单的命令来感受这个简单的命令行界面。首先，通过发出list命令，如ls：
```
[zk: 127.0.0.1:2181(CONNECTED) 2] ls /
[zookeeper]
```
接下来，通过运行create /zk_test my_data创建一个新的znode。这将创建一个新的znode，且关联字符串“my_data”的节点。您应该看到：
```
[zk: 127.0.0.1:2181(CONNECTED) 3] create /zk_test my_data
Created /zk_test
```
再发出一个ls/命令来查看该目录的样子：
```
[zk: 127.0.0.1:2181(CONNECTED) 4] ls /
[zookeeper, zk_test]
```
注意，zk_test目录现已创建。
接下来，运行get命令验证与znode关联的数据，如：
```
[zk: 127.0.0.1:2181(CONNECTED) 5] get /zk_test
my_data
cZxid = 0x4
ctime = Mon May 30 18:06:33 CST 2016
mZxid = 0x4
mtime = Mon May 30 18:06:33 CST 2016
pZxid = 0x4
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 7
numChildren = 0
```
我们可以通过发出set命令，更改与zk_test关联的数据，如：
```
[zk: 127.0.0.1:2181(CONNECTED) 6] set /zk_test junk
cZxid = 0x4
ctime = Mon May 30 18:06:33 CST 2016
mZxid = 0x5
mtime = Mon May 30 18:12:11 CST 2016
pZxid = 0x4
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 0
```
请注意，我们设置数据后做了一个get并执行，确实发生了变化。
最后，让我们发送命令删除节点：
```
[zk: 127.0.0.1:2181(CONNECTED) 7] delete /zk_test
[zk: 127.0.0.1:2181(CONNECTED) 8] ls /
[zookeeper]
```
现在就这样。要探索更多，继续本文档的部分，并阅读[程序员指南](http://zookeeper.apache.org/doc/r3.3.6/zookeeperProgrammers.html)。

###ZooKeeper编程
ZooKeeper的有Java绑定和C绑定。它们在功能上是一样的。在C绑定中存在两个变种：单线程和多线程。它们的区别只是消息如何循环传递。更多的信息，查阅ZooKeeper程序员指南里的示例代码，关于如何使用不同的API的简单的[示例代码](http://zookeeper.apache.org/doc/r3.3.6/zookeeperProgrammers.html#ch_programStructureWithExample)。

###运行复制的ZooKeeper
在单机模式下运行ZooKeeper便于评估开发和测试。但在生产中，您应该复制模式下运行的ZooKeeper。相同应用的服务器复制组叫quorum，在复制的模式中，quorum里的所有服务器都具有相同的配置文件的副本。该文件类似与用于独立模式，但有几个差异。这里就是一个例子：
```
tickTime=2000
dataDir=/var/zookeeper
clientPort=2181
initLimit=5
syncLimit=2
server.1=zoo1:2888:3888
server.2=zoo2:2888:3888
server.3=zoo3:2888:3888
```
新的属性，initLimit是quorum中的ZooKeeper服务器连接到leader的超时时间。The entry syncLimit limits how far out of date a server can be from a leader.

有了这两个超时，你可以使用tickTime指定时间单位。在这个例子中，initLimit的超时时间是5 ticks，一个tick是2000毫秒，即10秒钟。

属性server.X列出了组成ZooKeeper服务的服务器。当服务器启动时，通过在data目录寻找myid文件，它就知道自己是哪个服务。该文件包含了服务器编号，使用ASCII编码。

最后，请注意每个服务器名称后的端口号：“2888”和“3888”。节点使用前一个端口连接到其他节点。这样的连接是必要的，以便节点可以沟通，例如，同意顺序更新。更具体地说，一个ZooKeeper服务器使用此端口将followers连接到leader。当一个新的leader出现，follower打开一个TCP连接使用此端口连接到leader。因为默认的leader选举也使用TCP，为了leader选举我们目前需要另一个端口。这是server.X的第二个端口。
>注意：如果你想在一台机器上测试多个服务器，在服务器配置文件中为每个server.X指定servername为localhost，和独有的quorum和leader选举端口（在上面的例子中，即2888:3888，2889:3889，2890:3890）。X服务器的配置文件。当然，不同的dataDirs和唯一的clientPorts也是必要的（在上面复制的例子中，在一个独立的本地主机运行，你仍拥有三的配置文件）。

###其他优化
有一对配置参数，可以大大提高性能：
* 要获得更新的低延迟有一个专门的事务日志目录是很重要的。默认情况下事务日志与数据库快照、myid文件在同一个目录。dataLogDir参数表示为事务日志使用不同的目录。
* [待定：什么是其他配置参数？]


