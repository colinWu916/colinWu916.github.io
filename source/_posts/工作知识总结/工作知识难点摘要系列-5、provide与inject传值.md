---
title: 五、provide与inject
tags: 工作知识总结
categories: 热爱我的工作
comments: true
---

# 五、工作知识难点摘要之provide与inject跨组件通信
## 1、provide与inject
### 1.1、温馨提示
<p>&emsp;&emsp;<span style="color: red">在说provide与inject之前需要先强调一下，在实际工作中尽量慎用改方法做组件通信。provide/inject和Vuex是有区别的，Vuex 中的全局状态的每次修改是可以追踪回溯的，而 provide/inject 中变量的修改是无法控制的，换句话说，你不知道是哪个组件修改了这个全局状态。provide/inject 破坏单向数据流原则。如果有多个后代组件同时依赖于一个祖先组件提供的状态，那么只要有一个组件修改了该状态，那么所有组件都会受到影响。这一方面增加了耦合度，另一方面，使得数据变化不可控。如果在多人协作开发中，这将成为一个噩梦。</span></p>

### 1.2、简单介绍
<p>&emsp;&emsp;这对选项是一起使用的。以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在起上下游关系成立的时间里始终生效。通俗的说就是：组件的引入层次过多，我们的子孙组件想要获取祖先组件得资源，那么怎么办呢，总不能一直取父级往上吧，而且这样代码结构容易混乱。这个就是这对选项要干的事情，允许跨组件通信，但是要强调的一点是，provide只能写在上级组件,inject只能写在下级组件哦。。。</p>

<p>&emsp;&emsp;做一个通俗的解释：provide 可以在祖先组件中指定我们想要提供给后代组件的数据或方法，而在任何后代组件中，我们都可以使用 inject 来接收 provide 提供的数据或方法。看下面这个简单的例子：爷爷组件通过Provide传值给孙子，直接跨过父亲组件，注意改方法有严格的上下层级关系，例如该方法是不可用于兄弟间组件通信的。</p>

```javascript
//爷爷组件Grandfather.vue
<template>
  <div>
    <div>我是爷爷组件，我引入了父亲组件哦。。</div>
    <Father></Father>
  </div>
</template>
<script lang="ts">
import { Vue, Component, Provide } from 'vue-property-decorator';
@Component({
  components: {
    Father: () => import('@/pages/Father.vue')
  }
})
export default class TestCase extends Vue {
  tempArr:Array<any> = ['迪迦', '捷德', '赛罗', '贝利亚']
  @Provide('getTempArr')
  public getTempArr() {
    return this.tempArr;
  }
  mounted () {

  }
}
<script>

//父亲组件Father.vue
<template>
  <div>
    <div>我是父亲组件，我引入了孙子组件哦，可怜的我只是过度一下。。</div>
    <Son></Son>
  </div>
</template>
<script lang="ts">
import { Vue, Component,} from 'vue-property-decorator';
@Component({
  components: {
    Son: () => import('@/pages/Son.vue')
  }
})
export default class TestCase extends Vue {
  mounted () {

  }
}
<script>

//孙子组件Son.vue
<template>
  <div>
    <div>我是孙子组件，看我接收数据</div>
    <Button @click="getArr">点击我调用爷爷那边传来的函数哦。。</Button>
  </div>
</template>
<script lang="ts">
import { Vue, Component, inject} from 'vue-property-decorator';
@Component({
  components: {
    
  }
})
export default class TestCase extends Vue {
  sonTempArr:Array<any> = [];
  @Inject()
  getTempArr: Function;
  public getArr() {
    this.sonTempArr = this.getTempArr();
  }
  mounted () {

  }
}
<script>
```

### 1.3、一个好的demo
<p>&emsp;&emsp;上面那个例子啊，是我自己在工作中看到的，所以写了个自己熟悉的，下面这个例子是在晚上看到的，特别适合理解，就搬过来，语言是vue3+ts哦。</p>

### 1.3.1、情况一：一级组件传值给三级组件
<p>&emsp;&emsp;注：貌似provide没有提供批量方法， 只能每个变量写一句。</p>

```javascript
//一级组件
<template>
  <div>
    <son />
  </div>
</template>
<script setup>
import son from "./son.vue";
import { provide } from "vue";
provide("abc", "123");
</script>

//二级组件
<template>
  <div>
    <grandson />
  </div>
</template>
<script setup>
import grandson from "./grandson.vue";
</script>

//三级组件
<template>
  <div>我是孙子</div>
</template>
<script setup>
import { inject } from "vue";
const abc = inject("abc");
console.log(abc);
</script>
```

### 1.3.2、情况二：一级组件修改数据，孙子组件监听变更
<p>&emsp;&emsp;注：provide的变量必须是响应式变量，孙子组件监听的变量也必须是响应式变量。</p>

```javascript
//一级组件
<template>
  <div>
    <son />
    <button @click="abc = '456'">{{ abc }}</button>
  </div>
</template>
<script setup>
import son from "./son.vue";
import { provide } from "vue";
ref: abc = "123";
provide("abc", $abc);
</script>

//三级组件
<template>
  <div>我是孙子 - {{abc}}</div>
</template>
<script setup>
import { inject, watch } from "vue";
ref: abc = inject("abc");
watch($abc, () => {
  console.log(abc + '变了');
});
</script>
```

### 1.3.3、情况三：孙子组件修改数据，一级组件监听变更
<p>&emsp;&emsp;注：一级组件除了提供数据，还要提供一个修改数据的方法，孙子组件要接收并使用这个方法，这样修改的就是一级组件的数据，修改之后又会影响孙子组件的数据。</p>

```javascript
//一级组件
<template>
  <div>
    <son />
    <button @click="abc = '456'">{{ abc }}</button>
  </div>
</template>
<script setup>
import son from "./son.vue";
import { provide } from "vue";
ref: abc = "123";
function updateAbc(val) {
  abc = val;
}
provide("abc", $abc);
provide("updateAbc", updateAbc);
</script>

//三级组件
<template>
  <div @click="updateAbc('789')">我是孙子 - {{abc}}</div>
</template>
<script setup>
import { inject, watch } from "vue";
ref: abc = inject("abc");
const updateAbc = inject('updateAbc');
watch($abc, () => {
  console.log(abc + '变了');
});
</script>
```
### 1.3.4、情况四：禁止孙子组件修改一级组件的数据
<p>&emsp;&emsp;注：禁止的话，一级组件传递的变量必须是只读的，可以用readonly实现，也可以是shallowRef实现。这样孙子组件修改数据的话，一级组件不会有反应。</p>

```javascript
//一级组件
<template>
  <div>
    <son />
    <button @click="abc = '456'">{{ abc }}</button>
  </div>
</template>
<script setup>
import son from "./son.vue";
import { shallowRef, provide } from "vue";
let abc = shallowRef("123");
function updateAbc(val) {
  abc = val;
}
provide("abc", abc);
provide("updateAbc", updateAbc);
</script>
```

