# Vue/core/global-api/assets.js

## [fn] initAssetRegisters

将 config 中的配置（'component', 'directive', 'filter'）方法注册到 Vue 上

``` javascript
function initAssetRegisters (Vue: GlobalAPI) {
  // config._assetTypes 数组中有
  // 'component', 'directive', 'filter'
  // 将他们注册为 Vue 的全局函数
  config._assetTypes.forEach(type => {
    Vue[type] = function (
      id: string,
      definition: Function | Object
    ): Function | Object | void {
      if (!definition) {
        return this.options[type + 's'][id]
      } else {
        // 判断组件名是否是保留字
        if (process.env.NODE_ENV !== 'production') {
          if (type === 'component' && config.isReservedTag(id)) {
            warn(
              'Do not use built-in or reserved HTML elements as component ' +
              'id: ' + id
            )
          }
        }
        // 组件的定义（definition）是纯 js 对象 => 使用 extend 注册为全局组件
        if (type === 'component' && isPlainObject(definition)) {
          definition.name = definition.name || id
          definition = Vue.extend(definition)
        }
        // 指令的定义（definition）是函数 => 初始化绑定与更新
        if (type === 'directive' && typeof definition === 'function') {
          definition = { bind: definition, update: definition }
        }
        this.options[type + 's'][id] = definition
        return definition
      }
    }
  })
}
```
