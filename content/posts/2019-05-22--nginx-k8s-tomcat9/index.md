---
title: SpringBoot框架升级2.1.2导致的请求400异常
subTitle: springBoot
category: java
cover: java.png
---



​	近期参与公司SpringBoot框架升级,由1.5.3版本升级到2.1.2版本,遇到了一个问题,记录下来.

​	***问题描述:***

​		Springboot升级到2.1.2版本后,从外网发送请求返回400异常,控制台报错如下:

2019-05-21 11:19:36,364:INFO http-nio-9060-exec-10 (DirectJDKLog.java:175) - The host [enterprise-zuul-api.pro.svc.cluster.local,enterprise-zuul-api.pro.svc.cluster.local] is not valid
 Note: further occurrences of request parsing errors will be logged at DEBUG level.
java.lang.IllegalArgumentException: The character [,] is never valid in a domain name.
	at org.apache.tomcat.util.http.parser.HttpParser$DomainParseState.next(HttpParser.java:926) ~[tomcat-embed-core-9.0.14.jar!/:9.0.14]
	at org.apache.tomcat.util.http.parser.HttpParser.readHostDomainName(HttpParser.java:822) ~[tomcat-embed-core-9.0.14.jar!/:9.0.14]
	at org.apache.tomcat.util.http.parser.Host.parse(Host.java:71) ~[tomcat-embed-core-9.0.14.jar!/:9.0.14]
	at org.apache.tomcat.util.http.parser.Host.parse(Host.java:45) ~[tomcat-embed-core-9.0.14.jar!/:9.0.14]
	at org.apache.coyote.AbstractProcessor.parseHost(AbstractProcessor.java:288) [tomcat-embed-core-9.0.14.jar!/:9.0.14]
	at org.apache.coyote.http11.Http11Processor.prepareRequest(Http11Processor.java:809) [tomcat-embed-core-9.0.14.jar!/:9.0.14]
	at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:384) [tomcat-embed-core-9.0.14.jar!/:9.0.14]
	at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:66) [tomcat-embed-core-9.0.14.jar!/:9.0.14]
	at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:834) [tomcat-embed-core-9.0.14.jar!/:9.0.14]
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1417) [tomcat-embed-core-9.0.14.jar!/:9.0.14]
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49) [tomcat-embed-core-9.0.14.jar!/:9.0.14]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [?:1.8.0_171]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [?:1.8.0_171]
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) [tomcat-embed-core-9.0.14.jar!/:9.0.14]
	at java.lang.Thread.run(Thread.java:748) [?:1.8.0_171]



解决思路:

​	java.lang.IllegalArgumentException: The character [,] is never valid in a domain name 错误信息指出是由host中包含逗号引起的错误, 查阅资料说 tomcat高版本对特殊字符有更严格的校验 , 仔细看错误信息,The host [enterprise-zuul-api.pro.svc.cluster.local,enterprise-zuul-api.pro.svc.cluster.local] is not valid 发现host中包含了两个host地址,由逗号分隔判断并不是特殊字符引起.

​	下载了boot1.5.3和2.1.2对应的内置tomcat jar包,分别为tomcat-embed-core-8.5.14.jar 和 tomcat-embed-core-9.0.14.jar,对比了源码发现,8.5版本对host并没有做校验和操作,但是9版本对host做了操作,上述错误信息也可以看出,tomcat9对host进行了parse操作,发现包含逗号,所以抛出了java.lang.IllegalArgumentException异常,层层上抛后在Http11Processor中捕获该异常,设置400响应码.

​	问题转为host为何会有两个值, 对于ng并不熟悉,所以查阅了一些资料,看了一下ng的基本概念,发现了如下信息:

```
X-Forwarded-For 是一个HTTP拓展头，起初在 RFC2616 (HTTP/1.1) 中并未定义，但后来被广泛用于表示客户端真实IP。而后 RFC7239 (Forwarded HTTP Extension) 中又提供了标准的 Forwarded 头，使用 X-Forwarded-For 来提取真实IP也就成了事实上的标准。

X-Forwarded-For 存储了客户端IP以及请求链路上各代理IP，假设请求依次通过 proxy1、proxy2 后抵达服务，那 X-Forwarded-For 的值为：客户端IP, proxy1 IP, proxy2 IP，IP之间以逗号隔开。

```

跟公司同事讨论后得知,公司服务在外部请求进来后会先经过ng,再走zuul,再经过ng发送到k8s中,所以X-Forwarded-For中会保存多个地址,而k8s中ingress会获取X-Forwarded-For作为host传入项目,引起上述问题.



![1558513485533](/1.jpg)