# Vue/core/util/debug.js

非生产环境下，会导出两个函数 formatComponentName 和 warn

## [fn] formatComponentName

返回组件的名字。可能的情况有：根实例，有名字的组件，匿名组件

``` javascript
formatComponentName = vm => {
  if (vm.$root === vm) {
    return 'root instance'
  }
  const name = vm._isVue
    ? vm.$options.name || vm.$options._componentTag
    : vm.name
  return name ? `component <${name}>` : `anonymous component`
}
```

## [fn] warn

打印错误信息和位置。

``` javascript
const hasConsole = typeof console !== 'undefined'

warn = (msg, vm) => {
  // console 存在，非静默
  // 在 core/config.js 中配置
  if (hasConsole && (!config.silent)) {
    console.error(`[Vue warn]: ${msg} ` + (
      // 🔽🔽🔽
      vm ? formatLocation(formatComponentName(vm)) : ''
    ))
  }
}
```

- [config](../config.md)

_[fn] formatLocation_

``` javascript
const formatLocation = str => {
  if (str === 'anonymous component') {
    str += ` - use the "name" option for better debugging messages.`
  }
  return `(found in ${str})`
}
```
