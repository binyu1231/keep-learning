# Vue/core/components/keep-alive.js

keep-alive 组件的实现

> 如果你想要开关组件的活性，那么你可以添加一个 keep-alive 指令参数来保存组件的状态或者避免组件的重新渲染。- rc.vue.org

``` html
<component :is="currentView" keep-alive>
  <!-- inactive components will be cached! -->
</component>
```

## [Component] KeepAlive

``` javascript
export default {
  name: 'keep-alive',
  abstract: true, // 抽象组件
  created () {
    this.cache = Object.create(null)
  },
  render () {
    // getFirstComponentChild => core/vdom/helpers
    // 获取第一个有 componentOptions 子组件
    const vnode = getFirstComponentChild(this.$slots.default)
    if (vnode && vnode.componentOptions) {
      const opts = vnode.componentOptions
      // 缓存用键名
      const key = vnode.key == null
        // 相同的构造函数可能会被注册为不同的局部组件
        // 所以只有 cid 是不够的 (#3269)
        ? opts.Ctor.cid + '::' + opts.tag
        : vnode.key
      if (this.cache[key]) {
        vnode.child = this.cache[key].child
      } else {
        this.cache[key] = vnode
      }
      // ？在哪识别
      vnode.data.keepAlive = true
    }
    return vnode
  },
  destroyed () {
    // 销毁之后销毁子组件
    for (const key in this.cache) {
      const vnode = this.cache[key]
      callHook(vnode.child, 'deactivated')
      vnode.child.$destroy()
    }
  }
}
```

- [callHook](../instance/lifecycle.md#fn-callhook)
- [getFirstComponentChild](../vdom/helpers.md#fn-getfirstcomponentchild)
