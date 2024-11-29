---
title: 十一、在线预览PDF
tags: 工作知识总结
categories: 热爱我的工作
comments: true
---

# 一、PDF.js的使用

## 1、简单介绍
<p>&emsp;&emsp;最近做了个事，工作中遇到一个需求，就是要在线显示pdf，那咋办，我这垃圾小白也没做过。于是乎问了老大，老大说用pdf.js。然后乎我就去看了一下，发现好用又简单。既然这么实用那就给记下来吧！！！！！</p>

## 2、使用PDF.js

### 2.1、下载pdf.js
<p>&emsp;&emsp;首先啊，我们不需要额外下载插件，而是直接去官网，下载pdf.js的文件。下面附上下载链接</p>

[pdf.js官网](http://mozilla.github.io/pdf.js)

![](/images/pdfjs.jpg)

### 2.2、在项目中添加pdf.js

<p>&emsp;&emsp;下载后的文件结构是这样的，三个子文件，build、web、license。这个license是可以直接删除的，没啥用。</p>

![](/images/jiegou.png)

<p>&emsp;&emsp;那么我们呢直接把整个文件夹放进我们项目中的public文件夹下面就好啦！这样就可以用啦</p>

![](/images/mulu.jpg)

### 2.3、在项目中使用pdf.js

<p>&emsp;&emsp;那么怎么使用呢？首先啊，我们还是得用到upload组件，因为我们要上传pdf文件给后台，upload组件在音频那块已经说过了，这里简单说一下吧。。。那个那个回调函数就不写了啊，大家可以去看audio音频的那个文章，也是这个系列的。那么我们上传过pdf之后啊，后台会返回这个文件的标识：xxxxxxxxx.pdf</p>

<p>&emsp;&emsp;有了这个标识后我们应该怎么办呢，当然是调用接口啦，比如说这个接口叫：getPdfData,假装引用一下。接口是后端写的，我们以上传返回后的文件标识为参数调用改接口，然后接口会返回一个地址，这就是文件的地址了。</p>

```javascript
<template>
  <div class="container">
    <Upload ref="uploadStandardWord" style="display:inline-block" :action="uploadObj.action" :multiple="false" :show-upload-list="false" :accept="uploadObj.accept" :format="uploadObj.format" :max-size="uploadObj.maxSize" :on-exceeded-size="exceededSizeFun" :on-format-error="formatError" :on-success="(response)=>handleSuccess(response)" :on-error="handleError">
    </Upload>
    <button @click="openPDF">在新窗口打开pdf</button>
  </div>
</template>
<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';
import { getPdfData } from '@/assets/js/utils';

@Component({
  components: {

  }
})
export default class TestCase extends Vue {
  uploadObj: {
    action: string;
    accept: string;
    maxSize: number;
    format: Array<string>;
    data: any;
  } = {
      action: '/graphApi/api/v1/common/source/file/upload',
      accept: '.pdf',
      format: ['pdf'],
      maxSize: 1024 * 1024, //1个G
      data: {}
    }

  public async openPDF() {
    //params是接口参数，问后端
    const res = awiat getPdfData(params);
    if(res) {
      //window.open就是在新窗口打开网页，然后引用pdf.js的文件，并且把地址传进去就好了，是不是很简单
      window.open(`/pdf/web/viewer.html?file=${encodeURIComponent(res.pdfUrl)}`);
    }
  }
}
<script>
```