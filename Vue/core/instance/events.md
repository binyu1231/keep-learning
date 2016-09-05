# Vue/core/instance/events.js

## [fn] initEvents

为组件添加事件功能。

``` javascript
function initEvents (vm: Component) {
  vm._events = Object.create(null)
  // 初始化父级的附加事件d
  const listeners = vm.$options._parentListeners
  // 定义组件的绑定与解绑的函数
  const on = bind(vm.$on, vm)
  const off = bind(vm.$off, vm)

  // 更新监听函数，具体的添加与删除事件的机制参考上方链接
  vm._updateListeners = (listeners, oldListeners) => {
    updateListeners(listeners, oldListeners || {}, on, off)
  }
  // 如果存在附加事件则在初始化时进行绑定
  if (listeners) {
    vm._updateListeners(listeners)
  }
}
```

- [bind](../../shared/util.md#fn-bind)
- [toArray](../../shared/util.md#fn-toarray)
- [updateListeners](../vdom/helpers.md#fn-updateListeners)

## [fn] eventsMixin

这个函数的作用是在 Vue 的原型链上定义了若干事件的方法。使得这些方法可以被实例继承。这些方法均返回组件的实例

### Vue.prototype.$on

将同名事件添加到对应的数组中

``` javascript
Vue.prototype.$on = function (event: string, fn: Function): Component {
  const vm: Component = this
  ;(vm._events[event] || (vm._events[event] = [])).push(fn)
  return vm
}
```

### Vue.prototype.$once

绑定的事件只执行一次，

``` javascript
Vue.prototype.$once = function (event: string, fn: Function): Component {
  const vm: Component = this
  function on () {
    // 解绑，
    // ？先解绑
    vm.$off(event, on)
    // 执行
    fn.apply(vm, arguments)
  }
  // 存 fn，是为了在从多个函数中解绑一个函数时作判断用
  on.fn = fn
  vm.$on(event, on)
  return vm
}
```

### Vue.prototype.$off

解绑事件。

``` javascript
Vue.prototype.$off = function (event?: string, fn?: Function): Component {
  const vm: Component = this
  // 不传参数则解绑全部事件
  if (!arguments.length) {
    vm._events = Object.create(null)
    return vm
  }
  // 指定了事件（event）参数
  const cbs = vm._events[event]
  // 没有需要解绑的事件
  if (!cbs) {
    return vm
  }
  // 事件数组中只有一个函数
  if (arguments.length === 1) {
    vm._events[event] = null
    return vm
  }
  // 指定了对应函数（fn）参数
  // 则查找对应函数并删除
  let cb
  let i = cbs.length
  while (i--) {
    cb = cbs[i]
    if (cb === fn || cb.fn === fn) {
      cbs.splice(i, 1)
      break
    }
  }
  return vm
}
```

### Vue.prototype.$emit

触发当前组件实例上的事件。

``` javascript
Vue.prototype.$emit = function (event: string): Component {
  const vm: Component = this
  let cbs = vm._events[event]
  // 如果有同名事件的存放的数组则执行
  if (cbs) {
    cbs = cbs.length > 1 ? toArray(cbs) : cbs
    const args = toArray(arguments, 1)
    for (let i = 0, l = cbs.length; i < l; i++) {
      cbs[i].apply(vm, args)
    }
  }
  return vm
}
```
