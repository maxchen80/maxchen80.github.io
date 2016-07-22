---
layout: post
title: 
---

mysqlslap -- 负载仿真客户端
mysqlslap是MySQL官方提供的一个基准测试工具，用来为MySQL服务器模拟客户端负载，并报告每个阶段用时的诊断程序。它的工作原理就是模拟多个客户端访问服务器。
调用mysqlslap是这样的：
```bash
# ./mysqlslap [options]
```
一些选项，如--create或--query，使你能够指定一个包含SQL的字符串或文件。如果你指定了一个文件，默认情况下它必须每行包含一个sql语句（也就是说，隐含的语句分隔符是换行符）。使用--delimiter选项可以指定不同的分隔符，这使得你可以在指定跨越多行的语句或将多个语句置于一行。你不能在文件里包含注释，**mysqlslap**无法理解它们。

**mysqlslap**的运行过程分为3个阶段：
* 创建schema、table，和可选的任何用于测试的存储过程或数据。这个阶段使用一个客户端连接。
* 运行负载测试。这个阶段可以使用多个客户端连接
* 清理（断开链接，如果指定了的话也会删除table）。这个阶段使用一个客户端连接。

下面通过3个示例，显示一下mysqlslap的使用方式：

1. 提供你自己的创建和查询sql，使用50个客户端，每个客户端执行200个select查询：
```bash
# ./mysqlslap --delimiter=";" --create="CREATE TABLE a (b int);INSERT INTO a VALUES (23)" --query="SELECT * FROM a" --concurrency=50 --iterations=200
```

2. 让mysqlslap在有2个INT列和3个VARCHAR列的表上构建查询语句，使用5个客户端，每个客户端查询20次。不要创建表或插入数据（即使用前面的测试schema和数据）：
```bash
# ./mysqlslap --concurrency=5 --iterations=20 --number-int-cols=2 --number-char-cols=3 --auto-generate-sql
```

3. 让mysqlslap从指定的文件里加载创建、插入、和查询的SQL语句，create.sql文件拥有使用';'分割的多表创建语句和新增语句，query.sql文件拥有使用';'分割的多个查询语句。运行所有的负载语句，使用5个客户端运行query.sql文件里的所有查询语句（每个客户端5次）。
```bash
# ./mysqlslap --concurrency=5 --iterations=5 --query=query.sql --create=create.sql --delimiter=";"
```

下面是**mysqlslap**支持部分常用选项，完整选项请参考`./mysqlslap --help`或查阅[官方文档](http://dev.mysql.com/doc/refman/5.5/en/mysqlslap.html)。

* --auto-generate-sql, -a  
自动生产SQL语句，在使用文件和命令选项时不支持。
* --auto-generate-sql-add-autoincrement  
自动生成的表中，添加一个auto_increment列。
* --auto-generate-sql-execute-number=N  
指定自动生成的查询语句数量。
* --auto-generate-sql-guid-primary  
自动生产的表中，添加一个基于GUID的主键。
* --auto-generate-sql-load-type=type  
指定负载测试的类型。允许的值包括read（扫描表），write（插入表）、key（读取主键）、update（更新主键）、mix（一半插入、一半扫描）。默认选项是mix。
* --auto-generate-sql-unique-query-number=N  
为自动测试生成多少个不同的查询。例如，你想在key test里构建1000个select，你可以设置该项的值为1000来运行1000个不同的查询语句，默认值是10。
* --auto-generate-sql-write-number=N  
执行多少行插入。默认100。
* --compress, -C  
如果服务端和客户端支持压缩的话则压缩它们直接发送的所有信息。
* --concurrency=N, -c N  
模拟的并发客户端数量。
* --create=value  
包含建表语句的文件或字符串
* --create-schema=value  
运行测试的schema。注意：如果同时也指定了--auto-generate-sql选项，mysqlslap在测试完成后会删除schema。为避免该情况，可使用--no-drop选项。
* --debug-info, -T  
程序退出时打印调式信息、内存和CPU使用率。
* --delimiter=str, -F str  
在使用文件或命令行选项时SQL语句提供的分隔符。
* --engine=engine_name, -e engine_name  
用户创建表的存储引擎。
* --host=host_name, -h host_name  
连接到指定的MySQL服务器。
* --iterations=N, -i N  
运行测试的次数。
* --no-drop  
防止mysqlslap在测试过程中删除它创建的schema。
* --number-char-cols=N, -x N  
如果指定了--auto-generate-sql的话，表示VARCHAR的列数。
* --number-int-cols=N, -y N  
如果指定了--auto-generate-sql的话，表示INT的列数。
* --only-print  
不连接到数据库。mysqlslap仅仅只打印它的执行内容。
* --password[=password], -p[password]  
连接到服务器的密码。
* --port=port_num, -P port_num  
连接到服务器的端口。
* --query=value, -q value  
包含select语句的文件或字符串。
* --user=user_name, -u user_name  
连接到服务器时使用的MySQL用户名。
* --verbose, -v  
详细模式。打印出程序执行的更多信息。这个选项可以多次使用，以增加信息量。








