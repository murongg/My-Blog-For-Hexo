---
title: Vue侦听器watch获取this报undefined
date: 2020-01-14 18:31:15
thumbnail: https://i.picsum.photos/id/155/565/242.jpg
categories:
  - 程序世界
  - WEB前端
tags:
  - Vue
---

## 错误代码

```javascript
watch: {
  data: {
    handler: (newVal) => {
      this.info = newVal
    },
    deep: true
  }
},
```

## 修正后代码

```javascript
watch: {
  data: {
    handler: function (newVal) {
      this.info = newVal
    },
    deep: true
  }
},
```

## 原因

![](https://upload-images.jianshu.io/upload_images/15921555-497f8f2609da3308.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

总的来说还是 this 指向问题，可以查看[普通函数与箭头函数 this 指向问题](https://www.cnblogs.com/qdlhj/p/9877881.html)
