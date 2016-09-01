# Vue/core/global-api/use.js

## [fn] initUse

initUse 为 Vue 添加一个 use 函数，use 函数可以用来添加 Vue 兼容的插件，如 Vuex, VueRouter, VueGesture 等等。

``` javascript
function initUse (Vue: GlobalAPI) {
  Vue.use = function (plugin: Function | Object) {
    // 避免重复注册
    if (plugin.installed) {
      return
    }
    // 将第一个参数替换为 Vue，然后下面将参数数组传递给相应的插件，完成相应的安装
    const args = toArray(arguments, 1)
    args.unshift(this)
    if (typeof plugin.install === 'function') {
      plugin.install.apply(plugin, args)
    } else {
      plugin.apply(null, args)
    }
    plugin.installed = true
    return this
  }
}
```

- [toArray](../../shared/util.md#fn-toarray)
