# Vue/core/instance/render.js

## [fn] initRender

``` javascript
function initRender (vm: Component) {
  vm.$vnode = null // 父树的占位节点
  vm._vnode = null // 子树的根虚拟节点
  vm._staticTrees = null
  // 🔽🔽🔽
  vm.$slots = resolveSlots(vm.$options._renderChildren)
  // 将公共的 createElement 函数绑定到实例上
  // 以便我们能在其中得到是当的渲染上下文环境
  vm.$createElement = bind(createElement, vm)
  if (vm.$options.el) {
    vm.$mount(vm.$options.el)
  }
}
```

- [createElement](../vdom/create-element.md#fn-createelement)
- [$mount](../entries/web-runtime-with-compiler.md)

### [fn] resolveSlots

解析 slots

``` javascript
function resolveSlots (
  renderChildren: ?VNodeChildren
): { [key: string]: Array<VNode> } {
  const slots = {}
  if (!renderChildren) {
    return slots
  }
  // 标准化子节点
  const children = normalizeChildren(renderChildren) || []
  const defaultSlot = []
  let name, child
  for (let i = 0, l = children.length; i < l; i++) {
    child = children[i]
    if (child.data && (name = child.data.slot)) {
      delete child.data.slot
      const slot = (slots[name] || (slots[name] = []))
      // 忽略 template 标签
      if (child.tag === 'template') {
        slot.push.apply(slot, child.children)
      } else {
        slot.push(child)
      }
    } else {
      defaultSlot.push(child)
    }
  }
  // 忽略单个空格
  if (defaultSlot.length && !(
    defaultSlot.length === 1 &&
    defaultSlot[0].text === ' '
  )) {
    slots.default = defaultSlot
  }
  return slots
}
```

- [normalizeChildren](../vdom/helpers.md#fn-normalizechildren)

### [fn] renderMixin

添加实例方法

### Vue.prototype.$nextTick

``` javascript
Vue.prototype.$nextTick = function (fn: Function) {
  nextTick(fn, this)
}
```

- [nextTick](../util/env.md#fn-nexttick)

### ☆ Vue.prototype.\_render


``` javascript
Vue.prototype._render = function (): VNode {
  const vm: Component = this
  const {
    render,
    staticRenderFns,
    _parentVnode
  } = vm.$options

  if (staticRenderFns && !vm._staticTrees) {
    vm._staticTrees = []
  }
  // 设置父级虚拟节点，允许渲染函数有权访问占位节点上的数据
  vm.$vnode = _parentVnode
  // 渲染自身
  let vnode
  try {
    // 配置项中的 render 函数，函数接收一个 vm.$createElement 函数来创建元素
    vnode = render.call(vm._renderProxy, vm.$createElement)
  } catch (e) {
    if (process.env.NODE_ENV !== 'production') {
      warn(`Error when rendering ${formatComponentName(vm)}:`)
    }
    /* istanbul ignore else */
    if (config.errorHandler) {
      config.errorHandler.call(null, e, vm)
    } else {
      if (config._isServer) {
        throw e
      } else {
        setTimeout(() => { throw e }, 0)
      }
    }
    // 返回前一个虚拟节点，防止渲染错误导致产生的空白组件
    vnode = vm._vnode
  }
  // 在渲染函数出错的情况下返回空的虚拟节点
  if (!(vnode instanceof VNode)) {
    if (process.env.NODE_ENV !== 'production' && Array.isArray(vnode)) {
      warn(
        'Multiple root nodes returned from render function. Render function ' +
        'should return a single root node.',
        vm
      )
    }
    vnode = emptyVNode()
  }
  // 设置父级
  vnode.parent = _parentVnode
  return vnode
}
```

- [关于 rennder 中的 JSX 语法](https://github.com/vuejs/babel-plugin-transform-vue-jsx)
- [config](../config.md)
- [emptyVNode](../vdom/vnode.md#fn-emptyvnode)

### ☆ Vue.prototype.\_h/\_s/\_n/\_m/\_f/\_l/\_b/\_k

``` javascript
function renderMixin (Vue: Class<Component>) {
  // 渲染函数的简写形式
  Vue.prototype._h = createElement
  // toString for mustaches
  Vue.prototype._s = _toString
  // 转化为数字
  Vue.prototype._n = toNumber

  // 使用索引渲染静态树
  Vue.prototype._m = function renderStatic (
    index: number,
    isInFor?: boolean
  ): VNode | VNodeChildren {
    let tree = this._staticTrees[index]
    // 如果已经存在被渲染过的静态树，并且其中没有 v-for 指令
    // 我们可以使用相同的身份重用这个树
    if (tree && !isInFor) {
      return tree
    }
    // 否则就渲染一个动态的树
    tree = this._staticTrees[index] = this.$options.staticRenderFns[index].call(this._renderProxy)
    if (Array.isArray(tree)) {
      for (let i = 0; i < tree.length; i++) {
        tree[i].isStatic = true
        tree[i].key = `__static__${index}_${i}`
      }
    } else {
      tree.isStatic = true
      tree.key = `__static__${index}`
    }
    return tree
  }


  const identity = _ => _

  // filter 的解析函数
  Vue.prototype._f = function resolveFilter (id) {
    return resolveAsset(this.$options, 'filters', id, true) || identity
  }

  // 渲染 v-for
  Vue.prototype._l = function renderList (
    val: any,
    render: () => VNode
  ): ?Array<VNode> {
    let ret: ?Array<VNode>, i, l, keys, key
    if (Array.isArray(val)) {
      ret = new Array(val.length)
      for (i = 0, l = val.length; i < l; i++) {
        ret[i] = render(val[i], i)
      }
    } else if (typeof val === 'number') {
      ret = new Array(val)
      for (i = 0; i < val; i++) {
        ret[i] = render(i + 1, i)
      }
    } else if (isObject(val)) {
      keys = Object.keys(val)
      ret = new Array(keys.length)
      for (i = 0, l = keys.length; i < l; i++) {
        key = keys[i]
        ret[i] = render(val[key], key, i)
      }
    }
    return ret
  }

  // 处理 v-bind 对象
  Vue.prototype._b = function bindProps (
    vnode: VNodeWithData,
    value: any,
    asProp?: boolean) {
    if (value) {
      if (!isObject(value)) {
        process.env.NODE_ENV !== 'production' && warn(
          'v-bind without argument expects an Object or Array value',
          this
        )
      } else {
        if (Array.isArray(value)) {
          value = toObject(value)
        }
        const data: any = vnode.data
        for (const key in value) {
          if (key === 'class' || key === 'style') {
            data[key] = value[key]
          } else {
            // 判断是 prop 还是标签的属性
            const hash = asProp || config.mustUseProp(key)
              ? data.domProps || (data.domProps = {})
              : data.attrs || (data.attrs = {})
            hash[key] = value[key]
          }
        }
      }
    }
  }

  // 暴露 v-on 键盘编码
  Vue.prototype._k = function getKeyCodes (key: string): any {
    return config.keyCodes[key]
  }
}
```

- [resolveAsset](../util/options.md#fn-resolveasset)
- [config](../config.md)
- [toObject](../../shared/util.md#fn-toobject)
