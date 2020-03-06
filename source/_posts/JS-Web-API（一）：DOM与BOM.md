---
title: JS-Web-API（一）：DOM与BOM
date: 2020-03-06 19:49:44
thumbnail: https://i.picsum.photos/id/560/565/242.jpg
categories:
  - 程序世界
  - JavaScript
tag:
  - JavaScript
  - Web Api
  - DOM
  - BOM
---

## DOM
### DOM的本质

DOM的本质是从html文件解析出来的树状数据结构

### DOM节点操作

#### 获取DOM节点

```javascript
const div1 = document.getElementById('div1') // 获取一个元素
const divList = document.getElementsByTagName('div') // 获取一个元素集合
const containerList = document.getElementsByClassName('.container') // 集合
const pList = document.querySelectorAll('p') // 集合 
```
<!-- more -->
#### DOM节点的property

```javascript
const pList = document.querySelectorAll('p') // 获取一个元素集合
const p = pList[0] // 获取第一个元素

console.log(p.style.width) // 获取样式
p.style.width = '100px' // 修改样式

console.log(p.className) // 获取class
p.className = 'p1' // 修改class

// 获取nodeName和nodeType
console.log(p.nodeName)
console.log(p.nodeType)
```

#### DOM节点的attribute

```javascript
const pList = document.querySelectorAll('p') // 获取一个元素集合
const p = pList[0] // 获取第一个元素

p.getAttribute('data-name') // 获取data-name属性
p.setAttribute('data-name', 'abc') // 设置data-name属性的值为abc

p.getAttribute('style') // 获取style属性
p.setAttribute('style', 'font-size:30px;') // 设置style属性的值为font-size:30px;
```

#### DOM节点操作总结

- property：修改DOM对象属性，不会体现到html结构中
- attribute：修改html属性，会改变html结构
- 两者都有可能引起DOM重新渲染

### DOM结构操作

#### 新增/插入节点

```javascript
const div1 = document.getElementById('div1')
// 添加新节点
const p1 = document.createElement('p')
p1.innerHTML = 'this is p1'
div1.appendChild(p1) // 添加新创建的元素
// 移动已有节点，注意是移动！！！将DOM移动到指定节点中
const p2 = document.getElementById('p2')
div1.appendChild(p2)
```

#### 获取父/子元素
```javascript
const div1 = document.getElementById('div1')
// 添加新节点
const p1 = document.createElement('p')
p1.innerHTML = 'this is p1'
div1.appendChild(p1) // 添加新创建的元素
console.log(p1.parentNode) // 获取父元素
console.log(div1.childNodes) // 获取子元素列表
```

#### 删除子元素

```javascript
const div1 = document.getElementById('div1')
// 添加新节点
const p1 = document.createElement('p')
p1.innerHTML = 'this is p1'
div1.appendChild(p1) // 添加新创建的元素
div1.removeChild(div1.childNodes[0])
```

### DOM性能

- DOM操作非常“昂贵”，避免频繁操作DOM
- 对DOM查询做缓存，可以减少DOM操作
- 将频繁操作改为一次性操作

#### 对DOM查询做缓存
```javascript
// 不缓存DOM查询结果
for(let i = 0; i < document.getElementsByTagName('p').length; i++) {
    // 每次循环，都会计算length，频繁进行DOM查询
}

// 缓存DOM查询结果
const pList = document.getElementsByTagName('p')
const length = pList.length
for(let i = 0; i < length; i++) {
    // 缓存length，只进行一次DOM查询
}
```
#### 将频繁操作改为一次性操作
```javascript
const listNode = document.getElementById('list')

// 创建一个文档片段，此时还没有插入到DOM树中
const frag = document.createDocumentFragment()

// 执行插入
for(let i = 0; i < 10; i++) {
    const li = document.createElement('li')
    li.innerHTML = 'list item ' + i
    frag.appendChild(li)
}

// 都完成之后，再插入到DOM树中
listNode.appendChild(frag)
```

## BOM

### navigator和screen

```javascript
// navigator
const ua = navigator.userAgent
// 识别浏览器类型
const isChrome = ua.indexOf('Chrome')
console.log(isChrome)

// screen
console.log(screen.width) // 屏幕宽度
console.log(screen.height) // 屏幕高度
```

### location和history
```javascript
// location
// 拆解url各个部分
console.log(location.href) // 获取当前网址
console.log(location.protocol) // 获取当前网址协议
console.log(location.host) // 获取当前主机地址
console.log(location.search) // 获取当前网址中的参数
console.log(location.hash) // 获取当前网址中的哈希
console.log(location.pathname) // 获取当前网址中的路径

// history
history.back() // 后退
history.forward() // 前进
```
