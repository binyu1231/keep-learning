# Vue/core/util/lang.js

## [fn] isReserved

检查是否以 $ 和下划线开头

``` javascript
function isReserved (str: string): boolean {
  const c = (str + '').charCodeAt(0)
  return c === 0x24 || c === 0x5F
}
```
## [fn] def

封装了一个为对象添加属性的简化函数

``` javascript
function def (obj: Object, key: string, val: any, enumerable?: boolean) {
  Object.defineProperty(obj, key, {
    value: val,
    enumerable: !!enumerable, // 是否可枚举
    writable: true,
    configurable: true
  })
}
```

## [fn] parsePath

何处使用:
- `core/observer/watcher.js` 将表达式转化为 getter 函数

如果字符串中没有非字母，非. 非$，则返回一个函数用于找到某个对象中具体路径

``` javascript
const bailRE = /[^\w\.\$]/
function parsePath (path: string): any {
  // 包含非字母，非. 非$
  if (bailRE.test(path)) {
    return
  } else {
    const segments = path.split('.')
    return function (obj) {
      for (let i = 0; i < segments.length; i++) {
        if (!obj) return
        obj = obj[segments[i]]
      }
      return obj
    }
  }
}

// eg:
let colorPath = parsePath('$some.color')
colorPath(null) // => undefined
colorPath({ $any: 12, $some: { color: 'red' } }) // => 'red'
```
