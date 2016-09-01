# Vue/core/instance/init.js

## [fn] initMixin

为 Vue 的实例添加 \_init 函数，这个函数接收一个 options[object] 作为参数，将其合并到原始的 options 上后进行初始化操作。

``` javascript
function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    const vm: Component = this

    // 唯一的 uid
    vm._uid = uid++

    // 防止被观察（observed）的标记
    vm._isVue = true

    // 合并配置项
    if (options && options._isComponent) {

      // 优化内部组件实例，因为动态的配置项合并相当慢
      // 内部的组件配置项并不需要特殊处理 🔽🔽🔽 函数实现在下面
      initInternalComponent(vm, options)

    } else {
      // 没有传入配置项 || 传入的配置项不是组件
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm), // 解析构造函数配置项 🔽🔽🔽 函数实现在下面
        options || {},
        vm
      )
    }

    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
    // expose real self
    vm._self = vm
    initLifecycle(vm)
    initEvents(vm)
    callHook(vm, 'beforeCreate')
    initState(vm)
    callHook(vm, 'created')
    initRender(vm)
  }

  function initInternalComponent (vm: Component, options: InternalComponentOptions) {
    const opts = vm.$options = Object.create(resolveConstructorOptions(vm))
    // 这样做比动态枚举更快
    opts.parent = options.parent
    opts.propsData = options.propsData
    opts._parentVnode = options._parentVnode
    opts._parentListeners = options._parentListeners
    opts._renderChildren = options._renderChildren
    opts._componentTag = options._componentTag
    if (options.render) {
      opts.render = options.render
      opts.staticRenderFns = options.staticRenderFns
    }
  }

  function resolveConstructorOptions (vm: Component) {
    const Ctor = vm.constructor
    let options = Ctor.options
    if (Ctor.super) {
      const superOptions = Ctor.super.options
      const cachedSuperOptions = Ctor.superOptions
      if (superOptions !== cachedSuperOptions) {
        // 根组件配置项改变
        Ctor.superOptions = superOptions
        // /core/util/[fn]merageOptions
        options = Ctor.options = mergeOptions(superOptions, Ctor.extendOptions)
        if (options.name) {
          // 如果有组件名
          options.components[options.name] = Ctor
        }
      }
    }
    return options
  }
}

```
