---
title: 七、iview中的select组件
tags: 工作知识总结
categories: 热爱我的工作
comments: true
---

# 一、工作知识难点摘要-iview组件之自定义select
## 1、select组件
### 1.1、什么是Upload组件

<p>&emsp;&emsp;日常开局一句废话：iview的select组件使用模拟的增强下拉选择器来代替浏览器原生的选择器，选择器支持单选、多选、搜索。</p>

<p>&emsp;&emsp;也没啥好说的，主要就是强调一点，先来看正常写法。</p>

```javascript
<template>
  <Select v-model="tempOne" style="width:100px">
    <Option v-for="item in ultramans" :value="item.value" :key="item.value">{{ item.label }}</Option>
  </Select>
</template>

<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';
@Component({
  components: {

  }
})
export default class TestCase extends Vue {
  tempOne = '迪迦';
  ultramans:Array<any> = [
    {
      value:'迪迦',
      label:'迪迦'
    },{
      value:'赛罗',
      label:'赛罗'
    },{
      value:'雷欧',
      label:'雷欧'
    },{
      value:'艾克斯',
      label:'艾克斯'
    },{
      value:'艾克斯',
      label:'艾克斯'
    },
  ]
  mounted () {

  }
}
<script>
```

<p>&emsp;&emsp;简单的一批，都有点不想写了。就是说当我们往options里面塞dom元素的时候(有时候我们想自定义内部元素)，我这里包了一个span标签。要注意同时需要给options加上label，直接看怎么写的吧，对比一下就知道了，不加label的话，会有一些奇怪的错误，影响使用。</p>

<p>&emsp;&emsp;加一句iview组件官方解释:对选项内容可以进行自定义。注意在Option中使用label属性，可以让选择器优先读取该属性的值以显示，否则选中时显示的内容会和自定义的一样，这往往不是我们想要的。</p>

```javascript
<template>
  <Select v-model="tempOne" style="width:100px">
    <Option v-for="item in ultramans" :value="item.value" :label="item.label" :key="item.value">
      <span>{{item.label}}</span>
    </Option>
  </Select>
</template>

<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';
@Component({
  components: {

  }
})
export default class TestCase extends Vue {
  tempOne = '迪迦';
  ultramans:Array<any> = [
    {
      value:'迪迦',
      label:'迪迦'
    },{
      value:'赛罗',
      label:'赛罗'
    },{
      value:'雷欧',
      label:'雷欧'
    },{
      value:'艾克斯',
      label:'艾克斯'
    },{
      value:'艾克斯',
      label:'艾克斯'
    },
  ]
  mounted () {

  }
}
<script>
```