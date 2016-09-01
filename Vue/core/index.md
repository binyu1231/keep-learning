# Vue/core

| name | content |
|:---:|:---|
|[index.js](#indexjs)|导出 Vue|
|[config.js](./config.md)|基本设置|
|[instance/](./instance/index.md)|Vue 实现|
|[observer/](./observer/index.md)|Observer 实现|
|[components/](./components/index.md)| keep-alive |
|[vdom/](./vdom/index.md)|虚拟 DOM|
|[global-api/](./global-api/index.md)|全局 API|
|[util/](./util/index.md)|工具函数|

# index.js

## [Vue] Vue

``` javascript
// 初始化全局 API
initGlobalAPI(Vue)

// 是否是服务端渲染
Object.defineProperty(Vue.prototype, '$isServer', {
  get: () => config._isServer
})

// 当前版本
Vue.version = '2.0.0-rc.3'

export default Vue

```

- [initGlobalAPI](./global-api/index.md#fn-initglobalapi)
- [Vue](./instance/index.md#vue-vue)
- [config](./config.md)
