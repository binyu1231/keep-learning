# Vue/core/instance

| name | content |
|:---:|:---|
|index.js|导出|
|[event.js](./events.md)|$on $off $once $emit|
|[init.js](./init.md)|Vue.prototype._init 初始化|
|[lifecycle.js](./lifecycle.md)|实现生命周期|
|[proxy.js](./proxy.md)||
|[render.js](./render.md)|render 选项|
|[state.js](./state.md)|props/data/computed/methods/watch 选项|

# index.js

## [Vue] Vue

``` javascript
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

- [initMixin](./init.md#fn-initmixin)
- [stateMixin](./state.md#fn-statemixin)
- [renderMixin](./render.md#fn-rendermixin)
- [eventsMixin](./events.md#fn-eventsmixin)
- [lifecycleMixin](./lifecycle.md#fn-lifecyclemixin)
