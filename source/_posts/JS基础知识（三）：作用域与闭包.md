---
title: JS基础知识（三）：作用域与闭包
date: 2020-02-28 19:04:30
thumbnail: https://i.picsum.photos/id/174/565/242.jpg
categories:
  - 程序世界
  - JavaScript
tag:
  - JavaScript
  - JavaScript基础
  - 闭包
  - 作用域
  - this
---

## 作用域和自由变量

### 作用域
- 全局作用域
- 函数作用域
- 块级作用域（ES6新增）

### 自由变量
- 一个变量在当前作用域没有定义，但被使用了
- 向上级作用域，一层一层寻找，直到找到为止
- 如果到全局作用域都没有找到，则报错xx is not defined
<!-- more -->
## 闭包
 
作用域应用的特殊情况，由两种表现：

- 函数作为参数被传递
- 函数作为返回值被返回

例：
```javascript
// 函数作为返回值
function create() {
    let a = 100
    return function () {
        console.log(a)
    }
}

let fn = create()
let a = 200
fn() // 100
```

```javascript
function print(fn) {
    let a = 200
    fn()
}

let a = 100
function fn () {
    console.log(a)
}
print(fn) // 100
```

> 由上两个例子可以得出：  
**闭包：自由变量的查找，是在函数定义的地方，向上级作用域查找，不是在执行的地方！！！**

## this

`this`调用的几种情况：
- 作为普通函数调用
- 使用`call`、`apply`、`bind`
- 作为对象方法被调用
- 在`class`方法中被调用
- 箭头函数中调用

> **`this`取什么值在函数执行时确定，不是函数定义时确定**

例一：
```javascript
function fn1() {
  console.log(this);
}
fn(); // window

fn1.call({ x: 100 }); // this指向{x:100}

const fn2 = fn1.bind({ x: 200 }); // bind会返回一个新的函数
fn2(); // {x:200}
```

例二：
```javascript
const zhangsan = {
  name: "张三",
  sayHi() {
    // this指向当前对象
    console.log(this);
  },
  wait() {
    setTimeout(function() {
      // this指向window
      // 因为此时的setTimeout这个方法作为普通函数进行执行
      // 并不是作为zhangsan这个对象的方法进行执行
      console.log(this);
    });
  }
};
```

例三：

```javascript
const zhangsan = {
  name: "张三",
  sayHi() {
    // this指向当前对象
    console.log(this);
  },
  waitAgain() {
    setTimeout(() => {
      // this指向当前对象
      // 箭头函数的this永远取它上级作用域的this
      console.log(this);
    });
  }
};
```

例四：
```javascript
class People {
  constructor(name) {
    this.name = name;
    this.age = 20;
  }
  sayHi() {
    console.log(this);
  }
}
const zhangsan = new People('张三')
zhangsan.sayHi() // zhangsan这个对象
```

## 手写call、apply和bind

### call

手写`call`之前，需要分析一下需要几个步骤：  
我们将调用方法的对象叫做A，将需要改变this的对象叫做B
1. 判断是否传入参数，如果没有传入，那么默认为 window
2. 给 context 添加一个临时属性，属性指向A
3. 将 context 后面的参数取出来
4. 将参数传入A
5. 删除context临时添加的属性


```javascript
Function.prototype.myCall = function(context) {
  // 判断是否传入参数，如果没有传入，那么默认为 window
  context = context || window;
  // 给 context 添加一个属性，属性指向当前对象
  context.fn = this;
  // 将 context 后面的参数取出来
  const args = [...arguments].slice(1);
  // 将参数传入当前对象
  const result = context.fn(args);
  delete context.fn;
  return result;
};
```

### apply

`apply`与`call`差不多，只不过`apply`只需要传入两个参数，`call`可以传入无数个参数

```javascript
Function.prototype.myApply = function(context) {
  context = context || window;
  context.fn = this;
  // 需要判断是否存储第二个参数
  // 如果存在，就将第二个参数展开
  const result = arguments[1]
    ? context.fn(...arguments[1])
    : context.fn();
  delete context.fn;
  return result;
};
```

### bind

`bind`与上方两个的区别是`bind`会返回一个函数

```javascript
Function.prototype.myBind = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  var _this = this
  var args = [...arguments].slice(1)
  // 返回一个函数
  return function F() {
    // 因为返回了一个函数，我们可以 new F()，所以需要判断
    if (this instanceof F) {
      return new _this(...args, ...arguments)
    }
    return _this.apply(context, args.concat(...arguments))
  }
}
```