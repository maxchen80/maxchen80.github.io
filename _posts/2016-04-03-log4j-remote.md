---
layout: post
title: 使用log4j远程日志高效管理集群环境下的日志
---
为了保证线上业务的稳定性，一般服务器数至少是2台，对于压力比较大的业务很可能更多，比如我们的某某业务，线上服务器达到了16台。

享受集群带来的高稳定和高吞吐福利的同时，必然也涉及到管理成本上升的问题，我们线上苦恼的一个问题就是这么多的服务器，排查日志是个苦力活，尤其是你不确定请求落在哪一台服务器的情况下。

解决这个问题，前公司有一套自研的远程日志组件，能够讲每台机器上的日志统一打印到远程日志服务器。现在的公司使用的是log4j,有没有远程日志功能呢？答案是肯定的。

##服务端配置
log4j-server.properties配置清单如下：
```
log4j.appender.remote=org.apache.log4j.DailyRollingFileAppender
log4j.appender.remote.Encoding=utf-8
log4j.appender.remote.File=remote.log
log4j.appender.remote.layout=org.apache.log4j.PatternLayout
log4j.appender.remote.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %m%n

log4j.rootLogger=INFO, remote
```
启动服务进程命令如下：
```bash
# java -classpath log4j-1.2.17.jar org.apache.log4j.net.SimpleSocketServer 7890 log4j-server.properties
```

##客户端配置
log4j.properties配置清单如下：
```
log4j.appender.remote=org.apache.log4j.net.SocketAppender
log4j.appender.remote.RemoteHost=192.168.56.2
log4j.appender.remote.Port=7890

log4j.logger.remote=INFO,remote
```

打印日志的代码如下：
```java
org.apache.log4j.Logger logger = org.apache.log4j.Logger.getLogger("remote");
logger.info("I'm a log4j's remote msg!");
```

查看服务端的日志文件remote.log，已经正常打印日志信息：
```
2016-04-03 16:57:28 INFO  Connected to client at /192.168.56.1
2016-04-03 16:57:28 INFO  Starting new socket node.
2016-04-03 16:57:28 INFO  Waiting to accept a new client.
2016-04-03 16:57:29 INFO  I'm a log4j's remote msg!
2016-04-03 16:57:28 INFO  Caught java.net.SocketException closing conneciton.
```

##远程日志的性能
将一个技术引入到生产环境之前，必须要做性能测试或者灰度。

开启8条线程，模拟8台服务器；每条线程每10毫秒打印一条日志信息，模拟每台服务器的每秒打印100条日志的压力。
```java
public class Log4jRemoteBench {
	
	public static void main(String[] args) {
		ScheduledExecutorService service = Executors.newSingleThreadScheduledExecutor();  
		service.scheduleAtFixedRate(new RemoteLogWriter(), 0, 10, TimeUnit.MILLISECONDS);
		service.scheduleAtFixedRate(new RemoteLogWriter(), 0, 10, TimeUnit.MILLISECONDS);
		service.scheduleAtFixedRate(new RemoteLogWriter(), 0, 10, TimeUnit.MILLISECONDS);
		service.scheduleAtFixedRate(new RemoteLogWriter(), 0, 10, TimeUnit.MILLISECONDS);
		service.scheduleAtFixedRate(new RemoteLogWriter(), 0, 10, TimeUnit.MILLISECONDS);
		service.scheduleAtFixedRate(new RemoteLogWriter(), 0, 10, TimeUnit.MILLISECONDS);
		service.scheduleAtFixedRate(new RemoteLogWriter(), 0, 10, TimeUnit.MILLISECONDS);
		service.scheduleAtFixedRate(new RemoteLogWriter(), 0, 10, TimeUnit.MILLISECONDS);
	}
}

class RemoteLogWriter implements Runnable
{
	@Override
	public void run()
	{
		Logger logger = Logger.getLogger("remote");
		logger.info(this + "\tI'm a log4j's remote msg!");
	}
}
```
保持8线程，每线程每秒打印100条日志的压力，稳定运行10分钟无任何出错信息，服务端cpu负载在10%左右。

##配置tomcat使用远程日志
tomcat默认使用java.util.logging打印日志，但是可以通过定制达到使用log4j的目的，具体定制的方式可以参加tomcat官方手册的日志部分：http://tomcat.apache.org/tomcat-8.0-doc/logging.html。
tomcat配置好使用log4j后，就可以参照前面案例，升级成远程日志。

##使用远程日志的建议
* 为了防止远程日志丢失，可以同时打印远程和本地日志
* 关注服务器磁盘空间，最好是有一个cron脚本或配置log4j最大备份文件数
* log4j的服务进程最好有一个死活监测线程，且监测到进程不存在后能自动拉起

