# Vue/core/global-api

| name | content |
|:---:|:---|
|[index.js](#indexjs)|暴露了一些工具函数给其他部分用|
|[assets.js](./assets.md)|将 config 中的配置（'component', 'directive', 'filter'）方法注册到 Vue 上|
|[extend.js](./extend.md)| 添加 Vue.extend |
|[mixin.js](./mixin.md)| 添加 Vue.mixin |
|[use.js](./use.md)| 添加 Vue.use |


/core/global-api 中的代码比较好理解，主要在 Vue 上添加一些公用方法。经过梳理我们大概就知道了。Vue的外围是什么样的。

# index.js

## [fn] initGlobalAPI

``` javascript
function initGlobalAPI (Vue: GlobalAPI) {
  // config
  const configDef = {}
  configDef.get = () => config
  if (process.env.NODE_ENV !== 'production') {
    configDef.set = () => {
      util.warn(
        'Do not replace the Vue.config object, set individual fields instead.'
      )
    }
  }
  // Vue.config.get => config /core/config.js
  Object.defineProperty(Vue, 'config', configDef)
  Vue.util = util // 暴露 util /core/util
  Vue.set = set // 设置对象属性函数 /observer/index
  Vue.delete = del // 删除对象属性函数 /observer/index
  Vue.nextTick = util.nextTick // 异步执行一组函数 /core/util/env.js

  // 创建配置对象，用于存储创建的 'component', 'directive', 'filter' 的 id
  Vue.options = Object.create(null)
  config._assetTypes.forEach(type => {
    Vue.options[type + 's'] = Object.create(null)
  })
  // 创建 Vue 自带组件
  util.extend(Vue.options.components, builtInComponents)

  initUse(Vue) // 初始化 Vue.use 方法
  initMixin(Vue) // 初始化 Vue.mixin 方法
  initExtend(Vue) // 初始化 Vue.extend 方法
  initAssetRegisters(Vue) // 初始化 Vue.component Vue.directive Vue.filter
}
```


- [util](../util/index.md)
- [util.extend](../../shared/util.md#fn-extend)
- [util.nextTick](../util/env.md#fn-nexttick)
- [set](../observer/index.md#fn-set)
- [del](../observer/index.md#fn-del)
- [builtInComponents](../components/index.md#indexjs)
- [initUse](./use.md#fn-inituse)
- [initMixin](./mixin.md#fn-initmixin)
- [initExtend](./extend.md#fn-initextend)
- [initAssetRegisters](./assets.md#fn-initassetregisters)
