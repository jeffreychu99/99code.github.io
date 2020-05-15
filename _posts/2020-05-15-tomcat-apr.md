---
layout: post
title: Tomcat apr模式开启方法
categories: Java
description: Tomcat apr模式开启方法
keywords: tomcat
---

apr(Apache Portable Runtime/Apache可移植运行时)，是Apache HTTP服务器的支持库。你可以简单地理解为，Tomcat将以JNI的形式调用Apache HTTP服务器的核心动态链接库来处理文件读取或网络传输操作，从而大大地提高Tomcat对静态文件的处理性能。 Tomcat apr也是在Tomcat上运行高并发应用的首选模式。

Tomcat apr运行模式的配置相对比较麻烦。据官方文档所述，Tomcat apr需要以下三个组件的支持：

1、APR library[APR库]

2、JNI wrappers for APR used by Tomcat (libtcnative)[简单地说，如果是在Windows操作系统上，就是一个名为tcnative-1.dll的动态链接库文件]

3、OpenSSL libraries[OpenSSL库]


此外，与配置nio运行模式一样，也需要将对应的Connector节点的protocol属性值改为org.apache.coyote.http11.Http11AprProtocol。不过上述只是在较早的版本才需要配置的，新的版本，如果9.0，默认已经是apr模式，建议用户Tomcat最新的版本。

![](/images/posts/tomcat-apr/1.png)

在windows上，apr模式的开启依赖tcnative-1.dll动态库，在9.0版本中已经包含了，其它如果不存在，需要自己下载配置。

基本步骤：

1、从http://archive.apache.org/dist/tomcat/tomcat-connectors/native/网址下载；

2、将tcnative-1.dll拷贝tomcat的bin目录；

3、启动如果出现下面内容，说明apr加载成功。

![](/images/posts/tomcat-apr/2.png)