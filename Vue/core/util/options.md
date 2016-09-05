# Vue/core/util/options.js

配置项会覆盖父选项和子选项的合并策略

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
    // 选择合并策略
    const strat = strats[key] || defaultStrat
    options[key] = strat(parent[key], child[key], vm, key)
  }
  return options
}
```

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

## [fn] resolveAsset

☆

``` javascript
/**
 * Resolve an asset.
 * This function is used because child instances need access
 * to assets defined in its ancestor chain.
 */
export function resolveAsset (
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


- [Vue](../instance/index.md#vue-vue)
- [config](../config.md)
- [warn](../util/debug.md#fn-warn)
- [set](../observer/index.md#fn-set)
- [extend](../../shared/util.md#fn-extend)
- [isObject](../../shared/util.md#fn-isobject)
- [isPlainObject](../../shared/util.md#fn-isplainobject)
- [hasOwn](../../shared/util.md#fn-hasown)
- [capitalize](../../shared/util.md#fn-capitalize)
- [isBuiltInTag](../../shared/util.md#fn-isbuiltintag)
