# Vue/core/global-api/mixin.js

## [fn] initMixin

initMixin 为 Vue 添加一个 mixin 函数，mixin 函数将配置合并到 Vue.options 上。既添加全局混合。

``` javascript
function initMixin (Vue: GlobalAPI) {
  Vue.mixin = function (mixin: Object) {
    Vue.options = mergeOptions(Vue.options, mixin)
  }
}
```

- [mergeOptions](../util/options.md#fn-mergeoptions)
