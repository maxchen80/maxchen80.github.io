---
layout: post
title: apache bench
---
[ab](https://httpd.apache.org/docs/2.4/programs/ab.html)是一个为Apache Http服务器提供基准测试的工具。它能快速的让你对当前安装完成Apache有一个印象，显示每秒钟能够处理多少请求。


###安装ab
```bash
# yum -y install httpd-tools
```

###ab的输入说明
```bash
# ab -h
Usage: ab [options] [http[s]://]hostname[:port]/path
Options are:
    -n requests     Number of requests to perform
    -c concurrency  Number of multiple requests to make
    -t timelimit    Seconds to max. wait for responses
    -b windowsize   Size of TCP send/receive buffer, in bytes
    -p postfile     File containing data to POST. Remember also to set -T
    -u putfile      File containing data to PUT. Remember also to set -T
    -T content-type Content-type header for POSTing, eg.
                    'application/x-www-form-urlencoded'
                    Default is 'text/plain'
    -v verbosity    How much troubleshooting info to print
    -w              Print out results in HTML tables
    -i              Use HEAD instead of GET
    -x attributes   String to insert as table attributes
    -y attributes   String to insert as tr attributes
    -z attributes   String to insert as td or th attributes
    -C attribute    Add cookie, eg. 'Apache=1234. (repeatable)
    -H attribute    Add Arbitrary header line, eg. 'Accept-Encoding: gzip'
                    Inserted after all normal header lines. (repeatable)
    -A attribute    Add Basic WWW Authentication, the attributes
                    are a colon separated username and password.
    -P attribute    Add Basic Proxy Authentication, the attributes
                    are a colon separated username and password.
    -X proxy:port   Proxyserver and port number to use
    -V              Print version number and exit
    -k              Use HTTP KeepAlive feature
    -d              Do not show percentiles served table.
    -S              Do not show confidence estimators and warnings.
    -g filename     Output collected data to gnuplot format file.
    -e filename     Output CSV file with percentages served
    -r              Don't exit on socket receive errors.
    -h              Display usage information (this message)
    -Z ciphersuite  Specify SSL/TLS cipher suite (See openssl ciphers)
    -f protocol     Specify SSL/TLS protocol (SSL2, SSL3, TLS1, or ALL)
```
###ab的输出说明
```bash
# ab -n 100 -c 10 http://192.168.56.2:8080/test.jsp
This is ApacheBench, Version 2.3 <$Revision: 655654 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 192.168.56.2 (be patient).....done


Server Software:        Apache-Coyote/1.1
Server Hostname:        192.168.56.2
Server Port:            8080

Document Path:          /test.jsp
Document Length:        14 bytes

Concurrency Level:      10
Time taken for tests:   0.055 seconds
Complete requests:      100
Failed requests:        0
Write errors:           0
Total transferred:      25755 bytes
HTML transferred:       1414 bytes
Requests per second:    1833.82 [#/sec] (mean)
Time per request:       5.453 [ms] (mean)
Time per request:       0.545 [ms] (mean, across all concurrent requests)
Transfer rate:          461.23 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    1   1.0      0       6
Processing:     1    4   4.2      3      19
Waiting:        1    4   3.1      3      17
Total:          1    5   4.8      3      23
WARNING: The median and mean for the initial connection time are not within a normal deviation
        These results are probably not that reliable.

Percentage of the requests served within a certain time (ms)
  50%      3
  66%      4
  75%      5
  80%      5
  90%     14
  95%     19
  98%     20
  99%     23
 100%     23 (longest request)
```
下面的列表描述了ab返回的值：
**Concurrency Level**
在测试过程中使用的并发客户端的数量。
**Time taken for tests**
从第一个套接字连接到最后一个响应被接收时所采取的时间。
**Complete requests**
收到成功响应的数量。
**Failed requests**
被视为失败的请求数。如果该数字大于零，则另一行将打印显示由于连接、读取、不正确的内容长度或异常而导致的请求数。
**Write errors**
写入（broken pipe）时失败的错误数。
**Total transferred**
从服务器接收的字节总数。这个数字基本上是发送过的字节的字节数。
**HTML transferred**
从服务器接收到的文档字节总数。这个数字不包括在HTTP头接收的字节数。
**Requests per second**
这是每秒请求数。这个值是由所用的总时间除以请求的数目的结果。
**Time per request**
每个请求的平均花费的时间。
第一个值是用公式concurrency*timetaken*1000/done；
第二个值是用公式timetaken*1000/done；
**Transfer rate**
传输速率的计算公式totalread/1024/timetaken。

###ab模拟已登陆用户测试
ab模拟用户登陆，主要是使用cookie的方式。在chrome浏览器下，设置》内容设置》所有cookie和网站数据里，搜索域名，可以查看到cookie信息。
只需要使用单个cookie的情况下,可以用如下第一种方式测试，如果有多个cookie，则可以使用http header的方式，多个cookie直接使用分号间隔：
```bash
ab -n 100 -c 10 -C JSESSIONID=E3D80E76DBC5EF6FEAC7C82BAB3ACF4C http://192.168.56.2:8080/test.jsp?p=profile
ab -n 100 -c 10 -H "Cookie:JSESSIONID=E3D80E76DBC5EF6FEAC7C82BAB3ACF4C" http://192.168.56.2:8080/test.jsp?p=profile
```


