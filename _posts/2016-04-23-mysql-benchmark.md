---
layout: post
title: 
---
purge binary logs to 'mysql-bin.000120';
MySQL的读写能力基准测试

基于当前硬件条件，最慢的就是磁盘，所以一般的业务系统，瓶颈往往是数据库。对数据库的读写能力有一个清晰的认识，在我们设计和评估系统的时候是至关重要的。由于目前MySQL使用的比较多，故针对MySQL进行了读写能力基准测试，使用的工具是mysqlslap。

##前置条件
不同的软硬件平台，测试出的数据也不一样，但是可以作为重要的参考数据。本次测试的软硬件信息如下：
* 操作系统：CentOS release 6.5 (Final)
* CPU：Core(TM) i7-4790K
* 内存：8G
* 硬盘：1T，7200转/分钟
* 数据库：mysql-5.5.50

针对mysql的性能测试，参数的配置对测试结果影响也是至关重要的，本次测试`my.cnf`配置如下：
```
[mysqld]
port            = 3306
socket          = /tmp/mysql.sock
skip-external-locking
key_buffer_size = 16M
max_allowed_packet = 1M
table_open_cache = 64
sort_buffer_size = 512K
net_buffer_length = 8K
read_buffer_size = 256K
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 8M
log-bin=mysql-bin
binlog_format=mixed
server-id       = 1
```

##写能力测试
执行测试的mysqlslap脚本如下：
```bash
./mysqlslap --host=127.0.0.1\
            --port=3306\
            --user=root\
            --password=\
            --auto-generate-sql\
            --auto-generate-sql-add-autoincrement\
            --auto-generate-sql-write-number=10000\
            --auto-generate-sql-unique-write-number=10000\
            --auto-generate-sql-secondary-indexes=2\
            --auto-generate-sql-load-type=write\
            --number-char-cols=10\
            --number-int-cols=10\
            --engine=myisam,innodb\
            --concurrency=10\
            --number-of-queries=10000\
            --iterations=10\
            --debug-info
```
上述脚本的目的，是创建一张表，包含1个自增主键、2列索引、10列int和10列varchar，初始化插入10W条数据。模拟10个客户端，共插入10000条数据。
测试过程中还会调整索引列数和数据量，用来观察对结果的影响。

得到的测试结果如下：
```
Benchmark
        Running for engine myisam
        Average number of seconds to run all queries: 0.678 seconds
        Minimum number of seconds to run all queries: 0.660 seconds
        Maximum number of seconds to run all queries: 0.697 seconds
        Number of clients running queries: 10
        Average number of queries per client: 1000

Benchmark
        Running for engine innodb
        Average number of seconds to run all queries: 3.032 seconds
        Minimum number of seconds to run all queries: 2.905 seconds
        Maximum number of seconds to run all queries: 3.234 seconds
        Number of clients running queries: 10
        Average number of queries per client: 1000
```
可以看出myisam引擎执行10000条插入平均耗时0.678秒，即14750条/秒；innodb引擎执行10000条插入平均耗时3.032秒，即3300条/秒。

最终写性能测试的结果如下表：
   |myisam_1W行|myisam_10W行|myisam_100W行|innodb_1W行|innodb_10W行|innodb_100W行
---|---
无binlog+无索引|20833|20533|20325|5353|5443|5417
有binlog+无索引|17182|16806|16556|3381|3410|3333
有binlog+有索引|12820|13333|12787|3198|3261|3080

##读能力测试
