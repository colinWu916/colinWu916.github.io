---
title: 一、复制execl批量搜索
date: 2024-10-01 10:53:11
author: 吴俊杰
top: false
keywords:
  - execl
  - 组件封装
summary: 工作没多久的时候遇到个需求，需要封装一个搜索框支持execl批量赋值标识实现批量的搜索，记录一下这个封装好的组件......
categories: 
  - 组件封装
tags:
  - 组件封装
  - execl批量搜索
---



# 一、Execl批量导入弹窗
## 1、自定义封装的弹窗组件
<p>&emsp;&emsp;啥也不说了，拿来就用，看代码</p>
<p>&emsp;&emsp;<span style="color: red">温馨提示哦，组件基于vue2+ts+iview组件库哦</span></p>

```javascript
<template>
  <div class="container">
    <div class="content">
      <ul>
        <li v-for="(item,index) in idetiferList" :key="index">{{item}}
          <Icon class="icon" type="ios-close-circle" @click="clearItem(index)"/>
        </li>
      </ul>
    </div>
    <div class="bottom">
      <Input class="search" type="text" v-model="identifers" :placeholder="`请输入或复制${nameMap[currentTab]}名称或标识，以空格、%、&分隔，最多输入100条`"/>
      <Button type="primary"  @click="getVerifyData">
        <Icon type="ios-search" style="font-size: 25px;"/>
        <span style="font-size: 16px; margin-left: 5px;">查询</span>
      </Button>
    </div>
  </div>
</template>
<script lang="ts">
import { Vue, Component, Prop, Watch, Emit } from 'vue-property-decorator';
@Component({
  components: {
    
  }
})
export default class PPrInfo extends Vue {

  @Prop({
    type: String,
    default: 'st'
  })
  currentTab: string;

  nameMap = {
    st: '试题',
    sj: '试卷',
    dc: '单词',
    klb: '课例包'
  }

  @Prop({
    type: String,
    default: ''
  })
  identifers: string;

  idetiferList:Array<string> = []

  lastOne:boolean = true;

  public clearItem(index:number):void {
    if(index === this.idetiferList.length - 1) {
      this.getLastArrOne();
    }
    this.idetiferList.splice(index,1);
    this.identifers = this.idetiferList.join(' ');
  }


  @Emit('getVerifyData')
  public getVerifyData() {
    return this.idetiferList;
  }

  @Watch('identifers', {
    deep: true,
    immediate: true
  })
  public WatchIdentifer(newValue:any):void {
    let tempArr = newValue.replace(/%/g,' ').replace(/&/g,' ').split(' ');
    let queryArr = [];
    if(tempArr.length) {
      queryArr = tempArr.filter(item => {
        return item !== '';
      })
      if(queryArr.length) {
        queryArr = Array.from(new Set(queryArr));
      }
    }
    this.idetiferList = queryArr;
    this.getArr();
  }

  @Emit('getLastArrOne')
  public getLastArrOne() {
    return false;
  }

  @Emit('getArr')
  public getArr() {
    return this.idetiferList;
  }

  mounted () {
  }
}
</script>

<style lang="less" scoped>
.container {
  width: 1000px;
  height: 270px;
  margin: 5px 0 0 90px;
  border-radius: 5px;
  box-shadow: 0px 0px 7px 0px rgba(0, 0, 0, .2);

  .content>ul {
    width: 100%;
    height: 220px;
    display: flex;
    padding-top: 15px;
    flex-wrap: wrap;
    justify-content: left;
    align-items: flex-start;
    overflow: auto;
    border-bottom: 1px solid #dcdee2;

    li {
      width: 188px;
      height: 50px;
      margin-left: 10px;
      padding: 0 10px;
      line-height: 50px;
      background-color: #f7fcff;
      position: relative;
      border: 1px solid #dcdee2;
      border-radius: 3px;
      margin-bottom: 18px;
      overflow: hidden;
      text-overflow:ellipsis;
      white-space:nowrap;

      .icon {
        font-size: 25px;
        position: absolute;
        top: 13px;
        right: 8px;
        color: #cbcfd4;
        cursor: pointer;
      }
    }
  }

  .bottom {
    width: 100%;
    height: 50px;
    display: flex;
    align-items: center;
    padding: 0 10px;
    /deep/ .ivu-input {
      border: 0px solid #dcdee2;
      width: 880px;
      height: 40px;
      font-size: 15px;
    }
    /deep/ .ivu-btn {
      margin-left: 362px;
      height: 40px;
      z-index: 100;
    }
  }
}
</style>
```

## 2、父组件调用改封装组件
<p>&emsp;&emsp;啥也不说了，拿来就用，看代码</p>
<p>&emsp;&emsp;<span style="color: red">温馨提示哦，当时为了兼容一些样式问题，所以做了一些调整，可能样式会有一点点的小问题，不过很容易改的哦。。</span></p>
<p>&emsp;&emsp;再提示一个问题，就是mounted里写了监听事件，主要是实现，点击其他空白处关闭该弹窗的功能，当然不要忘记在destoryed钩子里销毁监听事件哦，还有调用改组件时，有很多字段要传，还有一些事件函数也要传，这个就看各位小伙伴实际情况去改了哦。。</p>

```javascript
<template>
  <div>
    <InputModal class="input-fixed" id="inputModal" v-if="showInputModal" :currentTab="label" :identifers="name" 
    @getArr="getInputModalArr" @getVerifyData="getTableData" @getLastArrOne="getLastArrOne"></InputModal>
  </div>
</template>

<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';

@Component({
  components: {
    InputModal: () => import('./InputModal.vue')
  }
})

export default class TestCase extends Vue {

  mounted () {
    window.addEventListener('mousewheel', this.handleScroll);
      document.addEventListener("click", (e) => {
        const inputModal = document.getElementById("inputModal")
        const inputName = document.getElementById("inputName")
        if (inputModal && inputName) {
          if (!inputModal.contains((e as any).target) && !inputName.contains((e as any).target) && this.lastArrOne) {
            this.showInputModal = false;
          }
          this.lastArrOne = true;
        }
    });
  }

  destroyed(): void {
    window.removeEventListener('mousewheel', this.handleScroll);
  }
}
<script>
```

## 2、父组件调用改封装组件
<p>&emsp;&emsp;这里展示一下效果：</p>

![](/images/cp/1_1.gif)
