# Vue/core/observer/watcher.js

## [class] Watcher

-  `core/instance/state.js` Vue.prototype.$watch 由此实现

_构造函数_

``` javascript
class Watcher {
  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: Object = {}
  ) {
    this.vm = vm
    vm._watchers.push(this)
    // options
    this.deep = !!options.deep // boolean 深度监听
    this.user = !!options.user // boolean 用户使用
    this.lazy = !!options.lazy // boolean
    this.sync = !!options.sync // boolean 同步更新
    this.expression = expOrFn.toString() // string
    this.cb = cb    // Function
    this.id = ++uid // number 用于计数
    this.active = true  // boolean
    this.dirty = this.lazy // boolean 脏检查
    this.deps = [] // Array<Dep>
    this.newDeps = [] // Array<Dep>
    this.depIds = new Set() //Set
    this.newDepIds = new Set() // Set

    // 将 expOrFn 转为 getter
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn
    } else {
      // /core/util/lang.js [fn] parsePath
      // 返回一个函数，用于获取传入对象的 expOrFn 路径
      this.getter = parsePath(expOrFn)
      if (!this.getter) {
        this.getter = function () {}
        process.env.NODE_ENV !== 'production' && warn(
          `Failed watching path: "${expOrFn}" ` +
          'Watcher only accepts simple dot-delimited paths. ' +
          'For full control, use a function instead.',
          vm
        )
      }
    }
    this.value = this.lazy
      ? undefined
      : this.get()
  }
  // 实例方法 ...
}
```

_实例方法 get_

执行 getter 并重新依赖

``` javascript
get () {
  // core/observer/dep.js
  pushTarget(this)
  const value = this.getter.call(this.vm, this.vm)
  // "touch" every property so they are all tracked as
  // dependencies for deep watching
  if (this.deep) {
    traverse(value)
  }
  // core/observer/dep.js
  popTarget()
  this.cleanupDeps()
  return value
}
```

_实例方法 addDep_

为指令添加一个依赖项

``` javascript
addDep (dep: Dep) {
  const id = dep.id
  if (!this.newDepIds.has(id)) {
    this.newDepIds.add(id)
    this.newDeps.push(dep)
    if (!this.depIds.has(id)) {
      dep.addSub(this)
    }
  }
}
```

_实例方法 cleanupDeps_

清空依赖的集合，用新的依赖替换旧的依赖

``` javascript
cleanupDeps () {
  let i = this.deps.length
  while (i--) {
    const dep = this.deps[i]
    if (!this.newDepIds.has(dep.id)) {
      dep.removeSub(this)
    }
  }
  let tmp = this.depIds
  this.depIds = this.newDepIds
  this.newDepIds = tmp // 清除引用
  this.newDepIds.clear()
  tmp = this.deps
  this.deps = this.newDeps
  this.newDeps = tmp // 清除引用
  this.newDeps.length = 0
}
```

_实例方法 update_

调度程序接口，当依赖项发生变化时调用

``` javascript
update () {
  if (this.lazy) {
    this.dirty = true
  } else if (this.sync) {
    this.run()
  } else {
    // ./scheduler.js
    queueWatcher(this)
  }
}
```

_实例方法 run_

调度程序处理接口，由事务调用

``` javascript
run () {
  if (this.active) {
    const value = this.get()
    if (
      value !== this.value ||
      // Deep watchers and watchers on Object/Arrays should fire even
      // when the value is the same, because the value may
      // have mutated.
      isObject(value) ||
      this.deep
    ) {
      // set new value
      const oldValue = this.value
      this.value = value
      if (this.user) {
        try {
          this.cb.call(this.vm, value, oldValue)
        } catch (e) {
          process.env.NODE_ENV !== 'production' && warn(
            `Error in watcher "${this.expression}"`,
            this.vm
          )
          /* istanbul ignore else */
          if (config.errorHandler) {
            config.errorHandler.call(null, e, this.vm)
          } else {
            throw e
          }
        }
      } else {
        this.cb.call(this.vm, value, oldValue)
      }
    }
  }
}
```

_实例方法 evaluate_

计算实例的值，只会被惰性 watcher 调用

``` javascript
evaluate () {
  this.value = this.get()
  this.dirty = false
}
```

_实例方法 depend_

用实例依赖所有的依赖集合中的项目

``` javascript
depend () {
  let i = this.deps.length
  while (i--) {
    this.deps[i].depend()
  }
}
```

_实例方法 teardown_

将实例从所有的依赖的调度列表中删除

``` javascript
teardown () {
  if (this.active) {
    // remove self from vm's watcher list
    // this is a somewhat expensive operation so we skip it
    // if the vm is being destroyed or is performing a v-for
    // re-render (the watcher list is then filtered by v-for).
    if (!this.vm._isBeingDestroyed && !this.vm._vForRemoving) {
      remove(this.vm._watchers, this)
    }
    let i = this.deps.length
    while (i--) {
      this.deps[i].removeSub(this)
    }
    this.active = false
  }
}
```
