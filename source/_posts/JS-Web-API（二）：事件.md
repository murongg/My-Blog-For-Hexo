---
title: JS-Web-API（二）：事件
date: 2020-03-08 18:01:05
thumbnail: https://i.picsum.photos/id/115/565/242.jpg
categories:
  - 程序世界
  - JavaScript
tag:
  - JavaScript
  - Web Api
  - 事件
  - 事件代理
  - 事件冒泡
---


## 事件绑定

```javascript
const btn = document.getElementById('btn1')
btn.addEventListener('click', event => {
    console.log('clicked')
})
```
```javascript
// 通用的绑定函数
function bindEvent(elem, type, fn) {
    elem.addEventListener(type, fn)
}

const a = document.getElementById('link1')
bindEvent(a, 'click', e => {
    e.preventDefault() // 阻止默认行为
    alert('clicked')
})
```
<!-- more -->
## 事件冒泡

> 事件冒泡会顺着出发元素由DOM结构从内至外进行事件响应

```html
<body>
    <div id="div1">
        <p id="p1">激活</div>
        <p id="p2">取消</div>
    </div>
    <div id="div2">
        <p id="p3">取消</div>
        <p id="p4">取消</div>
    </div>
</body>

<script>
    const body = document.body
    const p1 = document.getElementById('p1')
    
    // 引用上面封装的方法
    bindEvent(p1, 'click', e => {
        console.log('激活')
    })
    
    bindEvent(body, 'click', e => {
        console.log('取消')
    })
    
    // 点击p1区域输出
    // 激活   取消
    
    // 点击body除p1区域输出
    // 取消
    
    // 阻止事件冒泡
    bindEvent(p1, 'click', e => {
        // 阻止默认行为
        e.stopPropagation()
        console.log('激活')
    })
    
    // 点击p1区域输出
    // 激活
    
    // 点击body除p1区域输出
    // 取消
</script>
```

## 事件代理

- 代码简洁
- 减少浏览器内存占用
- 不要滥用

### 代理绑定

```html
<!--在不确定元素数量的情况下使用事件代理-->
<body>
    <div id="div1">
       <a href="#">a1</a>
       <a href="#">a2</a>
       <a href="#">a3</a>
       ....
    </div>
</body>

<script>
    const div1 = document.getElementById('div1')
    // 事件代理
    bindEvent(div1, 'click', e => {
        // 阻止默认行为
        e.stopPropagation()
        const target = event.target
        if (target.nodeName === 'A') {
            console.log(target.innerHTML)
        }
    })
    // 点击a1 打印a1
    // 点击a2 打印a2
    // ....
</script>
```

### 更全面的通用绑定函数

```javascript

function bindEvent(elem, type, selector, fn) {
    // 如果第四个参数不存在
    // 匹配bindEvent(div1, 'click', function(e) {})的情况
    if (fn == null) {
       fn = selector
       selector = null
    }
    
    elem.addEventListener(type, event => {
        const target = event.target
        // 代理绑定
        if(selector) {
            // 判断当前target是否为指定的元素
            if (target.matches(selector)) {
                // 绑定this
                fn.call(target, event)
            }
        } else {
            // 普通绑定
            // 绑定this
            fn.call(target, event)
        }
    })
}

```