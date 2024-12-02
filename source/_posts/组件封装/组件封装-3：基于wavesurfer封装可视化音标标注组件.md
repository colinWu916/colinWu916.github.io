---
title: 三、基于wavesurfer封装可视化音标标注组件
date: 2024-10-02 08:56:28
author: 吴俊杰
top: false
keywords:
  - wavesurfer
  - regions
  - 音频剪辑
summary: 这个比较难，需要实现一个模拟在线音频剪辑的组件，展示波形，同时记录截切的选区，最后把时间传给后端后端实现音频剪辑......
categories: 
  - 组件封装
tags:
  - wavesurfer
  - regions
  - 音频剪辑
---



# 一、基于wavesurfer，regions封装的可视化音标标注控件

## 1、简单介绍
<p>&emsp;&emsp;基于wavesurfer，regions 封装的可视化音标标注控件。。。。这个案例是公司大佬写的博客，我抄过来，免得以后链接失效，哈哈哈哈哈啊哈。。。。</p>

## 2、基本流程
### 2.1、先简单看下效果
<p>1、将一段声频以波形展示在页面上，支持播放/暂停、重放、停止、点击跳转播放</p>

![](/images/cp/3_1.gif)

<p>2、支持渲染区域，支持用户手动选择区域和删除区域，支持拖动区域和调整区域大小；当操作区域时，最好能实时循环播放区域</p>

![](/images/cp/3_2.gif)

<p>3、点击区域时循环播放本区域，点击区域外时正常播放至结束</p>

![](/images/cp/3_3.gif)

### 2.2、Vue实现

```javascript
<template>
  <div>
    <div ref="waveformRef"></div>
    <div ref="waveTimelineRef"></div>
    <el-button type="primary" @click="wavesurfer.skip(-3)">后退</el-button>
    <el-button type="primary" @click="wavesurfer.playPause()">
      <i class="el-icon-video-play" />
      播放 / 暂停
    </el-button> <el-button type="primary" @click="wavesurfer.skip(3)">前进</el-button>
    <el-button type="primary" @click="rebroadcast">重放</el-button>
    <el-button type="primary" @click="handleStop">停止</el-button>
    <el-button @click="getRegions">打印区域</el-button>
  </div>
</template>
 
<script>
import WaveSurfer from 'wavesurfer.js'
import Region from 'wavesurfer.js/dist/plugin/wavesurfer.regions'
import Cursor from 'wavesurfer.js/dist/plugin/wavesurfer.cursor'
import Timeline from 'wavesurfer.js/dist/plugin/wavesurfer.timeline'

export default {
  props: ['voiceSrc'],
  data() {
    return { wavesurfer: null }
  },
  mounted() {
    this.init()
  },
  beforeDestroy() {
    this.wavesurfer && this.wavesurfer.destroy()
    this.wavesurfer = null
  },
  methods: {
    init() {
      const container = this.$refs.waveformRef
      container.addEventListener('click', () => {
        console.log('点击容器')
        this.clearLoop()
      })
      this.wavesurfer = WaveSurfer.create({
        container, // 容器，唯一一个必须参数
        cursorColor: 'red', // 音频光标颜色
        waveColor: '#ddd', // 波形颜色
        progressColor: '#bbb', // 已完成播放的波形颜色 当progressColor和waveColor相同时，完全不渲染进度波
        backend: 'MediaElement',
        autoCenter: false,
        plugins: [
          Region.create({
            // regionsMinLength: 1.5,
            regions: [
              { start: 1, end: 3, color: 'hsla(400, 100%, 30%, 0.5)' },
              { start: 5, end: 7, color: 'hsla(200, 50%, 70%, 0.4)' }
            ],
            dragSelection: true,
          }),
          Cursor.create({
            showTime: true,
            opacity: 1,
            customShowTimeStyle: { 'background-color': '#000', color: '#fff', padding: '2px', 'font-size': '10px' }
          }),
          Timeline.create({ container: this.$refs.waveTimelineRef })
        ]
      })

      this.wavesurfer.load(this.voiceSrc) // 加载音频url

      // 点击区域
      this.wavesurfer.on('region-click', (region) => {
        const timer = setTimeout(() => {
          console.log('定时器')
          region.playLoop()
        })
        this.$once('hook:beforeDestroy', () => {
          clearTimeout(timer)
          timer = null
        })
      })

      // 完成拖动或者完成大小调整时触发
      this.wavesurfer.on('region-update-end', (region) => {
        region.playLoop() // 循环播放选中区域
        this.createDeleteButton(region)
      })

      this.wavesurfer.on('ready', () => {
        // 为区域追加一个删除按钮
        const regionList = Object.values(this.wavesurfer.regions.list)
        for (const region of regionList) {
          this.createDeleteButton(region)
        }
      })
    },
    // 格式化时间
    formatTime(seconds) {
      seconds = Number(seconds)
      const minutes = Math.floor(seconds / 60)
      seconds = seconds % 60
      const secondsStr = Math.round(seconds).toString()
      secondsStr = seconds.toFixed(2)
      if (minutes > 0) {
        return `${minutes < 10 ? '0' + minutes : minutes}:${seconds < 10 ? '0' + secondsStr : secondsStr}`
      }
      return `00:${seconds < 10 ? '0' + secondsStr : secondsStr}`
    },
    // 给区域创建删除按钮
    createDeleteButton(region) {
      if (!region.hasDeleteButton) {
        const deleteButton = region.element.appendChild(document.createElement('button'))
        deleteButton.innerText = '删除'
        deleteButton.addEventListener('click', (e) => {
          e.stopPropagation()
          this.$confirm('确认删除此区域嘛?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => { region.remove() }).catch(() => { })
        })
        const css = { float: 'right', position: 'relative', cursor: 'pointer', color: 'red' }
        region.style(deleteButton, css)
        region.hasDeleteButton = true
      }
    },
    // 获取区域列表
    getRegions() {
      const regionList = Object.values(this.wavesurfer.regions.list)
      console.log(regionList)
    },
    // 重放
    rebroadcast() {
      this.clearLoop()
      this.wavesurfer.play(0)
    },
    // 停止
    handleStop() {
      this.clearLoop()
      this.wavesurfer.stop()
    },
    // 设置每个区域的loop为false
    clearLoop() {
      const regionList = Object.values(this.wavesurfer.regions.list)
      for (const item of regionList) item.loop = false
    }
  }
}
</script>
```

### 2.3、重点描述
#### 2.3.1、实例方法
<p>&emsp;&emsp;playPause 暂停时播放，播放时暂停；play(0) 从0开始播放；stop() 停止；skip() 正数为前进，负数为后退！</p>

#### 2.3.2、区域的删除按钮怎么添加的
<p>&emsp;&emsp;createDeleteButton函数用于创建button按钮，region.element可以用来appendChild节点；监听ready事件，这里可以获取到已有的区域列表，循环添加按钮；新添加的区域，会触发region-update-end事件，回调函数的参数是region，这里可以再次调用createDeleteButton函数。</p>

#### 2.3.3、点击区域进行循环播放，操作区域位置和大小时也会进行循环播放
<p>&emsp;&emsp;调用region.playLoop()即可！</p>

#### 2.3.4、点击区域时循环播放本区域，点击区域外时正常播放至结束
<p>&emsp;&emsp;clearLoop函数用于将每个区域中的loop设置为false，playLoop方法会将当前区域的loop设置为true；当点击区域外时，我在container的click事件回调中调用clearLoop，这样就可以正常播放至结束；当点击区域时，在定时器中调用playLoop方法，便又可以循环播放本区域。</p>

### 2.4、补充功能：当拖动区域或调整区域大小时，重叠部分自动吸附
#### 2.4.1、先看效果
![](/images/cp/3_4.gif)

#### 2.4.2、代码实现
<p>&emsp;&emsp;还是在region-update-end事件中处理！</p>

```javascript
// 完成拖动或者完成大小调整时触发
this.wavesurfer.on('region-update-end', (region) => {
  // region.playLoop() // 循环播放选中区域
  this.createDeleteButton(region);

  const { prevElement, nextElement, prevRegionId, nextRegionId } = this.getPrevAndNextElement(region) // 获取相邻的两个节点
  if (prevElement && prevElement.className === 'wavesurfer-region') { // 和前一个dom对齐
    const prevRegion = this.getRegion(prevRegionId)
    if (region.start < prevRegion.end) {
      prevRegion.update({ start: prevRegion.start, end: region.start })
    }
  }
  
  if (nextElement && nextElement.className === 'wavesurfer-region') { // 和后一个dom对齐
    const nextRegion = this.getRegion(nextRegionId)
    if (region.end > nextRegion.start) {
      nextRegion.update({ start: region.end, end: nextRegion.end })
    }
  }
})


// 根据区域id，获取区域实例
getRegion(id) {
  return this.wavesurfer.regions.list[id]
},
// 获取当前region的上一个region和下一个region
getPrevAndNextElement(currentRegion) {
  const regionList = Object.entries(this.wavesurfer.regions.list)
  const prevList = [], nextList = []
  for (const [key, region] of regionList) {
    if (key !== currentRegion.id) {
      if (region.start < currentRegion.start) {
        prevList.push({ key, val: currentRegion.start - region.start })
      } else {
        nextList.push({ key, val: region.end - currentRegion.end })
      }
    }
  }
  const prevListSort = this.sortArr(prevList, 'val'), nextListSort = this.sortArr(nextList, 'val') // 排序后的prevList和nextList
  const prevRegionId = prevListSort ? prevListSort[0].key : null
  const nextRegionId = nextListSort ? nextListSort[0].key : null
  const regionDOMList = Array.from(document.querySelectorAll('.wavesurfer-region'))
  const prevElement = regionDOMList.find(regionDOM => regionDOM.getAttribute('data-id') === prevRegionId)
  const nextElement = regionDOMList.find(regionDOM => regionDOM.getAttribute('data-id') === nextRegionId)
  return { prevRegionId, nextRegionId, prevElement, nextElement }
},
// 数组对象根据指定属性值进行升序排序
sortArr(list, property) {
  if (list.length) {
    return [...list].sort((a, b) => a[property] - b[property])
  } else {
    return null
  }
},

```

### 注意：需要提前引入wavesurfer、wavesurfer.regions 两个文件