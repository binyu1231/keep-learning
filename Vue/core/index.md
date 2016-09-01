# Vue/core

# index.js

## [Vue] Vue

``` javascript
import config from './config'
import { initGlobalAPI } from './global-api/index'
import Vue from './instance/index'

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
