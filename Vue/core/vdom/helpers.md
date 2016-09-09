# Vue/core/vdom/helpers.js

## [fn] normalizeChildren

将子元素规范化，将相邻的原始类型（`number`，`string`）的子元素合并到一个虚拟节点中

``` javascript
function normalizeChildren (
  children: any,
  ns: string | void
): Array<VNode> | void {
  if (isPrimitive(children)) {
    // 将原始类型转化为 vnode
    return [createTextVNode(children)] // 🔽🔽🔽
  }
  // 多个子元素
  if (Array.isArray(children)) {
    const res = [] // 标准化后子元素的容器
    for (let i = 0, l = children.length; i < l; i++) {
      const c = children[i]
      const last = res[res.length - 1] // 当前循环轮次 res 最后一个元素

      if (Array.isArray(c)) {
        // 子元素嵌套，递归操作
        res.push.apply(res, normalizeChildren(c, ns))
      } else if (isPrimitive(c)) {
        // 子元素为原始类型
        if (last && last.text) {
          // 上一次循环结束之后 res 的最后一项 text 项存在
          last.text += String(c)
        } else if (c !== '') {
          // last.text 不存在（res 中没有元素）
          // 且子元素不为空,
          // 将原始类型转化为文本虚拟节点 🔽🔽🔽
          res.push(createTextVNode(c))
        }
      } else if (c instanceof VNode) {
        // c 是 VNode 实例
        if (c.text && last && last.text) {
          // 不是 res 第一个元素
          // 是文本的虚拟节点
          last.text += c.text
        } else {
          // 不是文本节点 || 是res第一个一个元素 || 或者 res 第一个元素不是文本虚拟节点
          // 继承父级的命名空间
          if (ns) {
            applyNS(c, ns)
          }
          res.push(c)
        }
      }
    }
    return res
  }
}
```

- [isPrimitive](../../shared/util.md#fn-isprimitive)

_[fn] createTextVNode_

创建文本类型的虚拟节点（`VNode`）

``` javascript
function createTextVNode (val) {
  // 参数 tag, data, children, text
  return new VNode(undefined, undefined, undefined, String(val))
}

```

- [VNode](./vnode.md)

_[fn] applyNS_

分配命名空间

``` javascript
function applyNS (vnode, ns) {
  if (vnode.tag && !vnode.ns) {
    // 非文本节点，命名空间不存在
    vnode.ns = ns
    if (vnode.children) {
      // 子元素继承命名空间
      for (let i = 0, l = vnode.children.length; i < l; i++) {
        applyNS(vnode.children[i], ns)
      }
    }
  }
}
```

## [fn] getFirstComponentChild

获取第一个是组件的子元素

``` javascript
function getFirstComponentChild (children: ?Array<any>) {
  return children && children.filter(c => c && c.componentOptions)[0]
}
```

## [fn] mergeVNodeHook

用来将合并 VNode 实例的钩子函数

``` javascript
function mergeVNodeHook (def: Object, key: string, hook: Function) {
  const oldHook = def[key]
  if (oldHook) {
    def[key] = function () {
      oldHook.apply(this, arguments)
      hook.apply(this, arguments)
    }
  } else {
    def[key] = hook
  }
}
```

## [fn] updateListeners

更新监听器

使用:

- [initEvents](../instance/events.md#fn-initevents)

``` javascript
function updateListeners (
  on: Object,
  oldOn: Object,
  add: Function,
  remove: Function
) {
  let name, cur, old, fn, event, capture
  for (name in on) {
    cur = on[name]    // 需要更新的监听器
    old = oldOn[name] // 同名旧的更新器
    if (!cur) {
      // 更新项未定义
      process.env.NODE_ENV !== 'production' && warn(
        `Handler for event "${name}" is undefined.`
      )
    } else if (!old) {
      // 需要更新的监听器存在，同名旧的更新器不存在

      // 截取名字 ？什么情况下会有 ！
      capture = name.charAt(0) === '!'
      event = capture ? name.slice(1) : name
      if (Array.isArray(cur)) {
        // $on 不接受 capture ！
        // arrInvoker 🔽🔽🔽
        add(event, (cur.invoker = arrInvoker(cur)), capture)
      } else {
        // cur 由 fn => { fn: }
        fn = cur
        cur = on[name] = {}
        cur.fn = fn
        // fnInvoker 🔽🔽🔽
        add(event, (cur.invoker = fnInvoker(cur)), capture)
      }
    } else if (Array.isArray(old)) {
      old.length = cur.length
      for (let i = 0; i < old.length; i++) old[i] = cur[i]
      on[name] = old
    } else {
      old.fn = cur
      on[name] = old
    }
  }
  for (name in oldOn) {
    if (!on[name]) {
      event = name.charAt(0) === '!' ? name.slice(1) : name
      remove(event, oldOn[name].invoker)
    }
  }
}
```

_[fn] arrInvoker_

``` javascript
function arrInvoker (arr: Array<Function>): Function {
  return function (ev) {
    // 多参数时，防止更改 this 指向
    const single = arguments.length === 1
    for (let i = 0; i < arr.length; i++) {
      single ? arr[i](ev) : arr[i].apply(null, arguments)
    }
  }
}

```

_[fn] fnInvoker_

``` javascript

function fnInvoker (o: { fn: Function }): Function {
  return function (ev) {
    // 多参数时，防止更改 this 指向
    const single = arguments.length === 1
    single ? o.fn(ev) : o.fn.apply(null, arguments)
  }
}
```
