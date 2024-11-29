---
title: 三、audio标签的使用
tags: 自定义封装组件
categories: 组件封装
comments: true
summary: 今天用audio实现了一个功能。平常我们用audio的时候啊，都是一次性用到好几个，那么一个页面有好几个音频标签的时候，怎么控制单一音频的播放暂停呢？怎么控制每次点击是否从头播放呢？怎么协调所有音频的播放和暂停呢？
---

# 一、audio标签的使用
## 1、简单介绍
<p>&emsp;&emsp;我太牛了，我太牛了，我太牛了！！！！！！！！！！！！！！！！！！！！！！！！！！！今天用audio实现了一个功能。平常我们用audio的时候啊，都是一次性用到好几个，那么一个页面有好几个音频标签的时候，怎么控制单一音频的播放暂停呢？怎么控制每次点击是否从头播放呢？怎么协调所有音频的播放和暂停呢？？？？来来来，教学教学。那么音频的使用分为好几个部分，首先你得调用音频组件，其次你得上传音频给后端，再获取后端的音频地址，将地址赋值给audio标签的src属性，点击就可以播放音频啦。</p>

### 1.1、上传音频
<p>&emsp;&emsp;那我们从第一步开始，首先你想播放音频，必须得有音频，那么在实际工作中我们访问的音频一定是后端传过来的，那么后端的音频又是从哪里来的呢，要么本来就有默认的，要么就是用户自己导入的音频。那我们先来看怎么导入音频吧。xdm，上代码！！！</p>
<p>&emsp;&emsp;上传首先要用到upload组件，而且考虑的比较细致的话，上传的过程还是比较复杂的，对了我这里用的是iview的组件库，关于upload不熟悉的可以去官网看啦，这里简单解释一下组件绑定的属性都是什么意思。</p>
<p>&emsp;&emsp;&emsp;&emsp;<strong>action：</strong>那么这个呢就是我们上传的地址音频上传到哪里去（后端提供）。</p>
<p>&emsp;&emsp;&emsp;&emsp;<strong>multiple：</strong>这个是指是否支持上传多个文件。</p>
<p>&emsp;&emsp;&emsp;&emsp;<strong>show-upload-list：</strong>是否显示已上传文件列表。</p>
<p>&emsp;&emsp;&emsp;&emsp;<strong>accept：</strong>接受上传的文件类型有哪些（一般音频的话就是MP3, wov, Wav）。</p>
<p>&emsp;&emsp;&emsp;&emsp;<strong>format：</strong>支持的文件类型，与accept不同的是，format是识别文件的后缀名，accept为input标签原生的accept属性，会在选择文件时过滤，可以两者结合使用。</p>
<p>&emsp;&emsp;&emsp;&emsp;<strong>max-size：</strong>文件大小限制，单位kb。</p>
<p>&emsp;&emsp;好的，了解了基本属性后，那我们再来说一说这里面涉及到的很多的回调函数啊。</p>
<p>&emsp;&emsp;<strong>1、on-exceeded-size：</strong>这个是文件大小校验失败的回调，文件超出指定大小限制时的钩子。</p>
<p>&emsp;&emsp;<strong>2、on-format-error：</strong>文件格式验证失败时的钩子。</p>
<p>&emsp;&emsp;<strong>3、handleSuccess：</strong>文件上传成功后的钩子，注意response返回值。</p>
<p>&emsp;&emsp;<strong>4、on-error：</strong>文件上传失败后的钩子。</p>

```javascript
<template>
  <div class="container">
    <Upload ref="audioUpload" :action="uploadObj.action" :multiple="false" :show-upload-list="false" :accept="uploadObj.accept" :format="uploadObj.format" :max-size="uploadObj.maxSize" :on-exceeded-size="()=>exceededSizeFun('audio')" :on-format-error="()=>formatError('audio')" :on-success="(response)=>handleSuccess(response,'audio',index,audioIndex)" :on-error="handleError">
      <Button>导入音频</Button>
    </Upload>
  </div>
</template>
<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';

@Component({
  components: {}
})
export default class TestCase extends Vue {

  //upload属性参数
  public get uploadObj (): {
    action: string;
    accept: string;
    maxSize: number;
    format: Array<string>;
    data: any;
    } {
     return {
       action: `/graphApi/word-card/word/${this.id}/uploadFile`,
       accept: '.MP3,.wov,.Wav',
       format: ['MP3', 'wov', 'Wav'],
       maxSize: 1024 * 10,
       data: {
         id: this.id
       }
     }
   }

  //文件大小校验失败回调
  public exceededSizeFun (type: string): void {
    this.$Message.warning(
      `文件大小不能超过${(this[type === 'audio' ? 'uploadObj' : 'imageUploadObj'] as any).maxSize / 1024}M`
    );
  }

  // 文件格式不对回调
  public formatError (type: string): void {
    this.$Message.warning(
      `只支持上传${(this[type === 'audio' ? 'uploadObj' : 'imageUploadObj'] as any).format.join('、')}格式的文件！`
    );
  }

  // 上传成功的回调（这里的参数大家可能看不懂，我这里传参主要用到index和audioIndex，这跟我的数据结构有关，大家不用太关注，我拿到这两个索引是为了把数据改了）
  //大家只要关注response就好了，这是我们上传成功后的返回值，这个后面播放音频是要用到的
  public handleSuccess (response, type, index, audioIndex) {
    if (response.status === 200) {
      const { data } = response;
      this.$set(this.data[index].audioMessage[audioIndex], 'audioPath', data);
    }
  }

  // 上传失败
  public handleError (): void {
    this.$Message.error('上传失败，请重新上传！');
  }
}
<script>
```

### 1.2、引用音频标签
<p>&emsp;&emsp;通过上面的代码，我们已经实现了上传音频。那么接下俩我们使用音频这个标签呢？，直接看代码啊，我们对着代码来说啊。这里我们引用了audio标签，并且只给了src属性，这里调用了getPath方法，那么这个方法的参数其实就是handleSuccess的返回值response结构出来的data，通过getPath这个方法（里面做了一些判断，不用在意这个根据实际情况而定）我们会调用接口拿到这个音频的播放地址，这样就可以播放了（播放和暂停待会放在第三部分一起说）,对了音频有两个特别重要的属性，我在这里简单的说一下，就是currentTime和duration，分别是指当前音频播放的时常和当前音频的总时长，这两个在实际工作中对我们做一些判断是很有作用的。</p>

```javascript
<template>
  <div class="container">
    <audio style="display:none" :src="getPath(audioPath)"></audio>
    <Upload ref="audioUpload" :action="uploadObj.action" :multiple="false" :show-upload-list="false" :accept="uploadObj.accept" :format="uploadObj.format" :max-size="uploadObj.maxSize" :on-exceeded-size="()=>exceededSizeFun('audio')" :on-format-error="()=>formatError('audio')" :on-success="(response)=>handleSuccess(response,'audio',index,audioIndex)" :on-error="handleError">
      <Button>导入音频</Button>
    </Upload>
  </div>
</template>
<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';

@Component({
  components: {

  }
})
export default class TestCase extends Vue {

  //获取音频播放路径
  public getPath (path: string) {
    let finalPath = '';
    if (path) {
      if (path.indexOf('https://') === 0 || path.indexOf('http://') === 0) {
        finalPath = path;
      } else {
        //这里就是调用接口返回音频播放地址
        finalPath = getFileResource(path.split('/')[1], path.split('/')[0]);
      }
    }
    return finalPath;
  }
}
<script>
```

### 1.3、协调播放
<p>&emsp;&emsp;那么从第二部分我们拿到音频的播放地址了，就可以随意播放音频了，那么怎么播放呢。一般是给音频一个特定的ref，然后用this.$refs去访问音频元素，然后调用它的play（）和pause（）方法，也就是播放和暂停。我这里很有讲究的，直接把注释写代码里了，对着代码看，不然不容易解释。</p>

```javascript
<template>
  <div class="container">
    <audio style="display:none" :src="getPath(audioPath)"></audio>
    <Button type="primary" @click="playAudio(index,audioIndex)">播放音频</Button>
    <Upload ref="audioUpload" :action="uploadObj.action" :multiple="false" :show-upload-list="false" :accept="uploadObj.accept" :format="uploadObj.format" :max-size="uploadObj.maxSize" :on-exceeded-size="()=>exceededSizeFun('audio')" :on-format-error="()=>formatError('audio')" :on-success="(response)=>handleSuccess(response,'audio',index,audioIndex)" :on-error="handleError">
      <Button>导入音频</Button>
    </Upload>
  </div>
</template>
<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';

@Component({
  components: {

  }
})
export default class TestCase extends Vue {

  //用于判断当前音频是否处在播放状态
  isPlaying = true;
  //用于判断是否点击的是同意音频，用于协调不同音频的播放设置
  tempAudio = '';

  public playAudio(index, audioIndex) {
    //首先我这里一个页面有好几个音频，所以在点击播放时先拿到所有的音频，依次遍历暂停，防止都在播放干扰
    const audioAll = document.querySelectorAll('audio');
    for (const i in audioAll) {
      if ((typeof audioAll[i]) === 'object') {
        audioAll[i].pause();
      }
    }
    //利用传进来的两个参数，访问到当前被点击的音频
    const currentAudio = document.getElementById('audio' + index + audioIndex);
    if (currentAudio) {
      //判断：如果这个音频是不是上一次点击的音频
      if (this.tempAudio === index.toString() + audioIndex.toString()) {
        //如果点击的是上一次点击的音频，我们要做判断，改音频是不是在播放（isPlaying）属性，如果是就暂停，如果不是就播放
        if ((currentAudio as any).currentTime === 0 || this.isPlaying === false) {
          //这里是播放，注意用了currentTime，这个是播放的时间，置为0就是从头播放
          (currentAudio as any).currentTime = 0;
          //播放
          (currentAudio as any).play();
          //播放的同时，我们给一个状态，表示当前有音频正在播放
          this.isPlaying = true;
        } else {
          (currentAudio as any).pause()
          //暂停的时候我们也给一个状态，说明当前没有音频在播放
          this.isPlaying = false;
        }
      } else {
        //如果不是上一次点击的音频，直接走else，那就是直接从头开始播放该音频
        (currentAudio as any).play();
        (currentAudio as any).currentTime = 0;
        this.isPlaying = true;
      }
      //这里就是只要点击了音频，那我们就更新一下点击的是哪一个音频，用于与下一次点击时候做对比
      this.tempAudio = index.toString() + audioIndex.toString()
    }
  }
}
<script>
```

<p>&emsp;&emsp;好啦，这里简单说一下协调播放后的效果是什么样的。当你点击任何一个音频的时候，其他正在播放的音频会暂停，被点击的音频会从头播放；当一个音频正在播放，我们点击它的时候它会暂停，暂停后你再点击，它会从头播放。当有一个音频在播放时，我们去点击另外一个音频，那么这个音频会暂停，那个被点击的音频会从头播放。</p>