# Vue/shared/util.js

## [fn] \_toString

接收任何类型的参数并转化为字符串返回。null 返回空字符串，对象则返回缩进为2的对象型的字符串。

``` javascript
function _toString (val: any): string {
  return val == null
    ? ''
    : typeof val === 'object'
      ? JSON.stringify(val, null, 2)
      : String(val)
}
```

## [fn] toNumber

将字符串转化为数字并返回，如果转化后的逻辑值为假则返回原字符串。

``` javascript
function toNumber (val: string): number | string {
  const n = parseFloat(val, 10)
  return (n || n === 0) ? n : val
}
```

## [fn] makeMap

将以逗号分隔的字符串转化成 map。并返回一个函数，这个函数接收一个字符串，判断这个字符串是否在这个 map 中。具体可以参考下面这个函数的用法

``` javascript
function makeMap (
  str: string,
  expectsLowerCase?: boolean
): (key: string) => true | void {
  // 创建一个继承自 null 的 map
  const map = Object.create(null)
  const list: Array<string> = str.split(',')
  for (let i = 0; i < list.length; i++) {
    map[list[i]] = true
  }
  return expectsLowerCase
    ? val => map[val.toLowerCase()]
    : val => map[val]
}

```

## [fn] isBuiltInTag

查询是否为嵌套标签（`<slot></slot>`，`<component></component>`）

``` javascript
const isBuiltInTag = makeMap('slot,component', true)
```

eg:

``` javascript
isBuiltInTag('SLOT') // => true
isBuiltInTag('a') // => false
```

## [fn] remove

从数组（第一个参数）中删除一项（第二个参数）。

``` javascript
function remove (arr: Array<any>, item: any): Array<any> | void {
  if (arr.length) {
    const index = arr.indexOf(item)
    if (index > -1) {
      return arr.splice(index, 1)
    }
  }
}
```

## [fn] hasOwn
## [fn] hasOwnProperty

查看一个对象（第一个参数）是否有某种属性（第二个参数）

``` javascript
const hasOwnProperty = Object.prototype.hasOwnProperty
function hasOwn (obj: Object, key: string): boolean {
  return hasOwnProperty.call(obj, key)
}
```

## [fn] isPrimitive

判断是否为原始类型

``` javascript
function isPrimitive (value: any): boolean {
  return typeof value === 'string' || typeof value === 'number'
}
```

## [fn] cached

使用闭包在内存中缓存一个纯函数，重复使用时速度更快。

``` javascript
function cached (fn: Function): Function {
  const cache = Object.create(null)
  return function cachedFn (str: string): any {
    const hit = cache[str]
    return hit || (cache[str] = fn(str))
  }
}
```

## [fn] camelize

将连字符形式的字符串转化为驼峰命名形式的字符串

``` javascript
const camelizeRE = /-(\w)/g
const camelize = cached((str: string): string => {
  return str.replace(camelizeRE, (_, c) => c ? c.toUpperCase() : '')
})
```

## [fn] capitalize

返回一个首字母大写的字符串

``` javascript
const capitalize = cached((str: string): string => {
  return str.charAt(0).toUpperCase() + str.slice(1)
})
```

## [fn] hyphenate

将以驼峰形式命名的字符串转化为连字符形式

``` javascript
const hyphenateRE = /([^-])([A-Z])/g
const hyphenate = cached((str: string): string => {
  return str
    .replace(hyphenateRE, '$1-$2')
    .replace(hyphenateRE, '$1-$2')
    .toLowerCase()
})
```

## [fn] bind

☆

简单的绑定，比原生的要快（待解）

``` javascript
function bind (fn: Function, ctx: Object): Function {
  function boundFn (a) {
    const l: number = arguments.length
    return l
      ? l > 1
        ? fn.apply(ctx, arguments)
        : fn.call(ctx, a)
      : fn.call(ctx)
  }
  // record original fn length
  boundFn._length = fn.length
  return boundFn
}
```

## [fn] toArray

将类似数组对象转化为真实的数组，可以指定转化的起始位置

``` javascript
function toArray (list: any, start?: number): Array<any> {
  start = start || 0
  let i = list.length - start
  const ret: Array<any> = new Array(i)
  while (i--) {
    ret[i] = list[i + start]
  }
  return ret
}
```

## [fn] extend

将某个对象上属性扩展到到目标对象上

``` javascript
function extend (to: Object, _from: ?Object): Object {
  for (const key in _from) {
    to[key] = _from[key]
  }
  return to
}
```

## [fn] isObject

☆

用于快速检查类型是否为 'object'

``` javascript
/**
 * Quick object check - this is primarily used to tell
 * Objects from primitive values when we know the value
 * is a JSON-compliant type.
 */
function isObject (obj: any): boolean {
  return obj !== null && typeof obj === 'object'
}
```

## [fn] isPlainObject

检查是否是纯 js 对象，即含有0个及以上键值对的对象，`toString.call(null) => [object, Null]`

``` javascript
const toString = Object.prototype.toString
const OBJECT_STRING = '[object Object]'
function isPlainObject (obj: any): boolean {
  return toString.call(obj) === OBJECT_STRING
}
```

## [fn] toObject

将数组中所有对象合并到单独的一个对象上

``` javascript
function toObject (arr: Array<any>): Object {
  const res = arr[0] || {}
  for (let i = 1; i < arr.length; i++) {
    if (arr[i]) {
      extend(res, arr[i])
    }
  }
  return res
}
```

## [fn] noop

不执行任何操作

``` javascript
function noop () {}
```

## [fn] no

总返回 false

``` javascript
export const no = () => false
```

## ☆ [fn] genStaticKeys

从编译器模块中生成一个静态的字符串密钥

``` javascript
function genStaticKeys (modules: Array<ModuleOptions>): string {
  return modules.reduce((keys, m) => {
    return keys.concat(m.staticKeys || [])
  }, []).join(',')
}
```
