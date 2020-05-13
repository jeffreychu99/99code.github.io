---
layout: post
title: Vue.js动态组件的应用
categories: Vue.js
description: 介绍Vue.js如何动态创建组件并获取组件的属性
keywords: Vue.js, component
---

# 概述
很多场景下，我们需要根据后台的数据动态创建页面内容。这里我们通常的解决思路是使用v-for指令，遍历数据生成页面内容。如下所示：

```javascript
<form method="post">
    <div v-for='item in data' :key='item.name'>
    <input type='text' name='name' v-model='item.name' />
    <input type='text' name='age' v-model='item.age' />
    </div>
    <input type="submit" value="提交" />
</form>
```

提交表单数据如下图所示

![](/images/posts/vue-dynamic-component/post.png)

可以看到，提交表单的键值有重复。命名方面不太好管理，当然也可以利用一下hack技巧，譬如根据循环下标等，动态设置控件的name。

```javascript
<form method="post">
    <div v-for='(item, index) in data' :key='item.name'>
    <input type='text' :name="'name-' + index" v-model='item.name' />
    <input type='text' :name="'age-' + index" v-model='item.age' />
    </div>
    <input type="submit" value="提交" />
</form>
```

提交表单数据如下图所示

![](/images/posts/vue-dynamic-component/post2.png)

虽然键值名称改变了，然而这类数据提交到后台，后台解析起来非常不方便。

这里前端也可以在提交之前对数据进行一些改造，根据javascript的动态属性原理，将数据整理后再提交后台。

```javascript
[{name: jack, age: 10}, {name: lucy, age: 11}]
```

这样处理后，提交到后台的数据非常清晰。不管如何处理，这里处理动态创建的页面都很不方便，需要绕一圈才能解决问题，本文提出一种方法，基于vue.js动态组件的方式来解决问题，通用性强，利用组件数据封装和隔离性，可以很好解决这类问题。

# 解决思路

## vue-cli创建脚手架
vue create hello-vue

## 定义组件
