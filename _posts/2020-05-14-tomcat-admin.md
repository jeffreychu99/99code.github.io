---
layout: post
title: Tomcat管理页面登陆方法
categories: Java
description: Tomcat管理页面登陆方法
keywords: tomcat
---

Tomcat管理页面可以查看Tomcat的运行信息，然而通常管理页面时需要用户登陆，很多刚接触的人不了解用户名和密码 是多少，如何修改。。

这里介绍一种方法。

![](/images/posts/tomcat-admin/1.png)

实际上点击取消，Tomcat本身也是有提示的，对于角色的描述很详细。

![](/images/posts/tomcat-admin/3.png)

修改方式也很简单

```xml
<!--
  <role rolename="tomcat"/>
  <role rolename="role1"/>
  <user username="tomcat" password="tomcat" roles="tomcat"/>
  <user username="both" password="tomcat" roles="tomcat,role1"/>
  <user username="role1" password="tomcat" roles="role1"/>
-->
  <user username="admin" password="admin" roles="manager-gui"/>
```

这样，这可以通过admin/admin访问管理状态页面。

![](/images/posts/tomcat-admin/2.png)