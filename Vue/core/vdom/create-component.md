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
        // return nothing if this is indeed an async component
        // wait for the callback to trigger parent update.
        return
      }
    }
  }

  data = data || {}

  // 提取 props
  const propsData = extractProps(data, Ctor)

  // functional component
  if (Ctor.options.functional) {
    return createFunctionalComponent(Ctor, propsData, data, context, children)
  }

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

- [Vue.extend](../global-api/extend.md)
- [VNode](./vnode.md)
- [normalizeChildren](./helpers.md#fn-normalizechildren)
- [activeInstance](../instance/lifecycle.md#any-activeinstance)
- [callHook](../instance/lifecycle.md#fn-callhook)
- [$forceUpdate](../instance/lifecycle.md#vueprototypeforceupdate)
- [resolveSlots](../instance/render.md#fn-resolveslots)
- [warn](../util/debug.md#fn-warn)
- [validateProp](../util/props.md#fn-validateProp)
- [isObject](../../shared/util.md#fn-isobject)
- [hasOwn](../../shared/util.md#fn-hasown)
- [hyphenate](../../shared/util.md#fn-hyphenate)
