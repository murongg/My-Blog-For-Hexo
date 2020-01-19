---
title: vue-awesome-swiper报错踩坑记
date: 2020-01-14 18:30:31
thumbnail: https://picsum.photos/id/134/565/242
categories:
  - 程序世界
  - WEB前端
tags:
  - Vue
  - Nuxt
---

nuxt.js 引入 vue-awesome-swiper 后，控制台报错**window is not defined**
![window is not defined](https://s2.ax1x.com/2020/01/19/1CS9tH.jpg)

### 原因：

Nuxt 在服务端渲染时找不到 window

<!--more-->

### 查询官网文档：

![mount with ssr](https://s2.ax1x.com/2020/01/19/1CSF1I.png)

![SSR](https://s2.ax1x.com/2020/01/19/1CSeHS.png)

### 解决办法：

在 nuxt 的 plugins 目录下新建 vue-awesome-swiper.js 文件，代码如下：![vue-awesome-swiper.js](https://s2.ax1x.com/2020/01/19/1CSl3n.png)

```javascript
import Vue from "vue";
import "swiper/dist/css/swiper.css";
if (process.browser) {
  const VueAwesomeSwiper = require("vue-awesome-swiper/dist/ssr");
  Vue.use(VueAwesomeSwiper);
}
```

修改 nuxt.config.js 的 plugins 配置：
![nuxt.config.js](https://s2.ax1x.com/2020/01/19/1CSGuV.png)

在 nuxt.config.js 的 plugins 里加入：

```javascript
  {
    src: "@/plugins/vue-awesome-swiper",
    ssr: false
  }
```

IndexBanner.vue 的代码为：

```html
<div v-swiper:mySwiper="swiperOption" class="swiper">
  <div class="swiper-wrapper">
    <div class="swiper-slide">1</div>
    <div class="swiper-slide">2</div>
  </div>
  <div class="swiper-pagination"></div>
</div>
```
