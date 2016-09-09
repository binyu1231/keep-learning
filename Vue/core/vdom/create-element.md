# Vue/core/vdom/create-element.js

## [fn] createElement

包装了一层函数，提供更为灵活的接口形式。

``` javascript
function createElement (
  tag: any,
  data: any,
  children: any
): VNode | Array<VNode> | void {
  if (data && (Array.isArray(data) || typeof data !== 'object')) {
    children = data
    data = undefined
  }
  // 请务必使用真正的实例，而不是代理作为上下文
  return _createElement(this._self, tag, data, children)
}
```

_[fn] _createElement_

``` javascript

function _createElement (
  context: Component,
  tag?: string | Class<Component> | Function | Object,
  data?: VNodeData,
  children?: VNodeChildren | void
): VNode | Array<VNode> | void {
  if (data && data.__ob__) {
    process.env.NODE_ENV !== 'production' && warn(
      `Avoid using observed data object as vnode data: ${JSON.stringify(data)}\n` +
      'Always create fresh vnode data objects in each render!',
      context
    )
    return
  }
  if (!tag) {
    // 是组件的时候，设置为假值
    return emptyVNode()
  }
  if (typeof tag === 'string') {
    let Ctor
    const ns = config.getTagNamespace(tag)
    if (config.isReservedTag(tag)) {
      // 平台保留标签
      return new VNode(
        tag, data, normalizeChildren(children, ns),
        undefined, undefined, ns, context
      )
    } else if ((Ctor = resolveAsset(context.$options, 'components', tag))) {
      // 组件
      return createComponent(Ctor, data, context, children, tag)
    } else {
      // 未知 || 命名空间中没有列出的元素
      // 在运行时检查的原因是因为：
      // 当它的父级标准化他的所有子级时，
      // 它有可能会被分配一个命名空间。
      return new VNode(
        tag, data, normalizeChildren(children, ns),
        undefined, undefined, ns, context
      )
    }
  } else {
    // 组件配置对象，构造函数
    return createComponent(tag, data, context, children)
  }
}
```

- [VNode](./vnode.md#class-vnode)
- [emptyVNode](./vnode.md#fn-emptynode)
- [config](../config.md)
- [createComponent](./create-element.md#fn-createcomponent)
- [normalizeChildren](./helpers.md#fn-normalizechildren)
- [resolveAsset](../util/options.md#fn-resolveasset)
