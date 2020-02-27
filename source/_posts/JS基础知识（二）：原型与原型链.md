---
title: JS基础知识（二）：原型与原型链
date: 2020-02-27 19:56:48
thumbnail: https://i.picsum.photos/id/894/565/242.jpg
categories:
  - 程序世界
  - JavaScript
tag:
  - JavaScript
  - JavaScript基础
  - 原型
  - 原型链
---

## class和继承
### 类的基本使用

```javascript
// 定义一个类
class Student {
    constructor(name, number) {
        this.name = name
        this.numer = number
    }
    sayHi() {
        console.log(`姓名 ${this.name} , 学号 ${this.number}`)
    }
}
// 通过类 new 对象/实例
const xiaoming = new Student('小明', 50)
console.log(xiaoming.name) // 小明
console.log(xiaoming.number) // 50
xiaoming.sayHi() // 姓名 小明 , 学号 50
```
<!--more-->
### 继承
1. 子类通过`extends`继承父类
2. 子类必须在构造函数里使用`super()`才可以使用父类的类方法或类属性
3. `super()`方法里传入的参数会传进父类的构造函数里


```javascript
// 定义一个父类 
class People {
    constructor(name) {
        this.name = name
    }
    eat() {
        console.log(`${this.name}吃一下`)
    }
}

// 定义子类
class Student extends People {
    constructor(name, number) {
        super(name)
        this.numer = number
    }
    sayHi() {
        console.log(`姓名 ${this.name} 学号 ${this.numer}`)
    }
}

class Teacher extends People {
    constructor(name, major) {
        super(name)
        this.major = major
    }
    teach() {
        console.log(`${this.name} 教授 ${this.major}`)
    }
}

const student = new Student('小明', 200)
student.eat() // 小明吃一下
student.sayHi() // 姓名 小明 学号 200

const teacher = new Teacher('A老师', '数学')
teacher.eat() // A老师吃一下
teacher.teach() // A老师 教授 数学
```

## 原型

```javascript
// class实际上是函数，是ES6为我们提供的语法糖
typeof People // 'function'
typeof Student // 'function'

// 隐式原型和显式原型
console.log(student.__proto__) // 隐式原型
console.log(Student.prototype) // 显式原型

// 实例的隐式原型等于对应类的显式原型
console.log(student.__proto__ === Student.prototype)
```
### 原型关系
- **每个class都有显式原型prototype**
- **每个实例都有隐式原型__proto__**
- **实例的__proto__指向对应class的prototype**

### 执行规则
- **获取属性student.name或执行方法student.sayHi()时**
- **先寻找自身属性和方法**
- **如果找不到，则自动去__proto__中查找**

## 原型链

> 每个对象都可以有一个原型__proto__，这个原型还可以有它自己的原型，以此类推，形成一个原型链。查找特定属性的时候，我们先去这个对象里去找，如果没有的话就去它的原型对象里面去，如果还是没有的话再去向原型对象的原型对象里去寻找...... 这个操作被委托在整个原型链上，这个就是我们说的原型链了。
原文链接：https://www.jianshu.com/p/08c07a953fa0

### 流程图
> Object是所有class的父类

![原型链流程图](https://s2.ax1x.com/2020/02/27/3w8h59.png)


## instanceof类型判断
```javascript
student instanceof Student // true
student instanceof People // true
student instanceof Object // true

[] instanceof Array // true
[] instanceof Object // true

{} instanceof Object // true
```
**手写instanceof，步骤：**
1. 获得类型的原型
2. 获得对象的原型
3. 循环判断对象的类型是否等于类型的原型
4. 如果对象的类型等于类型的原型，则返回true
5. 如果等于null，则代表已经遍历到Object，返回false
6. 如果以上条件都不成立，递归，顺着原型链继续判断

**代码实现：**
```javascript
function instanceof(left, right) {
    // 获得类型的原型
    let prototype = right.prototype
    // 获得对象的原型
    left = left.__proto__
    // 判断对象的类型是否等于类型的原型
    while (true) {
    	if (left === null)
    		return false
    	if (prototype === left)
    		return true
    	left = left.__proto__
    }
}
```

## 手写实现简易jQuery考虑插件和扩展性

```javascript
class jQuery {
  constructor(selector) {
    const dom = document.querySelectorAll(selector);
    this.length = dom.length;
    this.selector = selector;
    // 将所有dom挂载到原型上
    for (let i = 0; i < this.length; i++) {
      this[i] = dom[i]
    }
  }
  // 获取某个dom
  get(index) {
    return this[index]
  }
  // jquery each方法
  each(fn) {
    for (let i = 0; i < this.length; i++) {
        fn(this[i])
    }
  }
  // 监听事件
  on(type, fn) {
    return this.each(elem => {
      elem.addEventListener(type, fn, false)
    })
  }
}

// 插件
jQuery.prototype.dialog = function (info) {
  console.log(info)
}

// 基于jQuery二次封装
class myJQuery extends jQuery {
  constructor(selector) {
    super(selector)
  }

  addClass() {
    console.log('addClass')
  }

  add() {
    console.log('add')
  }
}
```