# Vue/core/util/env.js

检测环境，做兼容

## [boolean] hasProto/inBrowser/devtools

``` javascript
// 是否可以使用 __proto__
const hasProto = '__proto__' in {}

// 是否运行在浏览器中
const inBrowser =
  typeof window !== 'undefined' &&
  Object.prototype.toString.call(window) !== '[object Object]'

// 检查 devtools
const devtools = inBrowser && window.__VUE_DEVTOOLS_GLOBAL_HOOK__

```

## [string] UA
``` javascript
const UA = inBrowser && window.navigator.userAgent.toLowerCase()
```

## [fn] nextTick

nextTick 这是一个立即执行的匿名函数 `(function)()` 返回的函数，所以函数的本体是最后返回的函数。作用是**异步推迟**执行一个任务，让细小的任务在一个统一的 tick 中执行。在 [MutationObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/MutationObserver#MutationObserverInit) 可用的情况下使用 MutationObserver，否则使用定时器来进行异步调用。

``` javascript
// 变动监听在 iOS 9.3 以上并且，没有 IndexedDB 时才可用
const hasMutationObserverBug =
  iosVersion &&
  Number(iosVersion[0]) >= 9 &&
  Number(iosVersion[1]) >= 3 &&
  !window.indexedDB

const nextTick = (function () {
  let callbacks = []  // 所有需要在下一次 tick 中执行的任务都会被放到这个数组中
  let pending = false // 是否推迟执行
  let timerFunc       // 定时器函数

  function nextTickHandler () {
    pending = false
    const copies = callbacks.slice(0)
    // 清空引用，注意这里将任务中的引用都清空了，所以需要才需要传入 ctx 进行绑定
    callbacks = []
    // 在当前 tick 执行所有积攒的任务
    for (let i = 0; i < copies.length; i++) {
      copies[i]()
    }
  }

  /* istanbul ignore else */
  /* 上面一行注释，让之后的 else 不进行测试覆盖率检查 npm: istanbul */

  if (typeof MutationObserver !== 'undefined' && !hasMutationObserverBug) {
    var counter = 1
    var observer = new MutationObserver(nextTickHandler)
    // 文本节点
    var textNode = document.createTextNode(String(counter))
    observer.observe(textNode, {
      characterData: true // 观测目标节点的文本节点 textNode
    })
    timerFunc = function () {
      // 改变 textNode 内部的文本节点，继而触发 observer
      counter = (counter + 1) % 2
      textNode.data = String(counter)
    }
  } else {
    // 原注释：webpack 尝试为立即定时器 setImmediate 注入垫片
    // 如果立即定时器全局可用，我们则需要做一些处理避免捆绑不必要的代码
    var context = inBrowser
      ? window
      : typeof global !== 'undefined' ? global : {}
    timerFunc = context.setImmediate || setTimeout
  }

  // nextTick 本体
  return function (cb: Function, ctx?: Object) {
    const func = ctx
      ? function () { cb.call(ctx) }
      : cb
    callbacks.push(func)
    if (pending) return
    pending = true
    timerFunc(nextTickHandler, 0)
  }
})()
```

## [fn] \_Set

``` javascript
let _Set
/* istanbul ignore if */
// 上面一行注释，让之后的 if 不进行测试覆盖率检查 npm: istanbul

if (typeof Set !== 'undefined' && /native code/.test(Set.toString())) {
  // 检查原生 Set 是否可用
  _Set = Set
} else {
  // 实现一个 key 只支持是原始类型的 set
  _Set = class Set {
    set: Object;
    constructor () {
      this.set = Object.create(null)
    }
    has (key: string | number) {
      return this.set[key] !== undefined
    }
    add (key: string | number) {
      this.set[key] = 1
    }
    clear () {
      this.set = Object.create(null)
    }
  }
}

```
