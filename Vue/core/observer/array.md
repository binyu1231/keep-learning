# Vue/core/observer/array.js

## [object] arrayMethods

实现一个能通知变化的新的 Array

``` javascript
const arrayProto = Array.prototype
export const arrayMethods = Object.create(arrayProto)

// 拦截能改变数组的方法，并派发事件
;[ 'push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse']
.forEach(function (method) {

  // 保存原来的方法 eg: Array.prototype.pop
  const original = arrayProto[method]

  // /core/util/lang.js [obj, key, val]
  def(arrayMethods, method, function mutator () {

    // 避免遗漏参数: http://jsperf.com/closure-with-arguments
    let i = arguments.length
    const args = new Array(i)
    while (i--) {
      args[i] = arguments[i]
    }

    const result = original.apply(this, args)

    // core/observer/index.js
    const ob = this.__ob__
    let inserted
    switch (method) {
      case 'push':
        inserted = args
        break
      case 'unshift':
        inserted = args
        break
      case 'splice':
        inserted = args.slice(2)
        break
    }
    // 如果有新插入的元素[数组]，对其添加监听
    if (inserted) ob.observeArray(inserted)

    // 通知变化
    ob.dep.notify()
    return result
  })
})

```
