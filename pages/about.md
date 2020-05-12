---
layout: page
title: About
description: 打码改变世界
keywords: Zhuang Ma, 马壮
comments: true
menu: 关于
permalink: /about/
---

十年一线软件产品研发，丰富的经验，精通C/C++、Java、Python，擅长工具类软件开发。从事过嵌入式系统开发，Windows应用开发，安卓编程，服务器开发等。现就职于国内知名通信公司。著有《基于Flask实现后台权限管理系统》、《Wireshark插件开发 - 实现自定义协议解析 》、《Node.js开发中常见的那些坑》等电子书，上线百度阅读。

## 联系

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}
{% if site.url contains '99code.cc' %}
<li>
<!-- 微信公众号：<br />
<img style="height:192px;width:192px;border:1px solid lightgrey;" src="{{ assets_base_url }}/assets/images/qrcode.jpg" alt="闷骚的程序员" /> -->
</li>
{% endif %}
</ul>


## Skill Keywords

{% for skill in site.data.skills %}
### {{ skill.name }}
<div class="btn-inline">
{% for keyword in skill.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
