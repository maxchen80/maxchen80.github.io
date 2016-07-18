---
layout: post
title: 使用groovy定位和解决线上业务问题的方案
---
文章开始前，首先重点声明：本方案并非本人创意，而是学习的一位前同事的创意，重点是觉得这个方案非常简单、有效，所以有必要记录下来和大家分享。当然出于公司信息保密和安全的目的，文章的代码均重写，分享方案的同时也要保证公司的利益嘛。

大部分程序员的日常工作，无非就是实现新功能和维护老功能。程序员不是神，自然就会犯错。异常分别系统级别和业务级别，系统级别的异常如内存溢出、进程假死等比较高级，自然也就稀罕了呵呵。而更加常见和需要我们去解决的是业务异常。

什么叫做业务异常？来看下几个比较常见的情景：
* 页面上用户名显示不出来，
* ss
* 撒撒撒

这些类似的场景，在程序员的工作时间里经常出现。我们会猜测是某些组件、接口、缓存等引起，但是如何来验证呢？这个时候就考验日常编码习惯和经验了：是否打印了详细的日志，而日志太详细其实也是一把双刃剑，磁盘空间要求太高。

针对java环境，有没有一种有效的办法来定位和解决这些业务异常问题呢？设想一下，我现在怀疑某接口abc()返回的数据不对，那么能够打印和看到该接口的返回值，就可以迅速验证我的猜测了。如何在不上线的情况下打印该接口线上返回值呢？答案之一就是使用groovy。

groovy语言的介绍可以自己网上搜索，使用groovy进行java系统的线上业务问题定位主要是看重其两个重要特性：
* 运行在jvm上，无缝使用java的类库
* 它是动态语言，或叫脚本语言，无需手工编译

看看，具备了这两个特性，我们就可以上传groovy编写的代码，调用和打印线上相关java对象，达到验证我们的各种猜测的目的。

首先创建一个html页面：包含一个textarea，用于编写groovy代码；一个button，用于提交我们的代码；一个p节点，用于显示执行结果。
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<script src="http://cdn.bootcss.com/jquery/3.0.0/jquery.min.js"></script>
<title>groovy</title>
</head>
<body>
	<textarea placeholder="请在此处编写groovy代码" rows="20" cols="100"></textarea>
	<br><button>excute</button>
	<p></p>
</body>

<script type="text/javascript">
	$("button").click(function() {
		$.get("?", {code : $("textarea").val()}, function(data) {
			$("p").html(data);
		});
	});
</script>
</html>
```

页面创建完成之后，需要有一个servlet来接收并执行和返回页面提交groovy代码：
```java
public class GroovyServlet extends HttpServlet {
	
	static GroovyShell shell;
	
	@Override
	public void init() throws ServletException {
		GroovyClassLoader parent = new GroovyClassLoader(this.getClass().getClassLoader());
		Binding binding = new Binding();
		CompilerConfiguration config = new CompilerConfiguration();
		shell = new GroovyShell(parent, binding, config);
	}

	@Override
	public void service(ServletRequest req, ServletResponse res)
			throws ServletException, IOException {
		if(!Strings.isNullOrEmpty(req.getParameter("code")))
		{
			String code = req.getParameter("code");
			System.out.println("==========source code:"+System.getProperty("line.separator")+code);
			Object obj = null;
			try{
				System.out.println("==========excute code:");
				obj = shell.evaluate(code);
			}catch (Exception e){
				obj = e;
			}
			System.out.println("==========return json:"+System.getProperty("line.separator")+obj);
			res.getOutputStream().write(new ObjectMapper().writeValueAsString(obj).getBytes());
			res.getOutputStream().close();
		}
		else
		{
			InputStream is = this.getClass().getResourceAsStream("groovy.html");
			ServletOutputStream os = res.getOutputStream();
			IOUtils.copy(is, os);
			is.close();
			os.close();
		}
	}
}
```
这段代码会将页面提交的groovy代码执行，并将执行结果返回的结果，使用jackson序列化成json字符串，返回给页面。
最后在web.xml进行配置，如下：
```xml
<servlet>
	<servlet-name>groovy</servlet-name>
	<servlet-class>demo.groovy.GroovyServlet</servlet-class>
</servlet>
<servlet-mapping>
	<servlet-name>groovy</servlet-name>
	<url-pattern>/groovy</url-pattern>
</servlet-mapping>
```
最后访问我们的页面，提交代码并执行，可以看到结果如下：
```
==========source code:
println demo.spring.controller.TestController.i
demo.spring.controller.TestController.i+=10
println demo.spring.controller.TestController.i
==========excute code:
20
30
```
至此，我们通过这个servlet已经可以将groovy代码提交到服务器并在执行，也可以访问和修改服务器上的数据，已经可以达到快速验证自己的猜测（使用groovy读服务器数据）和临时解决问题（使用groovy写服务器数据）。

###高级一点的应用
我们线上很对系统都是使用Spring开发的，大量的对象都是启动的时候初始化的，如何使用该servlet快读读取线上spring bean的状态呢，一种方式就是在初始化GroovyShell的时候，将这个spring bean绑定到Binding上面，后续直接使用：
```java
Binding binding = new Binding();
WebApplicationContext wac = ContextLoader.getCurrentWebApplicationContext();
for (String name:wac.getBeanDefinitionNames()) 
	binding.setVariable(name, wac.getBean(name));
CompilerConfiguration config = new CompilerConfiguration();
config.setScriptBaseClass("demo.groovy.ScriptExtend");
```
demo.groovy.ScriptExtend的代码清单如下：
```java
public class ScriptExtend extends Script {

	@Override
	public Object run() {
		return null;
	}
	
	public Object ls(){
		StringBuffer sb = new StringBuffer();
		for(Object obj:getBinding().getVariables().keySet())
			sb.append(obj).append("<BR>");
		return sb.toString();
	}
}
```
如下图所示，现在我们已经可以直接调用线上已实例化的对象和方法了。
```
==========source code:
testServiceImpl.hello('maxchen')
==========excute code:
==========return json:
hello maxchen
```
###关于安全
关于将代码提交到线上环境并直接访问线上环境数据，最大的问题就是安全性了，尤其是该提交入口在公网可以访问，一旦被别有用心的人利用将是致命级的bug。关于如何避免被恶意利用该功能，我们目前的方案是该代码仅仅在预发布环境（与线上完全一致，但仅仅内网可访问）存在，如果没有这样的环境，应该也不难，可以使用基于账密、证书等方式，这个不是本片文章的核心内容就不具体讨论了。