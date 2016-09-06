# Vue/core/instance/state.js

## [fn] initState

在 [Vue.prototype.\_init](./init.md#fn-initmixin) 中被调用，初始化组件的各个配置项。

``` javascript
function initState (vm: Component) {
  vm._watchers = []
  initProps(vm)    // props 定义在 vm 上
  initData(vm)     // data 定义到 vm._data 上，再使用代理
  initComputed(vm) // computed 定义在 vm 上
  initMethods(vm)  // methods 定义在 vm 上
  initWatch(vm)
}
```

_[fn] initProps_

初始化 props

``` javascript
function initProps (vm: Component) {
  const props = vm.$options.props
  const propsData = vm.$options.propsData
  if (props) {
    const keys = vm.$options._propKeys = Object.keys(props)
    const isRoot = !vm.$parent

    // 根实例的 props 配置项需要转化为可响应的
    observerState.shouldConvert = isRoot

    for (let i = 0; i < keys.length; i++) {
      const key = keys[i]

      // defineReactive 定义响应式属性
      // validateProp 验证 propsData 中是否有 key 对应的值，
      // 并返回一个经过校验的合理的值
      if (process.env.NODE_ENV !== 'production') {
        defineReactive(vm, key, validateProp(key, props, propsData, vm), () => {
          if (vm.$parent && !observerState.isSettingProps) {
            warn(
              `Avoid mutating a prop directly since the value will be ` +
              `overwritten whenever the parent component re-renders. ` +
              `Instead, use a data or computed property based on the prop's ` +
              `value. Prop being mutated: "${key}"`,
              vm
            )
          }
        })
      } else {
        defineReactive(vm, key, validateProp(key, props, propsData, vm))
      }
    }
    // 恢复默认设置：需要转化成可响应的属性
    observerState.shouldConvert = true
  }
}
```

- [observerState](../observer/index.md#object-observerstate)
- [defineReactive](../observer/index.md#fn-definereactive)
- [validateProp](../util/props.md#fn-validateprop)


_[fn] initData_

``` javascript
function initData (vm: Component) {
  let data = vm.$options.data
  // data 配置项的类型为函数或者对象
  data = vm._data = typeof data === 'function'
    ? data.call(vm)
    : data || {}
  if (!isPlainObject(data)) {
    data = {}
    process.env.NODE_ENV !== 'production' && warn(
      'data functions should return an object.',
      vm
    )
  }

  // 将 data 代理到实例上
  const keys = Object.keys(data)
  const props = vm.$options.props
  let i = keys.length
  while (i--) {
    if (props && hasOwn(props, keys[i])) {
      // data 不能与 props 有同名项
      process.env.NODE_ENV !== 'production' && warn(
        `The data property "${keys[i]}" is already declared as a prop. ` +
        `Use prop default value instead.`,
        vm
      )
    } else {
      // vm.aaa 相当于 vm._data.aaa
      proxy(vm, keys[i])
    }
  }
  // 为 data 添加观察者，监控变化
  observe(data)
  // 标记依赖根 $data 的个数
  // 杜绝在运行时添加可响应的属性
  data.__ob__ && data.__ob__.vmCount++
}

function proxy (vm: Component, key: string) {
  if (!isReserved(key)) {
    // 不是 _ 和 $ 开头，非内部变量
    Object.defineProperty(vm, key, {
      configurable: true,
      enumerable: true,
      get: function proxyGetter () {
        return vm._data[key]
      },
      set: function proxySetter (val) {
        vm._data[key] = val
      }
    })
  }
}
```

- [isPlainObject](../../shared/util.md#fn-isplainobject)
- [isReserved](../util/lang.md#fn-isreserved)
- [observe](../observer/index.md#fn-observe)
- [hasOwn](../../shared/util.md#fn-hasown)
- [Object.defineProperty ie9+](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)

_[fn] initComputed_

``` javascript
// computed 默认修饰符定义
const computedSharedDefinition = {
  enumerable: true,
  configurable: true,
  get: noop,
  set: noop
}

function initComputed (vm: Component) {
  const computed = vm.$options.computed
  if (computed) {
    for (const key in computed) {
      const userDef = computed[key]
      if (typeof userDef === 'function') {
        // computed: { foo: function () {} }
        computedSharedDefinition.get = makeComputedGetter(userDef, vm)
        computedSharedDefinition.set = noop
      } else {
        // computed: { foo: { get: bar1, set: bar2, cache: true } }
        computedSharedDefinition.get = userDef.get
          ? userDef.cache !== false
            ? makeComputedGetter(userDef.get, vm)
            : bind(userDef.get, vm) // 原因：computed 函数中的 this 指向 vm
          : noop
        computedSharedDefinition.set = userDef.set
          ? bind(userDef.set, vm) // 原因：computed 函数中的 this 指向 vm
          : noop
      }
      Object.defineProperty(vm, key, computedSharedDefinition)
    }
  }
}

function makeComputedGetter (getter: Function, owner: Component): Function {
  const watcher = new Watcher(owner, getter, noop, {
    lazy: true
  })
  return function computedGetter () {
    if (watcher.dirty) {
      watcher.evaluate()
    }
    if (Dep.target) {
      watcher.depend()
    }
    return watcher.value
  }
}
```

- [noop](../../shared/util.md#fn-noop)
- [bind](../../shared/util.md#fn-bind)
- [Object.defineProperty ie9+](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)
- [Watcher](../observer/watcher.md#class-watcher)
- [Dep](../observer/dep.md#class-dep)

_[fn] initMethods_

``` javascript
function initMethods (vm: Component) {
  const methods = vm.$options.methods
  if (methods) {
    for (const key in methods) {
      // bind(fn, ctx)
      // 原因：methods 里的函数不指向 vm 的 methods 对象
      // 而指向 vm
      vm[key] = bind(methods[key], vm)
    }
  }
}
```

- [bind](../../shared/util.md#fn-bind)

_[fn] initWatch_


``` javascript
function initWatch (vm: Component) {
  const watch = vm.$options.watch
  if (watch) {
    for (const key in watch) {
      const handler = watch[key]
      if (Array.isArray(handler)) {
        // watch: { foo: [ handler1, handler2 ] }
        for (let i = 0; i < handler.length; i++) {
          createWatcher(vm, key, handler[i])
        }
      } else {
        // watch: { foo: handler }
        createWatcher(vm, key, handler)
      }
    }
  }
}

function createWatcher (vm: Component, key: string, handler: any) {
  let options
  if (isPlainObject(handler)) {
    // 传的是对象
    options = handler
    handler = handler.handler
  }
  if (typeof handler === 'string') {
    // 传的是字符串,访问 methods 上的方法
    handler = vm[handler]
  }
  vm.$watch(key, handler, options)
}
```

- [watch 配置项的使用方法](http://vuejs.org.cn/api/#watch)
- [isPlainObject](../../shared/util.md#fn-isplainobject)
- $watch 实现在下面


## [fn] stateMixin

``` javascript
function stateMixin (Vue: Class<Component>) {
  // flow 会在使用 Object.defineProperty 直接声明定义对象是发生一些问题。
  const dataDef = {}
  dataDef.get = function () {
    return this._data
  }
  // 不能替换掉实例的根 $data
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

    // watch 配置项是用户行为
    options.user = true

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

- [set](../observer/index.md#fn-set)
- [del](../observer/index.md#fn-del)
- [Watcher](../observer/watcher.md#class-watcher)
