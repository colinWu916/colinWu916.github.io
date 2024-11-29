---
title: 六、iview中的upload组件
tags: 工作知识总结
categories: 热爱我的工作
comments: true
---

# 一、工作知识难点摘要-iview组件之upload
## 1、Upload组件
### 1.1、什么是Upload组件

<p>&emsp;&emsp;一句废话：用于文件选择上传和拖拽上传控件。那为啥我要写这个呢，是因为工作中遇到了一个实际场景，也挺有意思的，可以作为一种思路，或许用的好可以发散思维。下面用两个例子对比一下实际场景，大家就可以清晰的感受啦！！！</p>

### 1.2、普通场景

<p>&emsp;&emsp;在实际的开发过程中，我们喜欢把Butotn组件写在Upload组件里面，正常情况下，Upload组件是空标签，没有内容，所以莫得渲染，我们加一个Button之后，就会渲染出一个虽然我们看着Button组件，但是加外一层包着Upload标签。如下例子所示，注意Upload一定要有一个action，这是后端提供的一个接口，所谓的上传就是把音频传给后端，然后后端再给个地址，音频再绑定这个地址，点击播放啥的就都可以了。（Emmm.....不知道这个理解对不对诶。。。）</p>

```javascript
<Upload :action="这里放上传的地址">
  <Button>导入音频</Button>
</Upload>  
```

<p>&emsp;&emsp;上面这个代码实际渲染就是一个button，当我们点击button的时候，会通过冒泡事件触发Upload的点击事件（handleClick）,随后出发一系列操作，弹窗文件窗口，选择音频，确认等.....</p>

### 1.3、特殊场景
<p>&emsp;&emsp;很普通对吧，也很简单，但是现在有一个场景，我想在Upload的文件弹窗弹出来之前，加个Modal框做个判断，我点击确定才继续执行Upload，点击取消就终止Upload事件。</p>

<p>&emsp;&emsp;我想了挺长时间，然后看到了Uplaod本身带着before-upload事件。官方解释：上传文件之前的钩子，参数为上传的文件，若返回 false 或者 Promise 则停止上传。我在before-upload里加了一个Modal，但并没有解决，这里确实会弹出模态框，但是在弹出模态框的之前Upload上传的弹窗已经弹出，无法阻止Uplaod继续执行。可以看一下下面的示例：</p>

![](/images/upload-fail.gif)

<p>&emsp;&emsp;注意文件上传弹窗和模态框的弹窗顺序，这并不是我们想要的，我们想Modal框先弹出来，当我点击确定的时候再继续弹出文件上传弹窗，而当我点击取消就直接终止上传事件。随后我又想了，给Button添加一个时间，在button事件里写一个Modal，点击确定的时候继续执行，点击取消就取消冒泡，反正也没解决。最后使用了下面这个方法。</p>

<p>&emsp;&emsp;那么怎么办呢，别急有方法。。这个时候啊。Button就不能写在Upload里面喽，要写在外面，然后我们给button一个点击事件，事件里弹出Modal框，我们点击确定的时候，利用$ref访问Upload组件，然后调用它的点击事件（注意是handleClick不是click哦。。。我们点击取消的时候就啥也不做就好啦，完美），好了看代码</p>

```javascript
<div>
  <Upload :action="这里放上传的地址" ref='audioUpload'></Upload> 
  <Button @click="getAudio">导入音频</Button> 
</div>

<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';
@Component({
  components: {

  }
})
export default class TestCase extends Vue {
  public getAudio() {
    this.$Modal.confirm({
      title: '替换确认',
      content: '<p>确认替换此音频？</p>',
      onOk: () => {
        (this.$refs.audioUpload as any).handleClick();
      },
      onCancel: () => { }
    })
  }
  mounted () {

  }
}
<script>
```
<p>恩恩讷讷讷讷，针不戳，然我们一起来看看效果吧。。。。</p>

![](/images/upload-success.gif)