# Vue/core/observer/dep.js

## [class] Dep

Dep 实例是可观察的，可以有多个指令同时订阅它。目标 wather 被置于栈顶

``` javascript
let uid = 0

class Dep {
  constructor () {
    this.id = uid++
    this.subs = []  // 订阅实例的 Watcher 数组
  }
  // 添加订阅
  addSub (sub: Watcher) {
    this.subs.push(sub)
  }
  // 删除订阅
  removeSub (sub: Watcher) {
    // shard/util.js 数组中删除一项
    remove(this.subs, sub)
  }

  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }
  // 通知所有订阅者执行更新操作
  notify () {
    // ？先稳定订阅者列表
    const subs = this.subs.slice()
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}

// 当前目标（target） watcher 处于计算中
// 任何情况下只会有一个 watcher 处于计算状态
// 所以 Dep.target 是唯一的
Dep.target = null

// 目标栈
const targetStack = []

```
## [fn] pushTarget

目标 watcher 入栈

``` javascript
function pushTarget (_target: Watcher) {
  // 将之前的 watcher 推入栈中
  if (Dep.target) targetStack.push(Dep.target)
  // 切换目标 watcher
  Dep.target = _target
}
```

没用 targetStack 最后一个元素赋值，节省了一个空间

## [fn] popTarget

推出栈顶 watcher

``` javascript
function popTarget () {
  // 舍弃当前目标，用上一个 watcher 作为目标
  Dep.target = targetStack.pop()
}
```
