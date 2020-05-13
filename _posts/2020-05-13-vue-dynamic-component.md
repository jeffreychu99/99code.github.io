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

```javascript
<template>
    <div>
        <input type="text" v-model="name" />
        <input type="text" v-model="age" />
        <button @click="changeName">click</button>
    </div>
</template>

<script>
export default {
    name: 'MyControl',
    data() {
        return {
            name: "jefrey",
            age: 10
        }
    },
    methods: {
        changeName() {
            this.name = "another name"
        }
    }
}
</script>
```

click按钮事件改变的是该组件内文本框的值，对其它组件没有影响，互不干涉，很好的说明了组件的封装性。

## 应用组件

```javascript
<template>
  <div id="app">
    <button @click="add">动态增加</button>
    <button @click="get">获取组件数据</button>
    <MyControl v-for="componet in components" :ref='componet.id' :key= 'componet.id'/>
  </div>
</template>

<script>
import MyControl from './components/MyControl'

export default {
  name: 'App',
  components: {
    MyControl
  },
  data() {
    return {
      components: [{
        id: 1
      }],
      id: 1
    }
  },
  methods: {
    add() {
      this.components.push({
        id: ++this.id
      })
    },
    get() {
      for (let refIndex in this.$refs) {
        let ref = this.$refs[refIndex]
        console.log(ref[0].name)
        console.log(ref[0].age)
      }     
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

```

关键的两段代码如下所示

```javascript
// 遍历数据，动态创建组件列表
<MyControl v-for="componet in components" :ref='componet.id' :key= 'componet.id'/>
```

```javascript
// 获取组件的数据
for (let refIndex in this.$refs) {
    let ref = this.$refs[refIndex]
    console.log(ref[0].name)
    console.log(ref[0].age)
}
```

从下动图可以看出，组件内部数据都是独立的，获取组件数据按钮可以遍历出各个组件的数据内容，这样在提交后台的时候，前端javascript就可以很方便的整合数据了。

![](/images/posts/vue-dynamic-component/record.gif)


```javascript
// 获取组件的数据
for (let refIndex in this.$refs) {
    let ref = this.$refs[refIndex]
    console.log(ref[0].name)
    console.log(ref[0].age)
}
```

## 获取组件

整合数据提交到后台，如下代码所示，遍历完成后就可以直接将data按照AJAX等方式提交给后台。

```javascript
// 待提交的数据
let data = []
for (let refIndex in this.$refs) {
    let ref = this.$refs[refIndex]

    data.push({name: ref[0].name, age: ref[0].age})
}
```