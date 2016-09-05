## Vue/core/vdom/vnode.js

## [class] VNode

``` javascript
class VNode {
  constructor (
    tag?: string,        // 标签名
    data?: VNodeData,
    children?: Array<VNode> | void,
    text?: string,
    elm?: Node,
    ns?: string | void,  // 命名空间
    context?: Component, // 在这个组件的作用于下渲染
    componentOptions?: VNodeComponentOptions // 虚拟节点是组件的判断标准
  ) {
    this.tag = tag
    this.data = data
    this.children = children
    this.text = text
    this.elm = elm
    this.ns = ns
    this.context = context
    this.key = data && data.key
    this.componentOptions = componentOptions
    this.child = undefined   // 组件实例
    this.parent = undefined  // 组件占位节点
    this.raw = false         // 包含原始的 HTML
    this.isStatic = false    // 挂起的静态节点
    this.isRootInsert = true // 进入过渡检查的必要项
    this.isComment = false   // 是注释
    // 调用构造函数的钩子
    // 这个钩子函数在渲染（render）期间被调用，在修补（patch）之前
    // 不同于其他钩子函数，服务器端和客户端都会被调用。
    const constructHook = data && data.hook && data.hook.construct
    if (constructHook) {
      constructHook(this)
    }
  }
}
```

### [fn] emptyVNode

返回一个空的虚拟节点

``` javascript
const emptyVNode = () => {
  const node = new VNode()
  node.text = ''
  node.isComment = true
  return node
}
```
