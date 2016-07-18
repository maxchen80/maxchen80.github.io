---
layout: post
title: java使用poi导出excel
---
业务软件开发规程中，导出excel是一个非常经典的场景，但凡工作过半年以上的程序员，很少有人没有遇到过导出一个XXX报表这样的需求。使用java导出报表，最常见的是使用apache的poi工具包，api非常清晰简单，参照官方文档和demo，相信一个新手10分钟就可以自己导出一个报表。

但是其中的坑，只有踩过的同学才能明白。前段时间，我们的一个面向用户的业务系统，在导出一个1W行（每行数据十几kb）的excel时，内存溢出宕机了，重启后用户还在一直导出导致系统不停的宕机，最后只好屏蔽该功能后重启才恢复正常，在此鄙视一下我们的系统。

这里引申出一个问题，poi使用的不恰当，在做excel导出的时候很容易内存溢出。为了解决这个问题，我们来重新了解一下excel和poi。

####excel的文件格式
目前excel有两种格式：xls和xlsx。主要区别见下表：
文件 | office版本 | 最大行数 | 最大列数
---- | ----
.xls | 97~2003 | 65536 | 256
.xlsx| 2007及以上 | 1048576 | 16384

####导出xls
引入poi包，有poi和poi-ooxml两个，区别是poi-ooxml包含了XSSF，因为我后面还要测试xlsx的，所以引入的是后者。
```xml
<dependency>
	<groupId>org.apache.poi</groupId>
	<artifactId>poi-ooxml</artifactId>
	<version>3.14</version>
</dependency>
```
测试过程很简单:jvm运行参数是`-Xms128m -Xmx128m`;生成一个每行4列，每列随机256字节字符串，共65536行的.xls文件:
```java
Workbook wb = new HSSFWorkbook();
Sheet sheet = wb.createSheet("sheet001");
for (int i = 0; i < 65536; i++) {
	Row row = sheet.createRow(i);
	row.createCell(0).setCellValue(getRandomString(256));
	row.createCell(1).setCellValue(getRandomString(256));
	row.createCell(2).setCellValue(getRandomString(256));
	row.createCell(3).setCellValue(getRandomString(256));
	System.out.println("write " + i + " line complete!");
}
FileOutputStream fileOut = new FileOutputStream("workbook.xls");
wb.write(fileOut);
fileOut.close();
```
`getRandomString`函数很简单，就是生成指定长度的随机字符串。该程序写到第3.7W行的时候必然报错，异常为`java.lang.OutOfMemoryError: GC overhead limit exceeded`。对应的文章开头我提到的悲催的线上内存溢出导致宕机。

那么如何解决呢？
如果一定要使用.xls格式，说实话我没找到，也在继续尝试寻找。
我尝试过每xxx行提交、再追加写的方式，仍然不行。参考其他网站的方案，写XML然后使用poi提供的示例转化成excel，我没有验证过。
如果能够接受xlsx格式，也就是意味着用户安装的office2007或以上版本，那么问题就容易了。

####导出xlsx
前面提到过，xlsx最大行数为1048576，那么这次我们换成xlsx格式，导出1048576行：
```java
Workbook wb = new XSSFWorkbook();
Sheet sheet = wb.createSheet("sheet001");
for (int i = 0; i < 1048576; i++) {
	Row row = sheet.createRow(i);
	row.createCell(0).setCellValue(getRandomString(256));
	row.createCell(1).setCellValue(getRandomString(256));
	row.createCell(2).setCellValue(getRandomString(256));
	row.createCell(3).setCellValue(getRandomString(256));
	System.out.println("write " + i + " line complete!");
}
FileOutputStream fileOut = new FileOutputStream("workbook.xlsx");
wb.write(fileOut);
fileOut.close();
```
运行示例代码，大约至1.3W行的时候，非常不幸,`java.lang.OutOfMemoryError: GC overhead limit exceeded`内存溢出了。

查阅官网，[SXSSF](http://poi.apache.org/spreadsheet/how-to.html#sxssf)是XSSF在生产大型表格和内存空间有限的时候的一个延伸，SXSSF通过访问限制，实现了极低的内存占用。
修改`Workbook wb = new SXSSFWorkbook(100);`，再次执行上面的代码，顺利的生成了这个百万行的excel，且使用jvisualvm监视内存占用，不超过20MB，问题解决。

####导入xls和xlsx
poi导入大型excel一样存在溢出的风险，官方文档有给出坚决方案，即[Event API (HSSF Only)](http://poi.apache.org/spreadsheet/how-to.html#event_api)和[xssf_sax_api](http://poi.apache.org/spreadsheet/how-to.html#xssf_sax_api)。

####超过104W行的报表
超过104W行的报表怎么处理？这个已经超过excel和poi的范畴了，答案是csv。
> 逗号分隔值（Comma-Separated Values，CSV，有时也称为字符分隔值，因为分隔字符也可以不是逗号），其文件以纯文本形式存储表格数据（数字和文本）。纯文本意味着该文件是一个字符序列，不含必须像二进制数字那样被解读的数据。 ----[百度百科](http://baike.baidu.com/subview/468993/5926031.htm)

```java
FileWriter writer = new FileWriter(new File("test.csv"), false);
BufferedWriter bw = new BufferedWriter(writer);
for(int i=0;i<5000000;i++)
{
	StringBuffer sb = new StringBuffer(getRandomString(256)).append(",")
			.append(getRandomString(256)).append(",")
			.append(getRandomString(256)).append(",")
			.append(getRandomString(256)).append(System.getProperty("line.separator"));
	bw.write(sb.toString());
	System.out.println("write " + i + " line complete!");
}
bw.close();
writer.close();
```
上面的代码顺利生成500W行的csv文件，监视内存使用在5~40MB之间波动。

####总结
* 使用poi导出报表，一定要评估excel大小。
* 超过1MB的报表（1KB/行*1000行），一定使用SXSSF方式，因为要考虑多个用户同时执行操作。
* 如果报表仅仅为纯文本，可以考虑使用csv来代替。



