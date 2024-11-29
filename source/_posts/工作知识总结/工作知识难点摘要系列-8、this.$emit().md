---
title: 八、this.$emit()
tags: 工作知识总结
categories: 热爱我的工作
comments: true
---

# 一、工作知识难点摘要-this.$emit()的使用
## 1、this.$emit()的使用
### 1.1、背景

<p>&emsp;&emsp;这是一个很简单的知识点，那为啥我要写呢，这么基础的东西有啥好写的，妈的我就是菜，老子记不住，每次用都还得查一下，今天必须写在我的集锦里，大爷的！！！</p>

### 1.2 第一种写法
<p>&emsp;&emsp;首先，我是把vue和ts结合着写的，因为工作中都是结合的，第一种写法用的是ts的装饰器@Emit，那么直接看代码。首先啊，子组件得引入装饰器Emit,然后点击button，触发事件deliverValue，该事件就是传值的，我们在子组件写@Emit('这里放的是名称，待会父组件引用子组件时，子组件里绑定的事件要与这个名字保持一致哦')</p>

```javascript
//首先我们看子组件Son.vue
<template>
  <div class="container">
    <button @click="deliverValue"></button>
  </div>
</template>
<script lang="ts">
  import { Vue, Component, Emit } from 'vue-property-decorator';
  @Component({
    components: {
      
    }
  })
  export default class TestCase extends Vue {
    const utralMenArr:any = ['迪迦','塞罗','捷德','欧布','艾克斯','阿古茹']

    @Emit('deliverValue')
    public deliverValue() {
        return this.utralMenArr;
    }

    mounted () { }
  }
<script>


//然后我们来看父组件Father.vue
<template>
  <div class="container">
    <Son @deliverValue="getArr"></Son>
  </div>
</template>
<script lang="ts">
  import { Vue, Component, Emit } from 'vue-property-decorator';
  @Component({
    components: {
      Son: () => import('@/pages/Son.vue')
    }
  })
  export default class TestCase extends Vue {

    public getArr(value) {
        console.log(value);
    }

    mounted () { }
  }
<script>
```
### 1.3 第二种写法

<p>&emsp;&emsp;那么第二种写法呢，就是用this.$emit()了，基本用法和上面那个是一样的，只不过是换了一种写法，那我们直接来看吧。子组件的this.$emit('函数名','传的值')</p>

```javascript
//首先我们看子组件Son.vue
<template>
  <div class="container">
    <button @click="deliverValue"></button>
  </div>
</template>
<script lang="ts">
  import { Vue, Component } from 'vue-property-decorator';
  @Component({
    components: {
      
    }
  })
  export default class TestCase extends Vue {

    const utralMenArr:any = ['迪迦','塞罗','捷德','欧布','艾克斯','阿古茹']

    public deliverValue() {
        this.$emit('deliverValue', this.utralMenArr)
    }

    mounted () {}
  }
<script>


//然后我们来看父组件Father.vue
<template>
  <div class="container">
    <Son @deliverValue="getArr"></Son>
  </div>
</template>
<script lang="ts">
  import { Vue, Component, Emit } from 'vue-property-decorator';
  @Component({
    components: {
      Son: () => import('@/pages/Son.vue')
    }
  })
  export default class TestCase extends Vue {

    public getArr(value) {
        console.log(value);
    }

    mounted () { }
  }
<script>
```

