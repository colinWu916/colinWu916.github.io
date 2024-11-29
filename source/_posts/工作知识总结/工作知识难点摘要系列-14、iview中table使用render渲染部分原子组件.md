---
title: 十四、iview中table使用render渲染部分原子组件
tags: 工作知识总结
categories: 热爱我的工作
comments: true
---

# 一、在线浏览word并下载

## 1、简单介绍
<p>&emsp;&emsp;最近遇到了很多场景就是要在table里渲染组件（比如select、switch、radio），然后直接操作控制table的行数据以及动态显示，这里记录一下，这个以后应该能经常用到。。。。</p>

## 2、组件渲染
### 2.1、switch和radio组件的渲染
<p>&emsp;&emsp;简单介绍一下场景，需要一个表格，表格有四列，第三列放switch,第四列放radio，同时第三列的switch关闭的同时禁用radio，来吧看下，当然不懂的小伙伴可以先看下iview官网了解一下table组件。。还有一个很重要的，大家看第零列，在用render的时候我们做了条件判断并且用了循环去渲染option，其次我们渲染了div这种不是表单标签的普通标签，再写属性的时候是用domprops包裹的！！！！！</p>

```javascript
<template>
  <div class="container">
    <Table :data="tableData" :columns="tableColumns" :border="true"><Table>
  </div>
</template>
<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';
@Component({
  components: {

  }
})
export default class TestCase extends Vue {
// 比较有意思的是，我们在渲染radio的时候写的是Radio，但是渲染switch用的是i-switch,这是iview的组件写法，不知道为啥写成Switch就是不行。然后还有一个需要注意的地方就是radio渲染是先渲染了一个RadioGroup，然后在RadioGroup里再渲染两个radio。注意列3的key是isShow,列4的key是isrequired。
public stNodeColumns = [
    { 
      title: '列0',
      key: 'stNode',
      align: 'left',
      width: 140,
      render: (h, params) => {
        if (Array.isArray(params.row.stNode)) {
          return h('Select', {
            props: {
              value: params.row.stSelectNode,
              multiple: false,
              disabled: flag
            },
            on: {
              'on-change': (e) => {
                this.setSelectNode(params.row, e);
              }
            }
          },
          params.row.stNode.map(val => {
            return h('Option', {
              props: {
                value: val.value
              }
            }, val.label)
          }))
        } else {
          return h('div', {
          domProps: {
            innerHTML: params.row.stNode
          }
        })
        }
      }
    },
    {
      title: '列1',
      key: 'col-one',
      align: 'left',
      width: 160
    },
    {
      title: '列2',
      key: 'col-two',
      align: 'left'
    },
    {
      title: '列3',
      key: 'isShow',
      align: 'center',
      ellipsis: true,
      tooltip: true,
      render: (h, params) => {
        return h('i-switch', {
          props: {
            value: params.row.isShow
          },
          on: {
            'on-change': (e) => {
              this.changeStRequired(params.row,e)
            }
          }
        })
      }
    },
    {
      title: '列4',
      key: 'isRequired',
      width: 150,
      align: 'left',
      ellipsis: true,
      tooltip: true,
      render: (h, params) => {
        return h('RadioGroup', {
          props: {
            value: this.issirequie(params.row)
          },
          on: {
            'on-change': (e) => {
              this.changeRequired(params.row,e);
            }
          }
        },
        [h('Radio', {
          props: {
            label: '必填',
            disabled: !params.row.isShow
          }
        }),
        h('Radio', {
          props: {
            label: '选填',
            disabled: !params.row.isShow
          }
        })
        ])
      }
    }
  ]

  // 通过传过来的数据控制radioGroup的值，也同时控制了显示
  public issirequie(row: any) {
    if (row.isRequired === 0 && row.isShow) {
      return '必填';
    } else if (row.isRequired === 1 && row.isShow) {
      return '选填';
    } else {
      return null;
    }
  }

  // 改变radio选项时触发的事件
  public changeRequired(row,e) {
    if(e === '必填') {
      row.isRequired = 0
      this.tableData[row._index].isRequired = 0;
    } else if('选填') {
      row.isRequired = 1
      this.tableData[row._index].isRequired = 1;
    }
  }

  // 改变switch时，同时会控制radio是否可用
  public changeStRequired(row,e) {
    row.isRequired = null;
    row.isShow = !row.isShow;
    this.tableData[row._index].isShow = row.isShow;
  }
}
<script>
```

## 注意：需要注意的是，我们再每次触发三个函数的时候同时都更新了tableData的值，应为这是table绑定的行数据他更新后，列表也会随时更新！！