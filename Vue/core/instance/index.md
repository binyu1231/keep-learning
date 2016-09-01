# Vue/core/install

| name | content |
|:---:|:---|
|`index.js`|导出|
|`event.js`|$on $off $once $emit|
|`init.js`|Vue.prototype._init 初始化|
|`lifecycle.js`|实现生命周期|
|`proxy.js`||
|`render.js`|render 选项|
|`state.js`|props/data/computed/methods/watch 选项|

# index.js

## [Vue] Vue

``` javascript
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'

function Vue (options) {
  this._init(options)
}

// 添加 Vue.prototype._init
initMixin(Vue)

// 添加 Vue.prototype.$data/$watch/$set/$delete      
stateMixin(Vue)

// 添加 Vue.prototype.$on/$off/$once/$emit
eventsMixin(Vue)

// 添加 Vue.prototype._mount/_update/_updateFromParent/$forceUpdate/$destroy  
lifecycleMixin(Vue)

// 2.0 新增 render 选项

renderMixin(Vue)

// 导出 Vue 构造函数
export default Vue
```
