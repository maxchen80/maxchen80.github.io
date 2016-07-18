---
layout: post
title: 页面模板引擎（jsp/freemarker/velocity）性能对比
---

目前国内java web项目开发，常用的页面模板引擎是jsp+jstl、freemarker和velocity。关于谁优谁劣的问题网上讨论的也比较多，但是很多都是无数据的空谈，或者重点比较语法、标签特性，但是作为业务开发而言，真正需要关心的是谁最简单、最高效，下面构建一个基于spring mvc的测试demo，比较三者的性能和语法特性。

##前置条件
测试服务器的配置及环境清单：
```
CentOS release 6.5 (Final)
Intel(R) Core(TM) i7-4790K CPU @ 4.00GHz
Java(TM) SE Runtime Environment (build 1.8.0_66-b17)
Apache Tomcat/8.0.32
```

测试模板的版本清单：
```xml
<dependency>
	<groupId>jstl</groupId>
	<artifactId>jstl</artifactId>
	<version>1.2</version>
</dependency>
<dependency>
	<groupId>org.freemarker</groupId>
	<artifactId>freemarker</artifactId>
	<version>2.3.23</version>
</dependency>
<dependency>
	<groupId>org.apache.velocity</groupId>
	<artifactId>velocity</artifactId>
	<version>1.7</version>
</dependency>
```

##核心代码清单
测试过程是模拟一个简单的SpringMVC的web项目，后台随机的生成10个对象的集合，前端页面将这10个对象展示在表格里面，会使用的循环、流程控制和格式化标签，工程同时支持jsp、freemarker和velocity共3种视图解析。

SpringMVC配置多视图解析配置：
```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	<property name="prefix" value="/WEB-INF/jsp" />
	<property name="suffix" value="" />
	<property name="viewNames" value="*.jsp" />
</bean>

<bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
	<property name="templateLoaderPath" value="/WEB-INF/jsp" />
	<property name="defaultEncoding" value="UTF-8" />
</bean>
<bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
	<property name="prefix" value="" />
	<property name="suffix" value="" />
	<property name="viewNames" value="*.ftl" />
	<property name="contentType" value="text/html;charset=UTF-8"></property>
</bean>

<bean class="org.springframework.web.servlet.view.velocity.VelocityConfigurer">
	<property name="resourceLoaderPath" value="/WEB-INF/jsp" />
	<property name="velocityProperties">
		<props>
			<prop key="input.encoding">UTF-8</prop>
			<prop key="output.encoding">UTF-8</prop>
		</props>
	</property>
</bean>
<bean class="org.springframework.web.servlet.view.velocity.VelocityViewResolver">
	<property name="prefix" value="" />
	<property name="suffix" value="" />
	<property name="viewNames" value="*.vm" />
	<property name="contentType" value="text/html;charset=UTF-8"></property>
</bean>
```

控制器代码主要是随机生产10个对象的集合，返回给前端页面，代码清单如下：
```java
@Controller
@RequestMapping("/template")
public class TemplateController {
	
	@RequestMapping("/{val}")
	public String bench(@PathVariable(value = "val") String val, Model model) {
		Random ra =new Random();
		List<UserInfo> list = Lists.newArrayList();
		for(int i=0;i<10;i++)
		{
			UserInfo userInfo = new UserInfo();
			userInfo.setId(ra.nextInt(10));
			userInfo.setSex(ra.nextInt(2));
			userInfo.setName("maxchen_"+userInfo.getId());
			userInfo.setEmail(userInfo.getName()+"@mail.com");
			userInfo.setBirthday(new Date());
			userInfo.setRegister(new Date());
			list.add(userInfo);
		}
		model.addAttribute("list", list);
		model.addAttribute("df", new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
		return "/template."+val;
	}
}
```

模板页面负责循环读取控制器生产的集合，将10个对象以表格的形式呈现出来，其中会用到循环、流程控制和格式化标签，这个页面属于繁简适中，很符合实际业务场景。

template.jsp代码清单如下：
```
<html lang="zh-CN">
<body>
<table>
	<c:forEach items="${list}" var="e">
		<tr>
		 	<td>${e.id}</td>
		 	<td>${e.name}</td>
		 	<td><c:if test="${e.sex==0}">男</c:if><c:if test="${e.sex==1}">女</c:if></td>
		 	<td>${e.email}</td>
		 	<td><fmt:formatDate value="${e.birthday}" pattern="yyyy-MM-dd"/></td>
		 	<td><fmt:formatDate value="${e.register}" pattern="yyyy-MM-dd HH:mm:ss"/></td>
		</tr>
	</c:forEach>
</table>
</body>
</html>
```

template.ftl代码清单如下：
```
<html lang="zh-CN">
<body>
<table>
	<#list list as e>
		<tr>
		 	<td>${e.id}</td>
		 	<td>${e.name}</td>
		 	<td><#if e.sex==0>男<#else>女</#if></td>
		 	<td>${e.email}</td>
		 	<td>${e.birthday?string("yyyy-MM-dd")}</td>
		 	<td>${e.register?string("yyyy-MM-dd HH:mm:ss")}</td>
		</tr>
	</#list>
</table>
</body>
</html>
```

template.vm代码清单如下：
```
<html lang="zh-CN">
<body>
<table>
	<#list list as e>
		<tr>
		 	<td>${e.id}</td>
		 	<td>${e.name}</td>
		 	<td><#if e.sex==0>男<#else>女</#if></td>
		 	<td>${e.email}</td>
		 	<td>${e.birthday?string("yyyy-MM-dd")}</td>
		 	<td>${e.register?string("yyyy-MM-dd HH:mm:ss")}</td>
		</tr>
	</#list>
</table>
</body>
</html>
```

##测试过程及结果
先进行预热，手工访问若干次测试url后，对每个视图解析地址使用ab并发100个线程共10000个请求，连续10次进行测试取平均值：
```bash
# ab -c 100 -n 10000 http://192.168.56.2:8080/demo-webapp/template/jsp
# ab -c 100 -n 10000 http://192.168.56.2:8080/demo-webapp/template/ftl
# ab -c 100 -n 10000 http://192.168.56.2:8080/demo-webapp/template/vm
```
最终测试耗时结果如下
模板|耗时(秒)
---|---
jsp+jstl|1.62
freemarker|0.737
velocity|0.729

对测试结果的分析和结论：
* freemarker和velocity的性能基本持平，jsp+jstl相对落后，具体到每个请求差距约9ms。
* 9ms相对于业务的处理和网络传输的耗时而言，具体占比需要根据实际情况核算；如果占比较大可以考虑废弃jsp，否则模板引擎不是影响系统性能的关键因素。
* velocity的格式化使用体验不是很好，且该项目自2010年起已未更新，不推荐使用。