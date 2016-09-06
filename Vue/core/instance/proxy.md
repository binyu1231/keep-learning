## Vue/core/instance/proxy.js

### [fn] initProxy

``` javascript
let hasProxy, proxyHandlers, initProxy

if (process.env.NODE_ENV !== 'production') {

  // 允许使用的全局 API

  const allowedGlobals = makeMap(
    'Infinity,undefined,NaN,isFinite,isNaN,' +
    'parseFloat,parseInt,decodeURI,decodeURIComponent,encodeURI,encodeURIComponent,' +
    'Math,Number,Date,Array,Object,Boolean,String,RegExp,Map,Set,JSON,Intl,' +
    'require' // for Webpack/Browserify
  )

  // Proxy 兼容性判断
  hasProxy =
    typeof Proxy !== 'undefined' &&
    Proxy.toString().match(/native code/)

  proxyHandlers = {
    has (target, key) {
      const has = key in target
      const isAllowed = allowedGlobals(key) || key.charAt(0) === '_'
      if (!has && !isAllowed) {
        warn(
          `Property or method "${key}" is not defined on the instance but ` +
          `referenced during render. Make sure to declare reactive data ` +
          `properties in the data option.`,
          target
        )
      }
      return has || !isAllowed
    }
  }

  initProxy = function initProxy (vm) {
    if (hasProxy) {
      vm._renderProxy = new Proxy(vm, proxyHandlers)
    } else {
      vm._renderProxy = vm
    }
  }
}

export { initProxy }
```

- [Proxy 代理](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)
