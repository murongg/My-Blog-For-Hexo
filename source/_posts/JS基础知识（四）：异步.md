---
title: JS基础知识（四）：异步
date: 2020-03-06 17:46:26
thumbnail: https://i.picsum.photos/id/175/565/242.jpg
categories:
  - 程序世界
  - JavaScript
tag:
  - JavaScript
  - JavaScript基础
  - 异步
---

## 同步和异步的区别
### 单线程和异步

- JS是单线程语言，只能同时做一件事
- 浏览器和nodejs以支持js启动进程，如Web Worker
- JS和DOM渲染共用同一个现成，因为JS可修改DOM结构
- 遇到等待（网络请求，定时任务）不能卡住
- 所以需要异步
- 异步基于callback函数形式

<!-- more -->

### 区别
- 异步基于JS是单线程语言
- 异步不会阻塞代码执行
- 同步会阻塞代码执行

## 应用场景
- 网络请求，如ajax图片加载
- 定时任务，如setTimeout

```javascript
// ajax
console.log('start')
$.get('./data1.json', function (data1) {
    console.log(data1)
})
console.log('end')

// 打印结果 1、start 2、end 3、data1
```

```javascript
// 图片加载
console.log('start')

let img = document.createElement('img')
img.onload = function () {
    console.log('loaded')
}
img.src = '/xxx.png'
console.log('end')

// 打印结果 1、start 2、end 3、loaded
```

```javascript
// setTimeout
console.log(100)
setTimeout(function () {
    console.log(200)
}, 1000)
console.log(300)
// 打印结果 1、100 2、300 3、200
```
```javascript
// setInterval
console.log(100)
setInterval(function () {
    console.log(200)
}, 1000)
console.log(300)
// 打印结果 1、100 2、300 3、200
```

## Promise
> Promise解决了“远古时期”回调地狱的问题
### Callback hell（回调地狱）

```javascript
// 获取第一份数据
$.get(url1 (data1) => {
    console.log(data1)
    // 获取第二份数据
    $.get(url2, (data2) => {
        console.log(data2)
         // 获取第三份数据
         $.get(url3, (data3) => {
            console.log(data3)
            ...
        })
    })
})
```
### Promise
```javascript
// 使用Promise封装异步请求

function getData(url) {
    return new Promise((resolve, reject) => {
        $.ajax({
            url,
            success(data) {
                resolve(data)
            },
            error(err) {
                reject(err)
            }
        })
    })
}
// 使用
getData(url1).then(data1 => {
    console.log(data1)
    return getData(url2)
}).then(data2 => {
    console.log(data2)
    return getData(url3)
}).then(data3 => {
    console.log(data3)
}).catch(err => {
    console.error(err)
})
```