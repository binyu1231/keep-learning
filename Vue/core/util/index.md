# Vue/core/util

| name | content |
|:---:|:---|
|`index.js`|导出|
|`debug.js`|warn formatComponentName 两个调试函数|
|`env.js`|检测环境，实现兼容的 nextTick 和 _Set|
|`lang.js`|作用于变量的工具函数|
|`options.js`|对配置项进行操作|
|`props.js`|验证组件的属性|

# index.js

将所有文件导出，方便外部调用 eg:

``` javascript
import { nextTick } from 'core/util/env'
// 🔽
import { nextTick } from 'core/util'
```

``` javascript
export * from 'shared/util'
export * from './lang'
export * from './env'
export * from './options'
export * from './debug'
export * from './props'
export { defineReactive } from '../observer/index'
```
