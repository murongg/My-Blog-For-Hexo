---
title: Vue2.x源码分析（二）：Vue实例挂载的实现
date: 2020-04-07 20:30:23
thumbnail: https://picsum.photos/id/28/565/242
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

在`Vue`实例挂载阶段，这里只分析`web`平台的实现，`web`平台的入口文件是`src/platforms/web/entry-runtime-with-compiler.js`，在此文件中`Vue`通过`$mount`方法进行实例挂载。

## `Vue`挂载阶段都做了什么？

在`src/platforms/web/entry-runtime-with-compiler.js`文件`$mount`方法定义之前，`$mount`已经在`runtime`\(`src/platforms/web/runtime/index.js文件`)里定义了一遍，在该文件里刚开始就对·`$mount`做一个缓存，缓存为`mount`变量，方便之后使用。
<!-- more -->
### `entry-runtime-with-compiler.js`文件做了什么？

`entry-runtime-with-compiler.js`文件中的`$mount`方法在`Vue` `init`时被调用，此方法接收两个参数，第一个参数`el`是`DOM`元素，可以是`string`或`Element`，第二个参数`hydrating`是关于`Vue`服务端渲染的，可以忽略。

首先，将传入的`el`参数通过`query`方法转换`DOM`对象，接着判断`el`是否为`body`或者`documentElement`，如果是则报出错误（不允许为`<html>` 或者` <body>`） 并直接`return`当前实例，不再往下执行。

接下来，对实例上的`$options`进行一个缓存，缓存变量为`options`，因为`Vue`支持直接传入`render`函数，所以会进行判断如果不存在`render`函数，则判断是否有`template`，再把`template`转换成一个`render`函数，最终目的是调用缓存的`mount`方法进行DOM挂载。

```javascript
// $mount已经在runtime里定义了一遍，在这里对$mount做一个缓存
const mount = Vue.prototype.$mount
// 重新定义$mount
// 此$mount在init时被调用
// 最终目的是调用mount方法进行DOM挂载
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  // 通过query方法转换DOM对象
  el = el && query(el)

  /* istanbul ignore if */
  // 判断是否为body或者documentElement，如果是则报出错误（不允许为<html> 或者 <body>）  
  // document.body：https://developer.mozilla.org/zh-CN/docs/Web/API/Document/body
  // document.documentElement：https://developer.mozilla.org/zh-CN/docs/Web/API/Document/documentElement
  if (el === document.body || el === document.documentElement) {
    process.env.NODE_ENV !== 'production' && warn(
      `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
    )
    return this
  }
  // options缓存
  const options = this.$options
  // resolve template/el and convert to render function
  // Vue支持传入render函数
  // 如果不存在render函数，则判断是否有template，再把template转换成一个render函数
  // 所有template最终都会被转换成render
  if (!options.render) {
    // 缓存template
    let template = options.template
    // 如果存在template
    if (template) {
      if (typeof template === 'string') {
        if (template.charAt(0) === '#') {
          // 如果template是一个字符串,并且第一个字符为'#'
          // 则此时的template是一个DOM的ID名
          // 通过idToTemplate方法转换成真实DOM
          template = idToTemplate(template)
          /* istanbul ignore if */
          if (process.env.NODE_ENV !== 'production' && !template) {
            warn(
              `Template element not found or is empty: ${options.template}`,
              this
            )
          }r
        }
      } else if (template.nodeType) {
        // 如果template是一个Node节点
        // 则将template赋值为template.innerHTML(DOM字符串)
        // nodeType：https://developer.mozilla.org/zh-CN/docs/Web/API/Node/nodeType
        // innerHTML：https://developer.mozilla.org/zh-CN/docs/Web/API/Element/innerHTML
        template = template.innerHTML
      } else {
        // 如果以上都不是,则报错
        if (process.env.NODE_ENV !== 'production') {
          warn('invalid template option:' + template, this)
        }
        return this
      }
    } else if (el) {
      // 如果el存在,则调用getOuterHTML获取outerHTML
      template = getOuterHTML(el)
    }
    if (template) {
      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile')
      }
      // 通过compileToFunctions获取到render
      const { render, staticRenderFns } = compileToFunctions(template, {
        outputSourceRange: process.env.NODE_ENV !== 'production',
        shouldDecodeNewlines,
        shouldDecodeNewlinesForHref,
        delimiters: options.delimiters,
        comments: options.comments
      }, this)
      options.render = render
      options.staticRenderFns = staticRenderFns

      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile end')
        measure(`vue ${this._name} compile`, 'compile', 'compile end')
      }
    }
  }
  return mount.call(this, el, hydrating)
}

/**
 * Get outerHTML of elements, taking care
 * of SVG elements in IE as well.
 * 获取outerHTML
 * outerHTML:https://developer.mozilla.org/zh-CN/docs/Web/API/Element/outerHTML
 * appendChild:https://developer.mozilla.org/zh-CN/docs/Web/API/Node/appendChild
 * cloneNode:https://developer.mozilla.org/zh-CN/docs/Web/API/Node/cloneNode
 * polyfill:https://developer.mozilla.org/zh-CN/docs/Glossary/Polyfill
 */
function getOuterHTML (el: Element): string {
  // 如果outerHTML存在，则直接return
  if (el.outerHTML) {
    return el.outerHTML
  } else {
    // 如果不存在，则创建一个空的div，将el深度克隆到新的div的尾部
    // 做一个outerHTML的polyfill
    const container = document.createElement('div')
    container.appendChild(el.cloneNode(true))
    // 返回当前innerHTML
    return container.innerHTML
  }
}
```



### `runtime/index.js` `$mount`文件做了什么？

`runtime`里的`$mount`同样接收两个参数，与`entry-runtime-with-compiler.js`文件中无差。

`runtime` `$mount`方法首先会将传入来的`el`参数通过`query`方法转换`DOM`对象（如果是浏览器环境），然后调用`mountComponent`方法生成虚拟DOM，并将生成的虚拟DOM`return`出去。

```javascript
// public mount method
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  // 通过query方法转换DOM对象
  el = el && inBrowser ? query(el) : undefined
  // 调用mountComponent生成虚拟DOM
  return mountComponent(this, el, hydrating)
}
```

### `mountComponent`方法做了什么？

`mountComponent`方法定义在`src/core/instance/lifecycle.js`下，此方法第一个第一个参数`vm`需传入一个`Vue`实例，第二个与第三个参数与`$mount`方法的参数意义一样。

`mountComponent`方法首先会将传入的`el`缓存挂载到`Vue`的实例上，名为`$el`，紧接着判断`render`函数是否存在，如果存在则会创建一个空的`VNode`，然后判断是否为`production`环境，因为在`production`环境中，`runtime-only`版本如果当前配置了`template`且`template`传入的不是一个`id`名称或者存在`el`属性，则报错，此时的情况只能使用`render`函数进行`DOM`挂载。

判断完是否存在`render`函数后，则会调用`beforeMount`生命周期钩子，此时`DOM`正在挂载中。下一步则创建一个`updateComponent`方法，该方法调用`render`方法生成虚拟`node`，然后实例化一个渲染`Watcher`，通过`Watcher`监听数据改变，当数据改变时，调用第二个参数触发_update进行更新DOM。

所有任务进行完成后，将当前实例重新`return`出去。

`mountComponent`方法主要作用就是渲染`DOM`，然后监听数据改变，进行`DOM`更新。

```javascript
export function mountComponent (
  vm: Component,
  el: ?Element,
  hydrating?: boolean
): Component {
  // 缓存$el
  vm.$el = el
  // 如果当前没有render函数
  if (!vm.$options.render) {
    // 创建一个空的VNode
    vm.$options.render = createEmptyVNode
    if (process.env.NODE_ENV !== 'production') {
      /* istanbul ignore if */
      // 在runtime-only版本如果当前配置了template且template传入的不是一个ID名称
      // 或者存在el属性
      // 则报错
      // 此时只能使用render函数
      if ((vm.$options.template && vm.$options.template.charAt(0) !== '#') ||
        vm.$options.el || el) {
        warn(
          'You are using the runtime-only build of Vue where the template ' +
          'compiler is not available. Either pre-compile the templates into ' +
          'render functions, or use the compiler-included build.',
          vm
        )
      } else {
        // 如果什么都没有传入,则报错
        warn(
          'Failed to mount component: template or render function not defined.',
          vm
        )
      }
    }
  }

  // 调用beforeMount生命周期钩子
  callHook(vm, 'beforeMount')

  let updateComponent
  /* istanbul ignore if 性能统计相关 */
  if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    updateComponent = () => {
      const name = vm._name
      const id = vm._uid
      const startTag = `vue-perf-start:${id}`
      const endTag = `vue-perf-end:${id}`

      mark(startTag)
      const vnode = vm._render()
      mark(endTag)
      measure(`vue ${name} render`, startTag, endTag)

      mark(startTag)
      vm._update(vnode, hydrating)
      mark(endTag)
      measure(`vue ${name} patch`, startTag, endTag)
    }
  } else {
    updateComponent = () => {
      // 调用render方法生成虚拟node
      vm._update(vm._render(), hydrating)
    }
  }

  // we set this to vm._watcher inside the watcher's constructor
  // since the watcher's initial patch may call $forceUpdate (e.g. inside child
  // component's mounted hook), which relies on vm._watcher being already defined
  // 实例化一个渲染watcher
  // 当数据改变时，调用第二个参数触发_update进行更新DOM
  new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted && !vm._isDestroyed) {
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
  hydrating = false

  // manually mounted instance, call mounted on self
  // mounted is called for render-created child components in its inserted hook
  if (vm.$vnode == null) {
    vm._isMounted = true
    callHook(vm, 'mounted')
  }
  return vm
}
```

