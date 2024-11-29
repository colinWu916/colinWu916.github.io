---
title: 三、component动态组件
tags: 工作知识总结
categories: 热爱我的工作
comments: true
---

# 三、工作知识难点摘要之动态组件
## 1、component动态组件的使用
### 1.1、简单介绍
<p>&emsp;&emsp;主要就是达到在各个引入的组件之间来回的动态切换，通过改变绑定的 is 属性。在实际的应用场景中，为了保证复用性和可维护性，采用vue内置的component组件来实现这一点。写法如下：</p>

```javascript
<component :is="currentComponent"></component>
```
<p>&emsp;&emsp;通过改变绑定的currentCompntent的值实现引用组件的切换。</p>

<p>&emsp;&emsp;另外一个场景就是，当在这些组件之间切换的时候，你有时会想保持这些组件的状态，以避免反复重新渲染导致的性能问题，写法如下：</p>

```javascript
<keep-alive>
  <component :is="currentComponent"></component>
</keep-alive>
```
### 1.2、用例介绍
<p>&emsp;&emsp;这里搭配了插槽使用，插槽会在后续章节单独说，这里只需要看compontent组件怎么用的就行。稍作解释一下，这里的CommonForm是一个自定义的表单组件，设置了插槽，这里调用插槽，并且通过动态组件的is属性绑定了slotProps.slotName（作用域插槽，暂时不用理解）的值。这个值的选项有五个分别是，caseOne、caseTwo、caseThree、caseFour、caseFive（分别对应了五个引入的组件），slotProps.slotName是哪一个值就渲染对应的组件，这样就可以实现，通过改变is属性的值来动态的决定要渲染什么组件。</p>

```javascript
<template>
  <div class="container">
    <CommonForm>
      <template v-slot:caseOne="slotProps">
        <component :is="slotProps.slotName"></component>
      </template>
      <template v-slot:caseTwo="slotProps">
        <component :is="slotProps.slotName"></component>
      </template>
      <template v-slot:caseThree="slotProps">
        <component :is="slotProps.slotName"></component>
      </template>
      <template v-slot:caseFour="slotProps">
        <component :is="slotProps.slotName"></component>
      </template>
      <template v-slot:caseFive="slotProps">
        <component :is="slotProps.slotName"></component>
      </template>
    </CommonForm>
  </div>
</template>
<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';
@Component({
  components: {
    CommonForm: () => import('@/pages/Common/CommonForm.vue'),
    caseOne: () => import('@/pages/Common/caseOne.vue'),
    caseTwo: () => import('@/pages/Common/caseTwo.vue'),
    caseThree: () => import('@/pages/Common/caseThree.vue'),
    caseFour: () => import('@/pages/Common/caseFour.vue'),
    caseFive: () => import('@/pages/Common/caseFive.vue'),
  }
})
export default class TestCase extends Vue {

}
<script>
```