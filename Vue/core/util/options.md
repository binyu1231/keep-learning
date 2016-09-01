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
  normalizeComponents(child)
  normalizeProps(child)
  normalizeDirectives(child)
  const extendsFrom = child.extends
  // 递归合并
  if (extendsFrom) {
    parent = typeof extendsFrom === 'function'
      ? mergeOptions(parent, extendsFrom.options, vm)
      : mergeOptions(parent, extendsFrom, vm)
  }
  // 将子组件的混合项合并到父组件上
  if (child.mixins) {
    for (let i = 0, l = child.mixins.length; i < l; i++) {
      let mixin = child.mixins[i]
      if (mixin.prototype instanceof Vue) {
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
    const strat = strats[key] || defaultStrat
    options[key] = strat(parent[key], child[key], vm, key)
  }
  return options
}
```

## ☆ [fn] resolveAsset
（待解）
