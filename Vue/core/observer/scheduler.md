# Vue/core/observer/scheduler.js

## [fn] queueWatcher

将一个 watcher 推进监视队列，拥有重复 ID 会被跳过。除非当队列正在刷新时，它被推了进去。

``` javascript
const queue: Array<Watcher> = []
let has: { [key: number]: ?true } = {}
let circular: { [key: number]: number } = {}
let waiting = false
let flushing = false
let index = 0

function queueWatcher (watcher: Watcher) {
  const id = watcher.id
  if (has[id] == null) {
    has[id] = true
    if (!flushing) {
      queue.push(watcher)
    } else {
      // 如果已经在刷新中，根据 watcher 的 id 大小顺序插入到数组中
      // 如果已经执行的 id 超过了它的 id，将会在下次执行
      let i = queue.length - 1
      while (i >= 0 && queue[i].id > watcher.id) {
        i--
      }
      queue.splice(Math.max(i, index) + 1, 0, watcher)
    }
    // 队列刷新
    if (!waiting) {
      waiting = true
      // 异步执行函数 // /core/util/env.js
      nextTick(flushSchedulerQueue) // ⬇️⬇️⬇️
    }
  }
}
```

_[fn] flushSchedulerQueue_

更新所有队列，并运行 watcher

``` javascript
function flushSchedulerQueue () {
  flushing = true
  // 在刷新前对队列进行排序，这是为了确保：
  // 1. 组件由父级更新到子级（因为父级总是先于子级创建）
  // 2. 一个组件的用户 watcher 要先于渲染 watcher
  // 执行（也是因为前者总是先被创建）
  // 3. 如果一个组件在它父组件 watcher 运行时被销毁
  // 那他的所有 watcher 可以被跳过
  queue.sort((a, b) => a.id - b.id)

  // 这里没有存储长度，因为在执行过程中会有更多的 watcher
  // 会被添加进来
  for (index = 0; index < queue.length; index++) {
    const watcher = queue[index]
    const id = watcher.id
    has[id] = null
    watcher.run()
    // 在开发环境中，检查并阻止无限更新
    if (process.env.NODE_ENV !== 'production' && has[id] != null) {
      circular[id] = (circular[id] || 0) + 1
      if (circular[id] > config._maxUpdateCount) {
        warn(
          'You may have an infinite update loop ' + (
            watcher.user
              ? `in watcher with expression "${watcher.expression}"`
              : `in a component render function.`
          ),
          watcher.vm
        )
        break
      }
    }
  }

  // devtool hook
  /* istanbul ignore if */
  if (devtools && config.devtools) {
    devtools.emit('flush')
  }

  resetSchedulerState() // 🔽🔽🔽
}
```

_[fn] resetSchedulerState_

重置任务状态

``` javascript
function resetSchedulerState () {
  queue.length = 0
  has = {}
  if (process.env.NODE_ENV !== 'production') {
    circular = {}
  }
  waiting = flushing = false
}
```
