---
title: 使用vue3+typescript编写一个图片懒加载插件
date: 2020-10-23 08:22:15
categories:
  - 程序世界
  - JavaScript
  - Vue
tag:
  - Vue3
  - 懒加载
---

本插件主要功能借鉴于：[vue-lazyload](https://github.com/hilongjw/vue-lazyload)

github项目地址: https://github.com/murongg/vue3-lazyload

求star 与 issues

我文采不好，可能写的文章不咋样，有什么问题可以在留言区评论，我会尽力解答

本项目已经发布到npm

安装：
```sh
$ npm i vue3-lazyload
# or
$ yarn add vue3-lazyload
```

<!-- more -->

## 需求分析

- 支持自定义 loading 图片，图片加载状态时使用此图片

- 支持自定义 error 图片，图片加载失败后使用此图片

- 支持 lifecycle hooks，类似于 vue 的生命周期，并同时在 `img` 标签绑定 `lazy` 属性，类似于

  ```html
  <img src="..." lazy="loading">
  <img src="..." lazy="loaded">
  <img src="..." lazy="error">
  ```

  并支持：

  ```css
    img[lazy=loading] {
      /*your style here*/
    }
    img[lazy=error] {
      /*your style here*/
    }
    img[lazy=loaded] {
      /*your style here*/
    }
  ```

- 支持使用 `v-lazy` 自定义指令，指定可传入 string/object ，当为 string 时，默认为需要加载的 url，当为 object 时，可传入
  -  `src:` 当前需要加载的图片 url
  -  `loading:` 加载状态时所用到的图片
  -  `error:` 加载失败时所用到的图片
  -   `lifecycle:` 本次 lazy 的生命周期，替换掉全局生命周期
- 



### 目录结构

```
- src
---- index.ts 入口文件，主要用来注册插件
---- lazy.ts  懒加载主要功能
---- types.ts 类型文件，包括 interface/type/enum 等等
---- util.ts  共享工具文件
```

## 编写懒加载类

懒加载主要通过 [`IntersectionObserver`](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserver)对象实现，可能有些浏览器不支持，暂未做兼容。

### 确定注册插件时传入的参数

`types.ts`：

```typescript
export interface LazyOptions {
  error?: string; // 加载失败时的图片
  loading?: string; // 加载中的图片
  observerOptions?: IntersectionObserverInit; // IntersectionObserver 对象传入的第二个参数
  log?: boolean; // 是否需要打印日志
  lifecycle?: Lifecycle; // 生命周期 hooks
}

export interface ValueFormatterObject {
  src: string,
  error?: string,
  loading?: string,
  lifecycle?: Lifecycle;
}

export enum LifecycleEnum {
  LOADING = 'loading',
  LOADED = 'loaded',
  ERROR = 'error'
}

export type Lifecycle = {
  [x in LifecycleEnum]?: () => void;
};
```

### 确定类的框架

**vue3** 的 [Custom Directives](https://v3.vuejs.org/guide/custom-directive.html#intro)，支持以下 **Hook Functions**：`beforeMount` 、`mounted`、`beforeUpdate`、`updated`、`beforeUnmount`、`unmounted`，具体释义可以去 vue3 文档查看，目前仅需要用到`mounted`、`updated`、`unmounted`，这三个 Hook。

`Lazy` 类基础框架代码，`lazy.ts`：

```typescript
export default class Lazy {
  public options: LazyOptions = {
    loading: DEFAULT_LOADING,
    error: DEFAULT_ERROR,
    observerOptions: DEFAULT_OBSERVER_OPTIONS,
    log: true,
    lifecycle: {}
  };
  constructor(options?: LazyOptions) {
    this.config(options)      
  }
  
  /**
   * merge config
   * assgin 方法在 util.ts 文件内，此文章不在赘述此方法代码，可在后文 github 仓库内查看此代码
   * 此方法主要功能是合并两个对象
   *
   * @param {*} [options={}]
   * @memberof Lazy
   */
  public config(options = {}): void {
    assign(this.options, options)
  }
  
	public mount(el: HTMLElement, binding: DirectiveBinding<string | ValueFormatterObject>): void {} // 对应 directive mount hook
  public update() {} // 对应 directive update hook
  public unmount() {} // 对应 directive unmount hook
}
```

### 编写懒加载功能

```typescript
  /**
   * mount
   *
   * @param {HTMLElement} el
   * @param {DirectiveBinding<string>} binding
   * @memberof Lazy
   */
  public mount(el: HTMLElement, binding: DirectiveBinding<string | ValueFormatterObject>): void {
    this._image = el
    const { src, loading, error, lifecycle } = this._valueFormatter(binding.value)
    this._lifecycle(LifecycleEnum.LOADING, lifecycle)
    this._image.setAttribute('src', loading || DEFAULT_LOADING)
    if (!hasIntersectionObserver) {
      this.loadImages(el, src, error, lifecycle)
      this._log(() => {
        throw new Error('Not support IntersectionObserver!')
      })
    }
    this._initIntersectionObserver(el, src, error, lifecycle)
  }
  
  /**
   * force loading
   *
   * @param {HTMLElement} el
   * @param {string} src
   * @memberof Lazy
   */
  public loadImages(el: HTMLElement, src: string, error?: string, lifecycle?: Lifecycle): void {
    this._setImageSrc(el, src, error, lifecycle)
  }

  /**
   * set img tag src
   *
   * @private
   * @param {HTMLElement} el
   * @param {string} src
   * @memberof Lazy
   */
  private _setImageSrc(el: HTMLElement, src: string, error?: string, lifecycle?: Lifecycle): void {        
    const srcset = el.getAttribute('srcset')
    if ('img' === el.tagName.toLowerCase()) {
      if (src) el.setAttribute('src', src)
      if (srcset) el.setAttribute('srcset', srcset)
      this._listenImageStatus(el as HTMLImageElement, () => {
        this._log(() => {
          console.log('Image loaded successfully!')
        })
        this._lifecycle(LifecycleEnum.LOADED, lifecycle)
      }, () => {
        // Fix onload trigger twice, clear onload event
        // Reload on update
        el.onload = null
        this._lifecycle(LifecycleEnum.ERROR, lifecycle)
        this._observer.disconnect()
        if (error) el.setAttribute('src', error)
        this._log(() => { throw new Error('Image failed to load!') })
      })
    } else {
      el.style.backgroundImage = 'url(\'' + src + '\')'
    }
  }

  /**
   * init IntersectionObserver
   *
   * @private
   * @param {HTMLElement} el
   * @param {string} src
   * @memberof Lazy
   */
  private _initIntersectionObserver(el: HTMLElement, src: string, error?: string, lifecycle?: Lifecycle): void {    
    const observerOptions = this.options.observerOptions
    this._observer = new IntersectionObserver((entries) => {      
      Array.prototype.forEach.call(entries, (entry) => {
        if (entry.isIntersecting) {
          this._observer.unobserve(entry.target)
          this._setImageSrc(el, src, error, lifecycle)
        }
      })
    }, observerOptions)
    this._observer.observe(this._image)
  }

  /**
   * only listen to image status
   *
   * @private
   * @param {string} src
   * @param {(string | null)} cors
   * @param {() => void} success
   * @param {() => void} error
   * @memberof Lazy
   */
  private _listenImageStatus(image: HTMLImageElement, success: ((this: GlobalEventHandlers, ev: Event) => any) | null, error: OnErrorEventHandler) {
    image.onload = success 
    image.onerror = error
  }

  /**
   * to do it differently for object and string
   *
   * @public
   * @param {(ValueFormatterObject | string)} value
   * @returns {*}
   * @memberof Lazy
   */
  public _valueFormatter(value: ValueFormatterObject | string): ValueFormatterObject {
    let src = value as string
    let loading = this.options.loading
    let error = this.options.error
    let lifecycle = this.options.lifecycle
    if (isObject(value)) {
      src = (value as ValueFormatterObject).src
      loading = (value as ValueFormatterObject).loading || this.options.loading
      error = (value as ValueFormatterObject).error || this.options.error
      lifecycle = ((value as ValueFormatterObject).lifecycle || this.options.lifecycle)
    }
    return {
      src,
      loading,
      error,
      lifecycle
    }
  }

  /**
   * log
   *
   * @param {() => void} callback
   * @memberof Lazy
   */
  public _log(callback: () => void): void {
    if (!this.options.log) {
      callback()
    }
  }

  /**
   * lifecycle easy
   *
   * @private
   * @param {LifecycleEnum} life
   * @param {Lifecycle} [lifecycle]
   * @memberof Lazy
   */
  private _lifecycle(life: LifecycleEnum, lifecycle?: Lifecycle): void {            
    switch (life) {
    case LifecycleEnum.LOADING:
      this._image.setAttribute('lazy', LifecycleEnum.LOADING)
      if (lifecycle?.loading) {
        lifecycle.loading()
      }
      break
    case LifecycleEnum.LOADED:
      this._image.setAttribute('lazy', LifecycleEnum.LOADED)
      if (lifecycle?.loaded) {
        lifecycle.loaded()
      }
      break
    case LifecycleEnum.ERROR:
      this._image.setAttribute('lazy', LifecycleEnum.ERROR)      
      if (lifecycle?.error) {
        lifecycle.error()
      }
      break
    default:
      break
    }
  }
```

### 编写 update hook

```typescript
  /**
   * update
   *
   * @param {HTMLElement} el
   * @memberof Lazy
   */
  public update(el: HTMLElement, binding: DirectiveBinding<string | ValueFormatterObject>): void {    
    this._observer.unobserve(el)
    const { src, error, lifecycle } = this._valueFormatter(binding.value)
    this._initIntersectionObserver(el, src, error, lifecycle)
  }
```

### 编写 unmount hook

```typescript
  /**
   * unmount
   *
   * @param {HTMLElement} el
   * @memberof Lazy
   */
  public unmount(el: HTMLElement): void {
    this._observer.unobserve(el)
  }
```

## 在 index.ts 编写注册插件需要用到的 install 方法

```typescript
import Lazy from './lazy'
import { App } from 'vue'
import { LazyOptions } from './types'

export default {
  /**
   * install plugin
   *
   * @param {App} Vue
   * @param {LazyOptions} options
   */
  install (Vue: App, options: LazyOptions): void {
    const lazy = new Lazy(options)

    Vue.config.globalProperties.$Lazyload = lazy
    // 留着备用，为了兼容$Lazyload
    // 选项api，可以通过this.$Lazyload获取到Lazy类的实例，组合api我还不知道怎么获取
    // 所以通过 provide 来实现此需求
    // 使用方式 const useLazylaod = inject('Lazyload')
    Vue.provide('Lazyload', lazy)
    Vue.directive('lazy', {
      mounted: lazy.mount.bind(lazy),
      updated: lazy.update.bind(lazy),
      unmounted: lazy.unmount.bind(lazy)
    })
  }
}

```

## 使用插件

Main.js:

```js
import { createApp } from 'vue'
import App from './App.vue'
import VueLazyLoad from '../src/index'

const app = createApp(App)
app.use(VueLazyLoad, {
  log: true,
  lifecycle: {
    loading: () => {
      console.log('loading')
    },
    error: () => {
      console.log('error')
    },
    loaded: () => {
      console.log('loaded')
    }
  }
})
app.mount('#app')

```

App.vue:

```vue
<template>
  <div class="margin" />
  <img v-lazy="'/example/assets/logo.png'" alt="Vue logo" width="100">
  <img v-lazy="{src: errorlazy.src, lifecycle: errorlazy.lifecycle}" alt="Vue logo" class="image" width="100"> 
  <button @click="change">
    change
  </button>
</template>

<script>
import { reactive } from 'vue'
export default {
  name: 'App',
  setup() {
    const errorlazy = reactive({
      src: '/example/assets/log1o.png',
      lifecycle: {
        loading: () => {
          console.log('image loading')
        },
        error: () => {
          console.log('image error')
        },
        loaded: () => {
          console.log('image loaded')
        }
      }
    })
    const change = () => {
      errorlazy.src = 'http://t8.baidu.com/it/u=3571592872,3353494284&fm=79&app=86&size=h300&n=0&g=4n&f=jpeg?sec=1603764281&t=bedd2d52d62e141cbb08c462183601c7'
    }
    return {
      errorlazy,
      change
    }
  }
}
</script>

<style>
.margin {
  margin-top: 1000px;
}
.image[lazy=loading] {
  background: goldenrod;
}
.image[lazy=error] {
  background: red;
}
.image[lazy=loaded] {
  background: green;
}
</style>

```


