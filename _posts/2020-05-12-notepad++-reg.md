---
layout: post
title: Notepad++正则表达式分组替换方法
categories: 工具
description: Notepad++正则表达式分组替换方法
keywords: Notepad++, 正则表达式
---

# 背景

将Mongo数据库的数据导出（导出格式为JSON），使用Node.js进行一些批量操作，可是Mongo中像注册时间等是这样的格式"activetime" : NumberLong("148487595004")，在解析为JSON的时候会出错。

因此，想要实现这样一种要求：

将NumberLong("xxxxx") 转化为 “xxxxx"  

# 解决思路

本文介绍一种使用Notepad++进行正则表达式替换方法

![](/images/posts/notepad++-reg/1.png)

查找目标：NumberLong\((".*[^"]")\)

(".*[^"]")匹配NumberLong("xxxx")对应的“xxx”部分，需要使用括号括起来，主要用于分组筛选，其中.*匹配所有字符，[^"]表示非"字符开始。


替换为：\1

\1表示查找目标找到的分组。


替换结果：

![](/images/posts/notepad++-reg/2.png)