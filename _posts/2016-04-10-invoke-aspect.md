---
layout: post
title: 使用Spring切面编程监视系统运行状况
---

我们线上业务有一个调用日志很有意思，它会记录每个方法是否成功以及调用耗时，可以用来排查系统里易失败、耗时高的代码块，尤其是涉及到外部接口的情况下，可以作为定位和解决问题的手段，也可以通过成功率、耗时反馈系统整体的运行状态。

这个日志的实现方式不复杂，就是利用Spring的切面功能，下面是剔除公司业务代码逻辑后的完整代码片段：
```java
@Aspect
@Component
public class InvokeAspect 
{
	private static final Log log = LogFactory.getLog("invoke_log");
	
	private static final Random random = new Random();
	
//	@Around(value="execution(* com.demo.spring.controller..*.*(..)) " +
//			"or execution(* com.demo.spring.service..*.*(..)) " +
//			"or execution(* com.demo.spring.repository..*.*(..)) ")
	@Around(value="execution(* com.demo.spring..*.*(..))")
	public Object invoke_log(ProceedingJoinPoint joinPoint) throws Throwable 
	{
		int ret = 0;

		long time = System.currentTimeMillis();
		String localIp = "127.0.0.1";
		String remoteIp = "127.0.0.1";
		String operator = "-";
		String service = "-";
		String method = "-";
		String remark = "success";
		
		StringBuffer sb = null;
		HttpServletRequest request = null;
		
		try 
		{
			RequestAttributes ra = RequestContextHolder.getRequestAttributes();
			if(ra != null) request = ((ServletRequestAttributes)ra).getRequest();
			
			localIp  = request==null?"127.0.0.1":request.getLocalAddr();
			remoteIp = request==null?"127.0.0.1":request.getRemoteAddr();
			operator = "-";
			service  = joinPoint.getTarget().getClass().getSimpleName();
			method   = joinPoint.getSignature().getName();
			
			Object obj = joinPoint.proceed(joinPoint.getArgs());
			return obj;
		} 
		catch (Exception e) 
		{
			ret = -1;
			remark = e.getMessage();
			throw e;
		} 
		finally 
		{
			if(!"healthCheck".equals(method) || random.nextInt(100)<1)
			{
				sb = new StringBuffer();
				sb.append(localIp).append("\t")
				  .append(remoteIp).append("\t")
				  .append(operator).append("\t")
				  .append(service).append("\t")
				  .append(method).append("\t")
				  .append(ret).append("\t")
				  .append(System.currentTimeMillis()-time).append("\t")
				  .append(remark);
				
				if(request != null)
					sb.append("\t").append(request.getServletPath()).append(request.getQueryString()==null?"":"?"+request.getQueryString());
				
				log.info(sb.toString());
			}
		}
	}
}
```

这段代码会打印如下的日志：
```
2016-04-10 11:08:56 INFO  127.0.0.1	127.0.0.1	-	TestController	test	0	3	success	/test
2016-04-10 11:08:56 INFO  127.0.0.1	127.0.0.1	-	TestRepository	hello	0	0	success	/test
2016-04-10 11:08:56 INFO  127.0.0.1	127.0.0.1	-	TestService		hello	0	1	success	/test
```

利用awk和sort可以方便的统计出成功率、平均耗时、最大耗时等信息，辅助我们定位和解决线上系统问题。