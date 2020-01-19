---
title: Vue @mousemove实现拖动，鼠标移动过快拖动卡顿
date: 2020-01-14 17:59:45
thumbnail: https://picsum.photos/id/156/565/242
categories:
  - 程序世界
  - WEB前端
tags:
  - Vue
  - JavaScript
---

## 需求

使用 vue 实现滑动拼图验证码

## 踩到的坑

使用@mousemove 绑定事件拖拽速度过快有严重的卡顿
![未优化之前效果图](https://s2.ax1x.com/2020/01/19/19zkeU.gif)

<!--more-->

### 源代码

```html
<template>
  <div class="slider-verify">
    <div class="img">
      <img src="./101.jpg" class="big-img" />
      <img
        ref="block"
        src="./101.jpg"
        class="small-img"
        @mousemove="handleDragMove"
        @mousedown="handleDragStart"
        @mouseup="handleDragEnd"
      />
    </div>
    <div ref="sliderContainer" class="sliderContainer">
      <div ref="sliderMask" class="sliderMask">
        <div ref="slider" class="slider">
          <i
            class="el-icon-right"
            @mousemove="handleDragMove"
            @mousedown="handleDragStart"
            @mouseup="handleDragEnd"
          ></i>
        </div>
      </div>
      <span class="sliderText">向右滑动填充拼图</span>
    </div>
  </div>
</template>

<script>
  export default {
    name: "SliderVerify",
    props: {
      width: {
        type: Number,
        default: 310
      }
    },
    data() {
      return {
        isMouseDown: false,
        originX: 0,
        originY: 0,
        slider: null,
        sliderMask: null,
        sliderContainer: null,
        block: null
      };
    },
    created() {
      this.$nextTick(() => {
        this.slider = this.$refs.slider;
        this.sliderMask = this.$refs.sliderMask;
        this.sliderContainer = this.$refs.sliderContainer;
        this.block = this.$refs.block;
      });
    },
    methods: {
      handleDragMove(e) {
        if (!this.isMouseDown) return false;
        const w = this.width;
        // 获取拖拽移动的距离
        const eventX = e.clientX || e.touches[0].clientX;
        const moveX = eventX - this.originX;
        if (moveX < 0 || moveX + 40 >= w) return false;
        this.slider.style.left = moveX + "px";
        this.block.style.left = moveX + "px";
        this.sliderMask.style.width = moveX + "px";
      },
      handleDragStart(e) {
        // 获取拖拽起始位置坐标
        this.originX = e.clientX || e.touches[0].clientX;
        this.originY = e.clientY || e.touches[0].clientY;
        this.isMouseDown = true;
      },
      handleDragEnd(e) {
        if (!this.isMouseDown) return false;
        this.isMouseDown = false;
        const eventX = e.clientX || e.changedTouches[0].clientX;
        if (eventX === this.originX) return false;
      }
    }
  };
</script>
...
```

## 解决方案

使用 JS 原生事件替代 Vue v-on 事件

### 优化后代码

```html
<template>
  <div class="slider-verify">
    <div class="img">
      <img src="./101.jpg" class="big-img" />
      <img
        ref="block"
        src="./101.jpg"
        class="small-img"
        @mousedown="handleDragStart"
      />
    </div>
    <div ref="sliderContainer" class="sliderContainer">
      <div ref="sliderMask" class="sliderMask">
        <div ref="slider" class="slider">
          <i class="el-icon-right" @mousedown="handleDragStart"></i>
        </div>
      </div>
      <span class="sliderText">向右滑动填充拼图</span>
    </div>
  </div>
</template>

<script>
  export default {
    name: "SliderVerify",
    props: {
      width: {
        type: Number,
        default: 310
      }
    },
    data() {
      return {
        isMouseDown: false,
        originX: 0,
        originY: 0,
        slider: null,
        sliderMask: null,
        sliderContainer: null,
        block: null
      };
    },
    created() {
      this.$nextTick(() => {
        this.slider = this.$refs.slider;
        this.sliderMask = this.$refs.sliderMask;
        this.sliderContainer = this.$refs.sliderContainer;
        this.block = this.$refs.block;
      });
    },
    methods: {
      handleDragStart(e) {
        // 获取拖拽起始位置坐标
        this.originX = e.clientX || e.touches[0].clientX;
        this.originY = e.clientY || e.touches[0].clientY;
        this.isMouseDown = true;
        document.onmousemove = ev => {
          if (!this.isMouseDown) return false;
          const w = this.width;
          // 获取拖拽移动的距离
          const eventX = ev.clientX || ev.touches[0].clientX;
          const moveX = eventX - this.originX;
          if (moveX < 0 || moveX + 40 >= w) return false;
          this.slider.style.left = moveX + "px";
          this.block.style.left = moveX + "px";
          this.sliderMask.style.width = moveX + "px";
        };
        document.onmouseup = ev => {
          if (!this.isMouseDown) return false;
          this.isMouseDown = false;
          const eventX = ev.clientX || ev.changedTouches[0].clientX;
          if (eventX === this.originX) return false;
        };
      }
    }
  };
</script>
...
```

### 优化后效果

![优化后效果](https://s2.ax1x.com/2020/01/19/19z6Yj.gif)

## Ending......
