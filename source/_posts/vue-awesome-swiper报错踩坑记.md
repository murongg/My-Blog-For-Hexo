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
![window is not defined](https://upload-images.jianshu.io/upload_images/15101357-9cb89cba7b40fcbd.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

### 原因：

Nuxt 在服务端渲染时找不到 window

<!--more-->

### 查询官网文档：![mount with ssr](https://upload-images.jianshu.io/upload_images/15921555-80453c695ade306e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![SSR](https://upload-images.jianshu.io/upload_images/15921555-4a0148911687516f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 解决办法：

在 nuxt 的 plugins 目录下新建 vue-awesome-swiper.js 文件，代码如下：![vue-awesome-swiper.js](https://upload-images.jianshu.io/upload_images/15921555-2cbfcb935b027c50.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```javascript
import Vue from "vue";
import "swiper/dist/css/swiper.css";
if (process.browser) {
  const VueAwesomeSwiper = require("vue-awesome-swiper/dist/ssr");
  Vue.use(VueAwesomeSwiper);
}
```

修改 nuxt.config.js 的 plugins 配置：
![nuxt.config.js](https://upload-images.jianshu.io/upload_images/15921555-31a870eeaabca83a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

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
