# Vue/core/global-api/extend.js

## [fn] initExtend

initExtend 为 Vue 添加一个 extend 函数，注册全局组件。使注册的组件继承自 Vue

``` javascript
function initExtend (Vue: GlobalAPI) {
  // 任何实例构造函数都有唯一的 cid，
  // 这使得我们可以针对原型继承和缓存功能创建可嵌套的“子构造函数”
  Vue.cid = 0
  let cid = 1

  Vue.extend = function (extendOptions: Object): Function {
    extendOptions = extendOptions || {}
    const Super = this
    const isFirstExtend = Super.cid === 0
    if (isFirstExtend && extendOptions._Ctor) {
      return extendOptions._Ctor
    }
    let name = extendOptions.name || Super.options.name
    // 检测组件名是否合法
    if (process.env.NODE_ENV !== 'production') {
      if (!/^[a-zA-Z][\w-]*$/.test(name)) {
        warn(
          'Invalid component name: "' + name + '". Component names ' +
          'can only contain alphanumeric characaters and the hyphen.'
        )
        name = null
      }
    }
    // 创建 VueComponent 使其继承自 Vue
    const Sub = function VueComponent (options) {
      this._init(options)
    }
    Sub.prototype = Object.create(Super.prototype)
    Sub.prototype.constructor = Sub
    Sub.cid = cid++ // 生成唯一的 cid
    Sub.options = mergeOptions(
      Super.options,
      extendOptions // 合并配置
    )
    // 可以使用 super 访问 Vue
    Sub['super'] = Super
    // 允许嵌套扩展
    Sub.extend = Super.extend
    // 创建资源注册机制，使子类也可以拥有私有资源（asset）
    // 'components', 'directives', 'filters'
    config._assetTypes.forEach(function (type) {
      Sub[type] = Super[type]
    })
    // 启用递归查找: 可以通过组件名访问自身
    if (name) {
      Sub.options.components[name] = Sub
    }
    // 在扩展时保留 Vue 配置的引用
    // 在之后的实例化中可以用来检测 Vue 的配置是否发生改变（update）。
    Sub.superOptions = Super.options
    Sub.extendOptions = extendOptions
    // 缓存构造函数
    if (isFirstExtend) {
      extendOptions._Ctor = Sub
    }
    return Sub
  }
}
```
