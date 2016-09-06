# Vue/core/util/options.js

设置配置项的合并策略，并提供合并函数

## [fn] mergeOptions

将两个配置对象合并为一个新的配置对象，使核心部分在实例和继承中均能使用。（待解）

``` javascript
function mergeOptions (
  parent: Object,
  child: Object,
  vm?: Component
): Object {
  normalizeComponents(child) // 🔽🔽🔽
  normalizeProps(child)      // 🔽🔽🔽
  normalizeDirectives(child) // 🔽🔽🔽

  const extendsFrom = child.extends
  // 递归合并，最终合并到 parent 上
  if (extendsFrom) {
    parent = typeof extendsFrom === 'function' // 是组件
      ? mergeOptions(parent, extendsFrom.options, vm)
      : mergeOptions(parent, extendsFrom, vm)
  }
  // 将子组件的混合项合并到父组件上
  if (child.mixins) {
    for (let i = 0, l = child.mixins.length; i < l; i++) {
      let mixin = child.mixins[i]
      if (mixin.prototype instanceof Vue) {
        // mixin 是组件
        mixin = mixin.options
      }
      parent = mergeOptions(parent, mixin, vm)
    }
  }
  const options = {}
  let key
  for (key in parent) {
    mergeField(key)
  }
  for (key in child) {
    if (!hasOwn(parent, key)) {
      mergeField(key)
    }
  }
  function mergeField (key) {
    // 选择合并策略 defaultStrat 🔽🔽🔽
    const strat = strats[key] || defaultStrat
    options[key] = strat(parent[key], child[key], vm, key)
  }
  return options
}
```

- [Vue](../instance/index.md#vue-vue)
- [hasOwn](../../shared/util.md#fn-hasown)

_[fn] normalizeComponents_

确保组件配置项被转换成真实的构造函数，[局部注册组件](http://vuejs.org.cn/guide/components.html#局部注册)

``` javascript
function normalizeComponents (options: Object) {
  if (options.components) {
    const components = options.components
    let def
    for (const key in components) {
      const lower = key.toLowerCase()
      if (isBuiltInTag(lower) || config.isReservedTag(lower)) {
        // slot, component || 被转换了的标签
        process.env.NODE_ENV !== 'production' && warn(
          'Do not use built-in or reserved HTML elements as component ' +
          'id: ' + key
        )
        continue
      }
      def = components[key]
      if (isPlainObject(def)) {
        components[key] = Vue.extend(def)
      }
    }
  }
}
```

- [config](../config.md)
- [isBuiltInTag](../../shared/util.md#fn-isbuiltintag)
- [isPlainObject](../../shared/util.md#fn-isplainobject)
- [Vue.extend](../global-api/extend.md#fn-initextend)


_[fn] normalizeProps_

确保所有的 props 配置项的语法标准化为基于对象的格式

``` javascript
function normalizeProps (options: Object) {
  const props = options.props
  if (!props) return
  const res = {}
  let i, val, name
  if (Array.isArray(props)) {
    // 数组形式 props， 转化为对象形式，不指定类型
    i = props.length
    while (i--) {
      val = props[i]
      if (typeof val === 'string') {
        // 连字符转化为驼峰
        name = camelize(val)
        res[name] = { type: null }
      } else if (process.env.NODE_ENV !== 'production') {
        warn('props must be strings when using array syntax.')
      }
    }
  } else if (isPlainObject(props)) {
    // 对象形式 props
    for (const key in props) {
      val = props[key]
      name = camelize(key)
      res[name] = isPlainObject(val)
        ? val // props: { foo: { type: String }}
        : { type: val } // props: { foo: String }
    }
  }
  options.props = res
}
```

- [camelize](../../shared/util.md#fn-camelize)
- [isPlainObject](../../shared/util.md#fn-isplainobject)


_[fn] normalizeDirectives_

将函数形式的指令转化为对象形式

``` javascript
function normalizeDirectives (options: Object) {
  const dirs = options.directives
  if (dirs) {
    for (const key in dirs) {
      const def = dirs[key]
      if (typeof def === 'function') {
        dirs[key] = { bind: def, update: def }
      }
    }
  }
}
```

_合并策略_

配置项的合并策略是由 **函数** 组成的，这些函数决定了父级与子级的配置项合并时生成最终值的策略。


_默认合并策略_

子级覆盖父级配置项

``` javascript
const strats = config.optionMergeStrategies

const defaultStrat = function (parentVal: any, childVal: any): any {
  return childVal === undefined
    ? parentVal
    : childVal
}
```

- [config](../config.md)

_el, propsData, name 合并策略_

``` javascript
if (process.env.NODE_ENV !== 'production') {
  strats.el = strats.propsData = function (parent, child, vm, key) {
    if (!vm) {
      warn(
        `option "${key}" can only be used during instance ` +
        'creation with the `new` keyword.'
      )
    }
    return defaultStrat(parent, child) // 🔼🔼🔼
  }

  strats.name = function (parent, child, vm) {
    if (vm && child) {
      warn(
        'options "name" can only be used as a component definition option, ' +
        'not during instance creation.'
      )
    }
    return defaultStrat(parent, child) // 🔼🔼🔼
  }
}
```

_data 合并策略_

``` javascript
strats.data = function (
  parentVal: any,
  childVal: any,
  vm?: Component
): ?Function {
  if (!vm) {
    // 在使用 Vue.extend 合并时, 合并双方都应为函数
    if (!childVal) {
      return parentVal
    }
    if (typeof childVal !== 'function') {
      process.env.NODE_ENV !== 'production' && warn(
        'The "data" option should be a function ' +
        'that returns a per-instance value in component ' +
        'definitions.',
        vm
      )
      return parentVal
    }
    if (!parentVal) {
      return childVal
    }
    // 当父级与子级的值都存在时，我们返回一个函数。
    // 这个函数返回传入的两个函数合并后的值。
    // 这里不必检查父级值是否为函数的原因是：
    // 如果它不是一个函数的话，就不能通过之前的合并。
    return function mergedDataFn () {
      // 将父级的值合并到子级上 🔽🔽🔽
      return mergeData(
        childVal.call(this),
        parentVal.call(this)
      )
    }
  } else if (parentVal || childVal) {
    return function mergedInstanceDataFn () {
      // instance merge
      const instanceData = typeof childVal === 'function'
        ? childVal.call(vm)
        : childVal
      const defaultData = typeof parentVal === 'function'
        ? parentVal.call(vm)
        : undefined
      if (instanceData) {
        return mergeData(instanceData, defaultData)
      } else {
        return defaultData
      }
    }
  }
}

// 递归合并 data 对象
function mergeData (to: Object, from: ?Object): Object {
  let key, toVal, fromVal
  for (key in from) {
    toVal = to[key]
    fromVal = from[key]
    if (!hasOwn(to, key)) {
      // 子级没有 key 才合并
      set(to, key, fromVal)
    } else if (isObject(toVal) && isObject(fromVal)) {
      mergeData(toVal, fromVal)
    }
  }
  return to
}

```

- [set](../observer/index.md#fn-set)
- [isObject](../../shared/util.md#fn-isobject)
- [hasOwn](../../shared/util.md#fn-hasown)

_钩子函数合并策略_

钩子函数和参数属性合并为数组

``` javascript
function mergeHook (
  parentVal: ?Array<Function>,
  childVal: ?Function | ?Array<Function>
): ?Array<Function> {
  return childVal
    ? parentVal
      ? parentVal.concat(childVal) // 有子级和父级值，添加到数组后
      : Array.isArray(childVal)    
        ? childVal                 // 有子级值无父级值，子级为数组
        : [childVal]               // 有子级值无父级值，子级不是数组
    : parentVal                    // 没有子级值
}

config._lifecycleHooks.forEach(hook => {
  strats[hook] = mergeHook
})
```

- [config](../config.md)


_config._assetTypes 合并策略_

资源。`['component', 'directive', 'filter']`

当实例存在的情况下。我们需要在构造函数，实例和父级的配置项中间做三项合并。

``` javascript
function mergeAssets (parentVal: ?Object, childVal: ?Object): Object {
  const res = Object.create(parentVal || null)
  return childVal
    ? extend(res, childVal)
    : res
}

config._assetTypes.forEach(function (type) {
  strats[type + 's'] = mergeAssets
})

```

- [config](../config.md)
- [extend](../../shared/util.md#fn-extend)

_watch 合并策略_

监视器不能覆盖，因此将它们合并到数组中。

``` javascript
strats.watch = function (parentVal: ?Object, childVal: ?Object): ?Object {
  // 无子级
  if (!childVal) return parentVal
  // 有子级无父级
  if (!parentVal) return childVal
  // 有子级 有父级
  const ret = {}
  extend(ret, parentVal)
  for (const key in childVal) {
    let parent = ret[key]
    const child = childVal[key]
    if (parent && !Array.isArray(parent)) {
      parent = [parent]
    }
    ret[key] = parent
      ? parent.concat(child)
      : [child]
  }
  return ret
}

```

- [extend](../../shared/util.md#fn-extend)

_props, methods, computed 合并策略_

``` javascript
strats.props =
strats.methods =
strats.computed = function (parentVal: ?Object, childVal: ?Object): ?Object {
  // 无自己值
  if (!childVal) return parentVal
  // 有子级值，无父级值
  if (!parentVal) return childVal
  // 有子级和父级值
  const ret = Object.create(null)
  extend(ret, parentVal)
  extend(ret, childVal)
  return ret
}

```

- [extend](../../shared/util.md#fn-extend)

## [fn] resolveAsset

解析一个资源

该函数在子实例需要访问定义在其原型链上的资源时使用。

``` javascript
function resolveAsset (
  options: Object,
  type: string,
  id: string,
  warnMissing?: boolean
): any {
  /* istanbul ignore if */
  if (typeof id !== 'string') {
    return
  }
  const assets = options[type]
  const res = assets[id] ||
    // camelCase ID
    assets[camelize(id)] ||
    // Pascal Case ID
    assets[capitalize(camelize(id))]
  if (process.env.NODE_ENV !== 'production' && warnMissing && !res) {
    warn(
      'Failed to resolve ' + type.slice(0, -1) + ': ' + id,
      options
    )
  }
  return res
}
```

- [camelize](../../shared/util.md#fn-camelize)
- [capitalize](../../shared/util.md#fn-capitalize)
