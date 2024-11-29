---
title: 一、this.$set()
tags: 工作知识总结
categories: 热爱我的工作
comments: true
---

# 一、工作知识难点摘要之this.$set()
## 1、this.$set()的使用
### 1.1、为什么要有this.$set()
<p>&emsp;&emsp;在我们使用vue进行开发的过程中，可能会遇到一种情况：生成vue实例后，再次给数据赋值时，有时候并不会自动更新到视图上去； 当我们去看vue文档的时候，会发现有这么一句话：如果在实例创建之后添加新的属性到实例上，它不会触发视图更新。如下代码，给 student 对象新增 age 属性。</p>

```javascript
<script lang="ts">
  import { Vue, Component } from 'vue-property-decorator';
  @Component({
    components: {
      
    }
  })
  export default class TestCase extends Vue {
    student:any = {
      name: '小先生',
      gender: '男'
    }
    mounted () {
      this.student.age = 24
    }
  }
<script>
```

<p>&emsp;&emsp;受 ES5 的限制，Vue.js 不能检测到对象属性的添加或删除。因为 Vue.js 在初始化实例时将属性转为 getter/setter，所以属性必须在 data 对象上才能让 Vue.js 转换它，才能让它是响应的，this.$set()的出现就是为了解决这个问题。</p>

<p>&emsp;&emsp;开发背景：平常在做项目的时候，后端接口返回 json 数据，数据是要渲染到页面的，这个时候并不会有什么问题。但是当因为某些特殊需求，需要在返回的数据对象里面添加一个字段，添加是添加进去了，但是页面渲染却没有变化，因为不是响应式的。如果我们要让这个新字段是响应式的，就要使用到this.$set()来注入数据。</p>

### 1.2、this.$set()的使用
<p>&emsp;&emsp;this.$set(target, key, value);</p>

<p>&emsp;&emsp;<strong>target</strong>：表示数据源，即要操作的数组或者对象;</p>

<p>&emsp;&emsp;<strong>key</strong>：要操作的对象的某一个属性;</p>

<p>&emsp;&emsp;<strong>value</strong>：更改的数据.</p>

<p>&emsp;&emsp;当我们要给student添加age属性时，可写为：this.$set(this.student, 'age', 18),这种写法会给 student 添加的 age 添加getter/setter方法，可以被监听，页面也可以实时渲染。<p>

### 1.3、this.$set()实例

<p><strong>注意</strong>：不用this.$set()方法，age是不会被渲染出来的。。。。</p>

![](/images/set.gif)


