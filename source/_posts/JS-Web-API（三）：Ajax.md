---
title: JS-Web-API（三）：Ajax
date: 2020-03-08 18:02:45
thumbnail: https://i.picsum.photos/id/112/565/242.jpg
categories:
  - 程序世界
  - JavaScript
tag:
  - JavaScript
  - Web Api
  - Ajax
---

## XMLHttpRequest

### GET请求

```javascript
// 初始化实例
const xhr = new XMLHttpRequest()
// 第三个参数true则为异步
xhr.open('GET','url', true)
xhr.onreadystatechange = function () {
    // 这里函数异步执行
    if (xhr.readyState === 4) {
        if (xhr.statusCode === 200) {
            console.log(xhr.responseText)
        } else {
            console.log('请求失败')
        }
    }
}
xhr.send()
```

<!-- more -->

### POST请求
```javascript
// 初始化实例
const xhr = new XMLHttpRequest()
// 第三个参数true则为异步
xhr.open('POST','url', true)
xhr.onreadystatechange = function () {
    // 这里函数异步执行
    if (xhr.readyState === 4) {
        if (xhr.statusCode === 200) {
            console.log(xhr.responseText)
        } else {
            console.log('请求失败')
        }
    }
}
const postData = JSON.stringify({
    a: 1
})
// 发送数据
xhr.send(postData)
```

### readyState

- 0 - 未初始化（还没调用send()方法）
- 1 - 载入（已调用send()方法，正在发送请求）
- 2 - 载入完成（send()方法执行完成，已经接收到全部相应内容）
- 3 - 交互（正在解析相应内容）
- 4 - 完成（相应内容解析完成，可以再客户端调用）

### status

- 1xx - 临时响应并需要请求者继续执行操作
- 2xx - 成功处理请求，比如200
- 3xx - 重定向，浏览器直接跳转，如301 302 304
- 4xx - 客户端请求错误，如404 403
- 5xx - 服务器错误  

## 同源策略与跨域

### 同源策略

- ajax请求时， **浏览器要求** 当前网页和server必须同源（安全）
- 同源：协议、域名、端口， **三者必须一致** 
- 加载图片、css、js文件可无视同源策略

### 跨域

- 所有的跨域，都必须经过server端允许与配合
- 未经server端允许就实现跨域，说明浏览器有漏洞，危险信号

## jsonp和cors

### jsonp

- script标签可绕过跨域限制
- 服务端可以任意动态拼接数据返回
- script标签可以获得跨域的数据，只要服务端愿意返回

```html
<script src="xxx/getData.js"></script>
<!--返回一个方法 callback({a:1,b:2}) -->

<script>
    // 调用返回的方法
    window.callback = function(data) {
        // 这里得到跨域的数据
        console.log(data)
    }
</script>
```

### cors

> 服务器设置http header

```javascript
// 第二个参数填写允许跨域的域名，不建议直接填写 “*”
response.setHeader("Access-Control-Allow-Origin", "http://localhost:8011");
response.setHeader("Access-Control-Allow-Headers", "X-Requested-With");
response.setHeader("Access-Control-Allow-Methods", "PUT,POST,GET,DELETE,OPTIONS");

// 接收跨域的cookie
response.setHeader("Access-Control-Allow-Credentials", "true")
```

## 使用Promise封装一个简易Ajax

```javascript
function Request(url, method, data) {
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest()
        xhr.open(method, url, true) 
        xhr.onreadystatechange = function() {
            if (xhr.readyState === 4) {
                if (xhr.status === 200) {
                    resolve(JSON.parse(xhr.responseText))
                } else if(xhr.status === 404) {
                    reject(new Error('404 not Found'))
                } else {
                    reject(JSON.parse(xhr.responseText))
                }
            }
        }
        xhr.send(JSON.stringify(data))
    })
}
```