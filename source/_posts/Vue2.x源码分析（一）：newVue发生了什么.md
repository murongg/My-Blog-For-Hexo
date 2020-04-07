---
title: Vue2.x源码分析（一）：new Vue发生了什么
date: 2020-03-31 18:31:45
thumbnail: https://picsum.photos/id/23/565/242
categories:
  - 程序世界
  - WEB前端
  - 源码分析
tags:
  - Vue
  - JavaScript
  - Vue源码分析
---

[Vue源码分析 + 逐行注释 Github地址](https://github.com/MuRongXiaoDouBi/vue2.x-analysis-comment)

## new Vue()做了什么？

首先，`Vue`会判断当前的`this`是否是`Vue`实例，如果是则会调用`this._init()`初始化`Vue`配置，如果不是则抛出警告

代码`src/core/instance/index.js`：

```javascript
function Vue(options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}
```
<!-- more -->

## Vue都初始化了什么？

在初始化`Vue`时，`Vue`会执行一些混入操作，将一些方法等挂载到`Vue`的原型链上，混入完成后，将`Vue`类（其实定义一个`function`则是定义一个类，只不过`Vue`没有使用`ES6`的`class`语法糖，而是用原型链的方式）导出

代码`src/core/instance/index.js`：

```javascript
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
function Vue(options) {
  ....
}
initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue
```

## this._init()从何而来

`this._init()`是由混入方法`initMixin`挂载到原型链上的一个方法。

```javascript
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    // a uid
    vm._uid = uid++

    let startTag, endTag
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      startTag = `vue-perf-start:${vm._uid}`
      endTag = `vue-perf-end:${vm._uid}`
      mark(startTag)
    }

    // a flag to avoid this being observed
    vm._isVue = true
    // merge options
    if (options && options._isComponent) {
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the
      // internal component options needs special treatment.
      initInternalComponent(vm, options)
    } else {
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )
    }
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
    // expose real self
    vm._self = vm
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')

    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      vm._name = formatComponentName(vm, false)
      mark(endTag)
      measure(`vue ${vm._name} init`, startTag, endTag)
    }

    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
  }
}
```

## 总结

`Vue`初始化主要做了合并配置、初始化生命周期、初始化事件中心、初始化渲染、初始化`data`、`props`、`computed`、`watcher`等。初始化完成后判断`$option`里有没用传入`el`（字符串），如果有，则使用原型上的`$mount`挂载DOM。