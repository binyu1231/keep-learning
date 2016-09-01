# Vue/core/instance/lifecycle.js

## [any] activeInstance

用于存储当前活跃的实例

## [fn] initLifecycle

初始化一些生命周期功能

``` javascript
function initLifecycle (vm: Component) {
  const options = vm.$options

  // 找到最近一个非抽象父
  let parent = options.parent
  if (parent && !options.abstract) {
    while (parent.$options.abstract && parent.$parent) {
      parent = parent.$parent
    }
    parent.$children.push(vm)
  }

  vm.$parent = parent
  vm.$root = parent ? parent.$root : vm

  vm.$children = []
  vm.$refs = {}

  vm._watcher = null
  vm._inactive = false
  vm._isMounted = false
  vm._isDestroyed = false
  vm._isBeingDestroyed = false
}
```

## [fn] lifecycleMixin

为 Vue 实例添加了几个方法

### Vue.prototype..\_mount

用于装载（载入）组件

``` javascript
Vue.prototype._mount = function (
  el?: Element | void,
  hydrating?: boolean
): Component {
  const vm: Component = this
  vm.$el = el
  if (!vm.$options.render) {
    vm.$options.render = emptyVNode
    if (process.env.NODE_ENV !== 'production') {
      /* istanbul ignore if */
      if (vm.$options.template) {
        warn(
          'You are using the runtime-only build of Vue where the template ' +
          'option is not available. Either pre-compile the templates into ' +
          'render functions, or use the compiler-included build.',
          vm
        )
      } else {
        // 既没有 template 也 没有 render
        warn(
          'Failed to mount component: template or render function not defined.',
          vm
        )
      }
    }
  }
  // 调用 beforeMount 生命周期函数
  callHook(vm, 'beforeMount')

  // /core/observer/Watcher[class]
  vm._watcher = new Watcher(vm, () => {

    // 变动执行函数，🔽🔽🔽 函数实现
    vm._update(vm._render(), hydrating)

  }, noop)
  hydrating = false
  // 根实例需自行调用 mounted 生命周期函数
  // 子组件在自己的钩子中调用了 mounted 函数
  if (vm.$root === vm) {
    vm._isMounted = true
    callHook(vm, 'mounted')
  }
  return vm
}
```

### ☆ Vue.prototype.\_update

用于更新组件 ？

``` javascript
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
  const vm: Component = this
  if (vm._isMounted) {
    // 如果加载完毕调用 beforeUpdate 生命周期
    callHook(vm, 'beforeUpdate')
  }
  // 存储更新前的信息
  const prevEl = vm.$el
  const prevActiveInstance = activeInstance
  activeInstance = vm
  const prevVnode = vm._vnode
  vm._vnode = vnode

  if (!prevVnode) {
    // Vue.prototype.__patch__ 在入口点已被注入
    // 基于后端如何渲染
    // ?
    vm.$el = vm.__patch__(vm.$el, vnode, hydrating)
  } else {
    vm.$el = vm.__patch__(prevVnode, vnode)
  }
  activeInstance = prevActiveInstance
  // 更新 __vue__ 接口 ？
  if (prevEl) {
    prevEl.__vue__ = null
  }
  if (vm.$el) {
    vm.$el.__vue__ = vm
  }
  // if parent is an HOC, update its $el as well ?
  if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
    vm.$parent.$el = vm.$el
  }
  // 调用钩子函数
  if (vm._isMounted) {
    callHook(vm, 'updated')
  }
}
```

### ☆ Vue.prototype.\_updateFromParent

从父节点更新 ？

``` javascript
Vue.prototype._updateFromParent = function (
  propsData: ?Object,
  listeners: ?Object,
  parentVnode: VNode,
  renderChildren: ?VNodeChildren
) {
  const vm: Component = this
  const hasChildren = !!(vm.$options._renderChildren || renderChildren)
  vm.$options._parentVnode = parentVnode
  vm.$options._renderChildren = renderChildren
  // update props
  if (propsData && vm.$options.props) {
    observerState.shouldConvert = false
    if (process.env.NODE_ENV !== 'production') {
      observerState.isSettingProps = true
    }
    const propKeys = vm.$options._propKeys || []
    for (let i = 0; i < propKeys.length; i++) {
      const key = propKeys[i]
      vm[key] = validateProp(key, vm.$options.props, propsData, vm)
    }
    observerState.shouldConvert = true
    if (process.env.NODE_ENV !== 'production') {
      observerState.isSettingProps = false
    }
  }
  // 更新事件监听器
  if (listeners) {
    const oldListeners = vm.$options._parentListeners
    vm.$options._parentListeners = listeners
    vm._updateListeners(listeners, oldListeners)
  }
  // 如果有子节点 解析 slots 并强制更新
  if (hasChildren) {
    vm.$slots = resolveSlots(renderChildren)
    vm.$forceUpdate()
  }
}
```

### Vue.prototype.$forceUpdate

强制更新

``` javascript
Vue.prototype.$forceUpdate = function () {
  const vm: Component = this
  if (vm._watcher) {
    vm._watcher.update()
  }
}
```

### Vue.prototype.$destroy

``` javascript
Vue.prototype.$destroy = function () {
  const vm: Component = this

  // 正在销毁
  if (vm._isBeingDestroyed) {
    return
  }

  // 调用 beforeDestroy 钩子函数
  callHook(vm, 'beforeDestroy')
  vm._isBeingDestroyed = true
  // 从父级中删除
  const parent = vm.$parent
  if (parent && !parent._isBeingDestroyed && !vm.$options.abstract) {
    remove(parent.$children, vm)
  }
  // 拆卸 watchers
  if (vm._watcher) {
    vm._watcher.teardown()
  }
  let i = vm._watchers.length
  while (i--) {
    vm._watchers[i].teardown()
  }
  // 从 data ob 上删除接口
  // 冻结的对象可能没有观察者
  if (vm._data.__ob__) {
    vm._data.__ob__.vmCount--
  }

  vm._isDestroyed = true
  callHook(vm, 'destroyed')
  // 撤销实例上所有的监听器
  vm.$off()
  // 删除 __vue__ 接口
  if (vm.$el) {
    vm.$el.__vue__ = null
  }
}
```

## [fn] callHook

调用一个生命周期函数。

``` javascript
function callHook (vm: Component, hook: string) {
  const handlers = vm.$options[hook]
  if (handlers) {
    for (let i = 0, j = handlers.length; i < j; i++) {
      handlers[i].call(vm)
    }
  }
  // 触发一次事件 eg: 'hook:ready'
  vm.$emit('hook:' + hook)
}
```
