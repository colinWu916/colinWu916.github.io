---
title: 十二、在线预览Word
tags: 工作知识总结
categories: 热爱我的工作
comments: true
---

# 一、在线浏览word并下载

## 1、简单介绍
<p>&emsp;&emsp;上篇文章介绍了在线浏览pdf工具，今天来写一篇在线浏览word并支持下载功能。我真是又努力又聪明啊。。。简直是放屁，其实是甲方爸爸改需求了，pdf变成word了，完了我那个pdf刚弄好，正开心着呢。害，我这小垃圾没弄过word啊，又去问了老大，哈哈哈哈哈哈哈哈。。。好吧弄好了，来记录下的。。。。。。</p>

## 2、实现流程
### 2.1、基本步骤
<p>&emsp;&emsp;首先啊，一样的，我们也是要上传word文档的啦,上传过word之后啊，我们一样拿到后端返回的文件标识：xxxxxxxxxxx.doc</p>

<p>&emsp;&emsp;那么怎么预览呢，我这里是用的v-html属性，因为调用的接口后端返回的是xml字符串。我来说一下步骤：首先你得自己配置一个路由，因为你要打开新的网页必须有地址，那么你得配一个路由地址，这个地址指向你新建的一个.vue文件，那么就是在这个文件里拿到xml字符串并显示的。随后啊，我们点击按钮（在新窗口打开word）。注意看哪个路径，我这里是路由路径拼接上了两个参数，因为我们要在新建的vue里调用接口，但是参数是在其他的.vue文件的，那么怎么把参数传过去的，这里我用的就是通过路由来传参。</p>

```javascript
<template>
  <div class="container">
    <Upload ref="uploadStandardWord" style="display:inline-block" :action="uploadObj.action" :multiple="false" :show-upload-list="false" :accept="uploadObj.accept" :format="uploadObj.format" :max-size="uploadObj.maxSize" :on-exceeded-size="exceededSizeFun" :on-format-error="formatError" :on-success="(response)=>handleSuccess(response)" :on-error="handleError">
    </Upload>
    <button @click="openPDF">在新窗口打开word</button>
  </div>
</template>
<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';
import { getWordData } from '@/assets/js/utils';

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
      accept: '.doc, .docx',
      format: ['doc', 'docx'],
      maxSize: 1024 * 1024,
      data: {}
    }

  public openPDF() {
    window.open(`#/home/atlasManage/viewWord?identifier=${this.currentRow.identifier}&path=${this.currentRow.specifications[0].value}`);
  }
}
<script>
```

<p>&emsp;&emsp;欧凯哦开，上个文件我们点击后要打开一个新的网页了，那么这个网页就是指向一个.vue文件嘛。。这个文件是这样的。我们通过路由拿到参数，然后拿参数调用接口，这个data啊，就是后端返回的xml字符串。</p>

<p>&emsp;&emsp;最后加一个下载功能，那么下载功能啊，当然是要接口的路径的，然后拼接上我这里是标识和path(上传后返回的文件标识：xxxxxxx.doc)，这两个都是从路由里拿的。所以你看到了吧，在上一个文件，我们在路由路径里拼接了两个参数，是用的上滴。。。因为这个本身是一个下载路径，多以window.open打开后就会自动下载拉，真棒。。。。。</p>

```javascript
<template>
  <div>
    <div v-html="data"></div>
  </div>
</template>
<script lang="ts">
import { Vue, Component} from 'vue-property-decorator';
import { sjRollMakingWordView } from '@/http/api/Atlas';

@Component({
  components: {

  }
})
export default class ViewWord extends Vue {

  data: any = '';

  public async viewWordContent() {
    const tempQuery = this.$route.query;
    const params = {
      identifier: tempQuery.identifier,//一个标识，每个项目参数不同不用太在意
      path: tempQuery.path//上传后返回的文件标识：xxxxxxx.doc
    }
    const res = await sjRollMakingWordView(params);
    if(res) {
      this.data = res.data;
    }
  }

  public async downLoadWord() {
    const tempQuery = this.$route.query;
    window.open(`/graphApi/api/v1/common/source/file/download?identifier=${tempQuery.identifier}&path=${tempQuery.path}`);
  }

  mounted () {
    this.viewWordContent()
  }
  
}
</script>
```













<p>&emsp;&emsp;</p>
<p>&emsp;&emsp;</p>
<p>&emsp;&emsp;</p>
<p>&emsp;&emsp;</p>