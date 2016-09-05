# Vue/core/vdom/create-component.js

## [fn] createComponent

``` javascript
const hooks = { init, prepatch, insert, destroy }
const hooksToMerge = Object.keys(hooks)

function createComponent (
  Ctor: Class<Component> | Function | Object | void,
  data?: VNodeData,
  context: Component, // 上下文
  children?: VNodeChildren,
  tag?: string
): VNode | void {
  if (!Ctor) {
    return
  }

  if (isObject(Ctor)) {
    // Ctor 是对象，以 Ctor 创建组件
    // 返回构造函数，进行下方判断
    Ctor = Vue.extend(Ctor)
  }

  if (typeof Ctor !== 'function') {
    if (process.env.NODE_ENV !== 'production') {
      warn(`Invalid Component definition: ${Ctor}`, context)
    }
    return
  }

  // 异步组件
  if (!Ctor.cid) {
    if (Ctor.resolved) {
      Ctor = Ctor.resolved
    } else {
      // 没有解析
      Ctor = resolveAsyncComponent(Ctor, () => {
        // 每次渲染过程中改变队列是可行的，
        // 因为 $forceUpdate 在异步情况下使用了 scheduler 进行缓冲
        // Vue/core/observer/scheduler.js => [fn] queueWatcher
        context.$forceUpdate()
      })
      if (!Ctor) {
        // 如果这确实是异步组件则不返回任何东西
        // 等待回调函数触发父级的更新
        return
      }
    }
  }

  data = data || {}

  // 提取 props extractProps 🔽🔽🔽
  const propsData = extractProps(data, Ctor)

  // 功能性组件 createFunctionalComponent 🔽🔽🔽
  if (Ctor.options.functional) {
    return createFunctionalComponent(Ctor, propsData, data, context, children)
  }
  // 提取监听器，
  // extract listeners, since these needs to be treated as
  // child component listeners instead of DOM listeners
  const listeners = data.on
  // replace with listeners with .native modifier
  data.on = data.nativeOn

  if (Ctor.options.abstract) {
    // 抽象组件不保存 props 和监听器意外的任何东西
    data = {}
  }

  // merge component management hooks onto the placeholder node
  mergeHooks(data)

  // return a placeholder vnode
  const name = Ctor.options.name || tag
  const vnode = new VNode(
    `vue-component-${Ctor.cid}${name ? `-${name}` : ''}`,
    data, undefined, undefined, undefined, undefined, context,
    { Ctor, propsData, listeners, tag, children }
  )
  return vnode
}
```

- [Vue.extend](../global-api/extend.md)
- [$forceUpdate](../instance/lifecycle.md#vueprototypeforceupdate)
- [warn](../util/debug.md#fn-warn)

_resolveAsyncComponent_

``` javascript
function resolveAsyncComponent (
  factory: Function,
  cb: Function
): Class<Component> | void {
  if (factory.requested) {
    // pool callbacks
    factory.pendingCallbacks.push(cb)
  } else {
    factory.requested = true
    const cbs = factory.pendingCallbacks = [cb]
    let sync = true
    factory(
      // resolve
      (res: Object | Class<Component>) => {
        if (isObject(res)) {
          res = Vue.extend(res)
        }
        // cache resolved
        factory.resolved = res
        // invoke callbacks only if this is not a synchronous resolve
        // (async resolves are shimmed as synchronous during SSR)
        if (!sync) {
          for (let i = 0, l = cbs.length; i < l; i++) {
            cbs[i](res)
          }
        }
      },
      // reject
      reason => {
        process.env.NODE_ENV !== 'production' && warn(
          `Failed to resolve async component: ${factory}` +
          (reason ? `\nReason: ${reason}` : '')
        )
      }
    )
    sync = false
    // return in case resolved synchronously
    return factory.resolved
  }
}
```


## [fn] createComponentInstanceForVnode

``` javascript
function createComponentInstanceForVnode (
  vnode: any, // we know it's MountedComponentVNode but flow doesn't
  parent: any // activeInstance in lifecycle state
): Component {
  const vnodeComponentOptions = vnode.componentOptions
  const options: InternalComponentOptions = {
    _isComponent: true,
    parent,
    propsData: vnodeComponentOptions.propsData,
    _componentTag: vnodeComponentOptions.tag,
    _parentVnode: vnode,
    _parentListeners: vnodeComponentOptions.listeners,
    _renderChildren: vnodeComponentOptions.children
  }
  // check inline-template render functions
  const inlineTemplate = vnode.data.inlineTemplate
  if (inlineTemplate) {
    options.render = inlineTemplate.render
    options.staticRenderFns = inlineTemplate.staticRenderFns
  }
  return new vnodeComponentOptions.Ctor(options)
}
```

- [VNode](./vnode.md)
- [normalizeChildren](./helpers.md#fn-normalizechildren)
- [activeInstance](../instance/lifecycle.md#any-activeinstance)
- [callHook](../instance/lifecycle.md#fn-callhook)
- [resolveSlots](../instance/render.md#fn-resolveslots)
- [validateProp](../util/props.md#fn-validateProp)
- [isObject](../../shared/util.md#fn-isobject)
- [hasOwn](../../shared/util.md#fn-hasown)
- [hyphenate](../../shared/util.md#fn-hyphenate)
