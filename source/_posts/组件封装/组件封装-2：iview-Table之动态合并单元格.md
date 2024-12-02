---
title: 二、iview-Table之动态合并单元格
date: 2024-10-02 08:34:01
author: 吴俊杰
top: false
keywords:
  - iview
  - table
  - 合并单元格
  - 组件封装
summary: iview越用越觉得不好用，很多内容不得不适用render函数去实现，这里记录一下使用iview table组件的时候实现单元格的合并......
categories: 
  - 组件封装
tags:
  - iview
  - table
  - 合并单元格
---


# 一、iview-Table之动态合并单元格
## 1、普通合并单元格
<p>&emsp;&emsp;我们先来看一下简单的合并单元格，也是iview官网给的示例，可以帮助我们理解。通过设置属性span-method可以指定合并行或列的算法，方法传入一个对象，对象里有四个属性,分别是:</p>
<p>&emsp;&emsp;&emsp;&emsp;<strong>row:</strong> 当前行</p>
<p>&emsp;&emsp;&emsp;&emsp;<strong>column:</strong> 当前行</p>
<p>&emsp;&emsp;&emsp;&emsp;<strong>rowIndex:</strong> 当前行</p>
<p>&emsp;&emsp;&emsp;&emsp;<strong>columnIndex:</strong> 当前行</p>
<p>&emsp;&emsp;该函数可以返回一个包含两个元素的数组，第一个元素代表rowspan,表示合并几行，第二个元素代表colspan表示合并几列。了解了用这个属性，那么怎么用呢，我们来看代码啊，并且对着代码看下效果哦！！</p>

```javascript
<template>
    <div>
      <Table :columns="columns" :data="data" :span-method="handleSpan"></Table>
    </div>
</template>
<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';
@Component({
  components: {

  }
})
export default class TestCase extends Vue {
  //表头列
  columns:Array<any> = [
    {
      title: 'Date',
      key: 'date'
    },
    {
      title: 'Name',
      key: 'name'
    },
    {
      title: 'Age',
      key: 'age'
    },
    {
      title: 'Address',
      key: 'address'
    },
  ];
  
  data: Array<any> = [
    {
      name: 'John Brown',
      age: 18,
      address: 'New York No. 1 Lake Park',
      date: '2016-10-03'
    },
    {
      name: 'Jim Green',
      age: 24,
      address: 'London No. 1 Lake Park',
      date: '2016-10-01'
    },
    {
      name: 'Joe Black',
      age: 30,
      address: 'Sydney No. 1 Lake Park',
      date: '2016-10-02'
    },
    {
      name: 'Jon Snow',
      age: 26,
      address: 'Ottawa No. 2 Lake Park',
      date: '2016-10-04'
    }
  ];

  handleSpan({ row, column, rowIndex, columnIndex }) {
    if (rowIndex === 0 && columnIndex === 0) {
      //如果是第一行第一列，则返回[1,2],就是该单元格合并一行，合并两列
      return [1, 2];
    } else if (rowIndex === 0 && columnIndex === 1) {
      //因为合并了第一行，第二列的单元格，所以要给这个单元格返回[0,0],就是关闭这个单元格
      return [0, 0];
    }
    if (rowIndex === 2 && columnIndex === 0) {
      //不同写法但同理，第三行第一列单元格合并两行，合并一列
      return {
        rowspan: 2,
        colspan: 1
      };
    } else if (rowIndex === 3 && columnIndex === 0) {
      //同理，被合并的单元格要置空
      return {
        rowspan: 0,
        colspan: 0
      };
    }
  }

  mounted () {

  }
}
<script>
```
<p>&emsp;&emsp;以上就是代码，那我们来看下这个合并后的效果，效果杠杠的！！第一行的前两列合并，第一列的后两行合并。</p>

![](/images/mergeTable.jpg)

## 2、复杂合并单元格
<p>&emsp;&emsp;以上的情况是非常简单的情景，但是实际情况下，我们总是要根据数据做动态合并的，而且实际情况中是不怎会合并两列的，英语列表头不一样，合并没意义。所以实际中我们总会合并一些特定行，主要就是哪些行数据相同时，我们就给他合并。注意啊，这个时候，我们对数据也是有要求的啊，就是内容相同的数据必须是相邻的行，因为跨行合并是在是没意义。那么，怎么合并呢，合并的方法还是上面那个，只不过我们要对数据做处理，记录每一行的数据相同的我们会给到标志，然后根据这个标志，来决定合并几行。。。</p>
<p>&emsp;&emsp;这里我还要解释一个情况，就是我们再合并第一列的数据相同的行后，我们再合并第二列的行，这个时候是在第一列行合并的基础上去做的，比如在第二列中相同的数据，但是在第一列中属于不同的合并部分，那么是不让合并的。</p>
<p style="color: red">&emsp;&emsp;主要看两个函数handleSpan和assembleData，数据是请求过来的，我们拿到数据之后，调用assembleData函数，然后渲染table的时候，table会自己根据handleSpan去渲染。</p>

```javascript
<template>
    <div>
      <Table :columns="columns" :data="data" :span-method="handleSpan"></Table>
    </div>
</template>
<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';
@Component({
  components: {

  }
})
export default class TestCase extends Vue {
  //表头列就不写了，一样的
  columns:Array<any> = [];
  //数据是请求过来的，也不写了啊
  data: Array<any> = [];

  //这里我们做前三列的行合并
  handleSpan({ row, column, rowIndex, columnIndex }) {
    //合并第一列的数据相同行
    if (columnIndex == 0) {
      let x = row.oneNum == 0 ? 0 : row.oneNum
      let y = row.oneNum == 0 ? 0 : 1
      return [x, y]
    }
    //合并第二列的数据相同行
    if (columnIndex == 1) {
      let x = row.twoNum == 0 ? 0 : row.twoNum
      let y = row.twoNum == 0 ? 0 : 1
      return [x, y]
    }
    //合并第三列的数据相同行
    if (columnIndex == 2) {
      let x = row.threeNum == 0 ? 0 : row.threeNum
      let y = row.threeNum == 0 ? 0 : 1
      return [x, y]
    }
  }

  //做行合并我们需要加标志（oneNum、twoNum、threeNum）
  assembleData(data) {
    //第一列
    for (var i = 0; i < data.length; i++) {
      //数据刚来时没有oneAlready的，所以一定会进来
      if (data[i].oneAlready !== 1) {
        if (data[i + 1]) {
          //这里加上标志oneNum用于待会判定合并几行
          data[i].oneNum = 1
          for (var a = i + 1; a < data.length; a++) {
            //这里的testPaperTemplate是表格第一列表头的字段
            if (data[i].testPaperTemplate === data[a].testPaperTemplate) {
              data[i].oneNum++
              data[a].oneNum = 0
              data[a].oneAlready = 1
            } else {
              break
            }
          }
        }
      }
    }
    //第二列
    for (var j = 0; j < data.length; j++) {
      if (data[j].oneNum > 1) {
        //这里我们依据oneNum做了判断，意思是我们第二列对的行合并要根据第一列的来
        for (var k = 0; k < data[j].oneNum; k++) {
          //twoAlready一样，不会对数据产生影响，只是用来做判断的
          if (data[j + k].twoAlready !== 1) {
            if (k + 1 < data[j].oneNum) {
              //这里加第二列的标志twoNum
              data[j + k].twoNum = 1
              for (var b = k + 1; b < data[j].oneNum; b++) {
                //这里的total是表头第二列的字段
                if (data[j + k].total === data[j + b].total) {
                  data[j + k].twoNum++
                  data[j + b].twoNum = 0
                  data[j + b].twoAlready = 1
                } else {
                  break
                }
              }
            }
          }
        }
      }
    }
    //第三列与第二列完全一致
    for (var j = 0; j < data.length; j++) {
      if (data[j].twoNum > 1) {
        for (var k = 0; k < data[j].twoNum; k++) {
          if (data[j + k].threeAlready !== 1) {
            if (k + 1 < data[j].twoNum) {
              data[j + k].threeNum = 1
              for (var b = k + 1; b < data[j].twoNum; b++) {
                if (data[j + k].jk_count === data[j + b].jk_count) {
                  data[j + k].threeNum++
                  data[j + b].threeNum = 0
                  data[j + b].threeAlready = 1
                } else {
                  break
                }
              }
            }
          }
        }
      }
    }
  }

  mounted () {

  }
}
<script>
```
<p>&emsp;&emsp;以上就是代码，那我们来看下这个合并后的效果吧，效果杠杠的！！</p>

![](/images/dMergeTable.jpg)


