# Vue/core/observer

| name | content |
|:---:|:---|
|`index.js`|Observer 类|
|`dep.js`|Dep 类, 监视变动的依赖项|
|`watcher.js`|Watcher 类|
|`scheduler.js`||
|`array.js`|实现一个能通知变化的新的 Array|

# index.js

## [object] observerState

默认情况下，当设置一个响应值时，这个值就会被转化成可响应的。但是当向下传递 props 的时候，由于它可能被嵌套在一个不可变的数据结构中，因此我们可能不想转化它。因为转化它会影响性能。

``` javascript
const observerState = {
  shouldConvert: true,
  isSettingProps: false
}
```

## [class] Observer
Observer 类被绑定到每一个观测对象上，只要绑定一次，实例就会将目标对象的属性都转化为 getter/setter 形式，这样就可以收集依赖，并派发更新了。[`Object.getOwnPropertyNames`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyNames)返回一个由指定对象的所有自身属性的属性名（包括不可枚举属性）组成的数组。
``` javascript

const arrayKeys = Object.getOwnPropertyNames(arrayMethods)

class Observer {
  constructor (value: any) {
    this.value = value
    this.dep = new Dep()
    this.vmCount = 0 // 有几处 vms 将这个对象作为根 $data

    // 将 Observer 的实例赋值给被观测对象的 __ob__ 属性
    def(value, '__ob__', this)

    if (Array.isArray(value)) {
      // core/util/env.js 能否是用 __proto__
      // ./array.js arrayMethods
      const augment = hasProto
        ? protoAugment // 🔽
        : copyAugment // 🔽
      augment(value, arrayMethods, arrayKeys)
      this.observeArray(value)
    } else {
      this.walk(value)
    }
  }

  // 将每一个属性转化为 getter/setter 形式
  // 这个方法的参数类型只能为 Object
  walk (obj: Object) {
    const val = this.value
    for (const key in obj) {
      // 定义可响应的属性值
      defineReactive(val, key, obj[key])
    }
  }

  // 观测数组中的所有项
  observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }
}

// 使用 __proto__ 拦截原型链来达到增强目标对象/数组的目的（arrayMethods）
function protoAugment (target, src: Object) {
  target.__proto__ = src
}

// 定义隐藏的属性值来达到增强目标对象/数组的目的（arrayMethods）
function copyAugment (target: Object, src: Object, keys: Array<string>) {
  for (let i = 0, l = keys.length; i < l; i++) {
    const key = keys[i]
    def(target, key, src[key])
  }
}
```

## [fn] defineReactive

在对象上定义一个可响应的属性， 函数 [`Object.getOwnPropertyDescriptor`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor) 返回一个描述当前属性（obj[key]）特性的对象

``` javascript
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: Function
) {
  const dep = new Dep()
  const property = Object.getOwnPropertyDescriptor(obj, key)
  // 属性可以配置（改变或删除）
  if (property && property.configurable === false) {
    return
  }

  // 之前定义过的 getter/setter
  const getter = property && property.get
  const setter = property && property.set

  let childOb = observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true, // 可枚举
    configurable: true, // 可改变可删除
    get: function reactiveGetter () {
      // getter 返回值
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        // 为当前唯一 watcher（Dep.target） 添加观察依赖项
        dep.depend()
        if (childOb) {
          // 为当前唯一 watcher（Dep.target） 添加观察依赖项
          childOb.dep.depend()
        }
        if (Array.isArray(value)) {
          for (let e, i = 0, l = value.length; i < l; i++) {
            e = value[i]
            // 为当前唯一 watcher（Dep.target） 添加观察依赖项
            e && e.__ob__ && e.__ob__.dep.depend()
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val

      if (newVal === value) {
        // 新值与旧值相等
        return
      }
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        // 非生产环境下自定义执行
        customSetter()
      }
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      // 观测新的值
      childOb = observe(newVal)
      // 通知属性值已经改变
      dep.notify()
    }
  })
}
```

## [fn] observe
尝试为 value 创建一个 observer 实例，如果监听成功返回这个新实例。如果 value 已经被观测，则返回已有的 observer 实例
``` javascript
function observe (value: any): Observer | void {
  if (!isObject(value)) {
    return
  }
  let ob: Observer | void
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
    // 如果已经有 observer
    ob = value.__ob__
  } else if (
    observerState.shouldConvert &&
    !config._isServer &&
    (Array.isArray(value) || isPlainObject(value)) &&
    Object.isExtensible(value) &&
    !value._isVue // 不是 Vue 系统值
  ) {
    ob = new Observer(value)
  }
  return ob
}
```

## [fn] set

设置对象的属性值，如果这个属性值之前不存在则添加这个新属性并触发变化通知。

``` javascript
function set (obj: Array<any> | Object, key: any, val: any) {
  if (Array.isArray(obj)) {
    obj.splice(key, 1, val)
    return val
  }
  if (hasOwn(obj, key)) {
    obj[key] = val
    return
  }
  // 是新属性
  const ob = obj.__ob__

  // 不能为 Vue 系统对象或者其根 $data 添加属性
  if (obj._isVue || (ob && ob.vmCount)) {
    process.env.NODE_ENV !== 'production' && warn(
      'Avoid adding reactive properties to a Vue instance or its root $data ' +
      'at runtime - delcare it upfront in the data option.'
    )
    return
  }
  // 为没被监控的对象添加属性
  if (!ob) {
    obj[key] = val
    return
  }
  // 为监控的对象添加可响应的属性值
  defineReactive(ob.value, key, val)
  // 派发通知
  ob.dep.notify()
  return val
}
```

## [fn] del

删除一个属性

``` javascript
function del (obj: Object, key: string) {
  const ob = obj.__ob__
  if (obj._isVue || (ob && ob.vmCount)) {
    process.env.NODE_ENV !== 'production' && warn(
      'Avoid deleting properties on a Vue instance or its root $data ' +
      '- just set it to null.'
    )
    return
  }
  if (!hasOwn(obj, key)) {
    return
  }
  delete obj[key]
  // 如果没有被观测则不通知删除
  if (!ob) {
    return
  }
  ob.dep.notify()
}

```
