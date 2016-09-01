# Vue/core/instance/state.js

## ☆ [fn] initState

在 Vue.prototype.\_init 被调用，初始化组件的选项

``` javascript
function initState (vm: Component) {
  vm._watchers = []
  initProps(vm)
  initData(vm)     
  initComputed(vm)
  initMethods(vm)
  initWatch(vm)
}
```

## ☆ [fn] stateMixin

``` javascript
function stateMixin (Vue: Class<Component>) {
  // flow 会在使用 Object.defineProperty 直接声明定义对象是发生一些问题。
  // 所以我们在这逐步建立对象 ？
  const dataDef = {}
  dataDef.get = function () {
    return this._data
  }
  if (process.env.NODE_ENV !== 'production') {
    dataDef.set = function (newData: Object) {
      warn(
        'Avoid replacing instance root $data. ' +
        'Use nested data properties instead.',
        this
      )
    }
  }
  Object.defineProperty(Vue.prototype, '$data', dataDef)

  // observer/index
  Vue.prototype.$set = set
  Vue.prototype.$delete = del

  // 返回一个可以结束当前 watch 的函数
  Vue.prototype.$watch = function (
    expOrFn: string | Function,
    cb: Function,
    options?: Object
  ): Function {
    const vm: Component = this
    options = options || {}
    // ? 哪里设置
    options.user = true

    // observer/watcher
    const watcher = new Watcher(vm, expOrFn, cb, options)
    if (options.immediate) {
      // 立即执行一次
      cb.call(vm, watcher.value)
    }

    return function unwatchFn () {
      watcher.teardown()
    }
  }
}
```
