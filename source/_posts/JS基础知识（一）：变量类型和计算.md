---
title: JS基础知识（一）：变量类型和计算
thumbnail: https://i.picsum.photos/id/456/565/242.jpg
date: 2020-02-27 19:49:15
categories:
  - 程序世界
  - JavaScript
tag:
  - JavaScript
  - JavaScript基础
  - 变量
---

## 值类型和引用类型
### 值类型
#### 值类型示例
```javascript
// 值类型
let a = 100
let b = a
a = 200
console.log(b) // 100
```
#### 值类型在内存中的存储方式
> 值类型一般都是在栈中存储一个值

<!--more-->
<div style="display:flex;justify-content: space-between;">
  <table style="width:30%;text-align:center;">
    <thead>
      <tr align="center">
        <th colspan="2" style="background:#BCC6CD;">栈</th>
      </tr>
    </thead>
    <tr align="center">
      <td>key</td>
      <td>value</td>
    </tr>
    <tr align="center">
      <td>a</td>
      <td>100</td>
    </tr>
    <tr align="center">
      <td>...</td>
      <td>...</td>
    </tr>
  </table>
  <table style="width:30%">
    <thead>
      <tr align="center">
        <th colspan="2" style="background:#BCC6CD;">栈</th>
      </tr>
    </thead>
    <tr align="center">
      <td>key</td>
      <td>value</td>
    </tr>
    <tr align="center">
      <td>a</td>
      <td>100</td>
    </tr>
    <tr align="center">
      <td>b</td>
      <td>100</td>
    </tr>
  </table>
  <table style="width:30%">
    <thead>
      <tr align="center">
        <th colspan="2" style="background:#BCC6CD;">栈</th>
      </tr>
    </thead>
    <tr align="center">
      <td>key</td>
      <td>value</td>
    </tr>
    <tr align="center">
      <td>a</td>
      <td>200</td>
    </tr>
    <tr align="center">
      <td>b</td>
      <td>100</td>
    </tr>
  </table>
</div>

#### 常见值类型
```javascript
// const意为定义一个常量
// const定义的变量必须赋值，不然会报错
// 这里使用let定义a变量
let a // undefined
const s = 'abc'
const n = 100
const b = true
const s = Symbol('s')
```
### 引用类型
#### 引用类型示例
```javascript
// 引用类型
let a = { age: 20 }
let b = a
b.age = 21
console.log(a.age) // 21
```

#### 引用类型在内存中的存储方式

> 变量在计算机中存储时，栈和堆是同时存在的，栈是从上至下累加，堆是从下至上累加。  
引用类型会在堆中申请一个内存地址，将变量的值存储在堆里。  
栈中存储的不在是一个值，而是对应堆的内存地址。  
为什么引用类型不将值直接存储在栈中？原因还是性能问题，引用类型值的占用空间比值类型大的多，无论是存储还是引用，都会导致速度非常慢。

<div style="display:flex;justify-content: space-between;margin-bottom: 20px;">
  <table style="width:30%;">
    <thead>
      <tr align="center">
        <th colspan="2" style="background:#BCC6CD;">栈</th>
      </tr>
    </thead>
    <tr align="center">
      <td>key</td>
      <td>value</td>
    </tr>
    <tr align="center">
      <td>a</td>
      <td>内存地址1</td>
    </tr>
    <tr align="center">
      <td>...</td>
      <td>...</td>
    </tr>
  </table>
  <table style="width:30%;">
    <thead>
      <tr align="center">
        <th colspan="2" style="background:#BCC6CD;">栈</th>
      </tr>
    </thead>
    <tr align="center">
      <td>key</td>
      <td>value</td>
    </tr>
    <tr align="center">
      <td>a</td>
      <td>内存地址1</td>
    </tr>
    <tr align="center">
      <td>b</td>
      <td>内存地址1</td>
    </tr>
  </table>
  <table style="width:30%;">
    <thead>
      <tr align="center">
        <th colspan="2" style="background:#BCC6CD;">栈</th>
      </tr>
    </thead>
    <tr align="center">
      <td>key</td>
      <td>value</td>
    </tr>
    <tr align="center">
      <td>a</td>
      <td>内存地址1</td>
    </tr>
    <tr align="center">
      <td>b</td>
      <td>内存地址1</td>
    </tr>
  </table>
</div>
<div style="display:flex;justify-content: space-between;">
  <table style="width:30%;">
     <tr align="center">
      <td>...</td>
      <td>...</td>
    </tr>
    <tr align="center">
      <td>内存地址1</td>
      <td>{age:20}</td>
    </tr>
    <tr align="center">
      <td>key</td>
      <td>value</td>
    </tr>
    <tr align="center">
      <th colspan="2" style="background:#EFE4B0">堆</th>
    </tr>
  </table>
   <table style="width:30%;">
     <tr align="center">
      <td>...</td>
      <td>...</td>
    </tr>
    <tr align="center">
      <td>内存地址1</td>
      <td>{age:20}</td>
    </tr>
    <tr align="center">
      <td>key</td>
      <td>value</td>
    </tr>
    <tr align="center">
      <th colspan="2" style="background:#EFE4B0">堆</th>
    </tr>
  </table>
   <table style="width:30%;">
     <tr align="center">
      <td>...</td>
      <td>...</td>
    </tr>
    <tr align="center">
      <td>内存地址1</td>
      <td>{age:21}</td>
    </tr>
    <tr align="center">
      <td>key</td>
      <td>value</td>
    </tr>
    <tr align="center">
      <th colspan="2" style="background:#EFE4B0">堆</th>
    </tr>
  </table>
</div>

#### 常见引用类型

```javascript
const obj = { x: 100 }
const arr = ['a','b', 'c']
const n = null // 特殊引用类型，指针指向为空地址

// 特殊引用类型，但不用于存储数据，所以没有”拷贝、复制函数“这一说
function fn() {}
```

## typeof和深拷贝
### 判断所有值类型

```javascript
// 判断所有值类型
typeof a; // 'undefined'
typeof "100"; // 'string'
typeof 100; // 'number'
typeof true; // 'boolean'
typeof Symbol("s"); // 'symbol'

// 判断函数
typeof console.log(); // 'function'
typeof function fn() {}; // 'function'

// 识别引用类型（不能深入识别）
typeof null; // 'object'
typeof ["a", "b"]; // 'object'
typeof { x: 100 }; // 'object'
```
### 深拷贝

> 深拷贝步骤：
> 1. 先判断值类型是否为数组或对象，如果不是则直接返回被拷贝的值。
> 2. 初始化返回结果，如果引用类型是数组则此次拷贝返回一个数组，如果是对象则返回一个对象。
> 3. 遍历引用类型的key，保证key不是原型上的属性.
> 4. 对遍历的值进行递归拷贝。


```javascript
function deepClone(obj = {}) {
  // 如果obj不是一个对象和数组，或者obj是null，则直接返回obj
  if (typeof obj !== "object" || obj == null) {
    return obj;
  }

  // 初始化返回结果
  let result;
  if (obj instanceof Array) {
    // 如果是数组类型则返回数组
    result = [];
  } else {
    // 如果是对象类型则返回对象
    result = {};
  }

  for (let key in obj) {
    // 保证key不是原型的属性
    if (obj.hasOwnProperty(key)) {
      // 递归
      result[key] = deepClone(obj[key])
    }
  }

  // 返回结果
  return result;
}}
```
## 变量计算
### 字符串拼接

```javascript
const a = 100 + 10 // 110
const b = 100 + '10' // '10010'
const c = true + '10' // 'true10'
```

### ==运算符
> == 会尝试将两边的值进行类型转换后使他们尽量相等  

```javascript
100 == '100' // true
0 == '' // true
0 == false // true
false == '' // true
null == undefined // true
// 除了 == null之外，其他一律使用 === ，例如：
const obj = { x: 100 }
if (obj.a == null ) {}
// 相当于
if (obj.a === null || obj.a === undefined) {}
```

### if语句和逻辑运算
#### if语句

> if语句判断的其实是truly变量和falsely变量

- **truly变量：!!a === true的变量**
- **falsely变量：!!a === false的变量**

```javascript
// truly变量
const a = true
if (a) {
    // ...
}
const b = 100
if (b) {
    // ...
}
```
```javascript
// falsely变量
const c = ''
if (c) {
    // ...
}
const d = null
if (d) {
    // ...
}
let e
if (e) {
    / ...
}
```
#### 逻辑运算

```javascript
10 && 0 // 0
'' || 'abc' // 'abc'
!window.abc // true
```
