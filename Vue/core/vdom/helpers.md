# Vue/core/vdom/helpers.js

## [fn] normalizeChildren

``` javascript
function normalizeChildren (
  children: any,
  ns: string | void
): Array<VNode> | void {
  if (isPrimitive(children)) {
    return [createTextVNode(children)]
  }
  if (Array.isArray(children)) {
    const res = []
    for (let i = 0, l = children.length; i < l; i++) {
      const c = children[i]
      const last = res[res.length - 1]
      //  nested
      if (Array.isArray(c)) {
        res.push.apply(res, normalizeChildren(c, ns))
      } else if (isPrimitive(c)) {
        if (last && last.text) {
          last.text += String(c)
        } else if (c !== '') {
          // convert primitive to vnode
          res.push(createTextVNode(c))
        }
      } else if (c instanceof VNode) {
        if (c.text && last && last.text) {
          last.text += c.text
        } else {
          // inherit parent namespace
          if (ns) {
            applyNS(c, ns)
          }
          res.push(c)
        }
      }
    }
    return res
  }
}
```

## [fn] getFirstComponentChild

``` javascript
function getFirstComponentChild (children: ?Array<any>) {
  return children && children.filter(c => c && c.componentOptions)[0]
}
```

## [fn] mergeVNodeHook

``` javascript
function mergeVNodeHook (def: Object, key: string, hook: Function) {
  const oldHook = def[key]
  if (oldHook) {
    def[key] = function () {
      oldHook.apply(this, arguments)
      hook.apply(this, arguments)
    }
  } else {
    def[key] = hook
  }
}
```

## [fn] updateListeners

``` javascript
function updateListeners (
  on: Object,
  oldOn: Object,
  add: Function,
  remove: Function
) {
  let name, cur, old, fn, event, capture
  for (name in on) {
    cur = on[name]
    old = oldOn[name]
    if (!cur) {
      process.env.NODE_ENV !== 'production' && warn(
        `Handler for event "${name}" is undefined.`
      )
    } else if (!old) {
      capture = name.charAt(0) === '!'
      event = capture ? name.slice(1) : name
      if (Array.isArray(cur)) {
        add(event, (cur.invoker = arrInvoker(cur)), capture)
      } else {
        fn = cur
        cur = on[name] = {}
        cur.fn = fn
        add(event, (cur.invoker = fnInvoker(cur)), capture)
      }
    } else if (Array.isArray(old)) {
      old.length = cur.length
      for (let i = 0; i < old.length; i++) old[i] = cur[i]
      on[name] = old
    } else {
      old.fn = cur
      on[name] = old
    }
  }
  for (name in oldOn) {
    if (!on[name]) {
      event = name.charAt(0) === '!' ? name.slice(1) : name
      remove(event, oldOn[name].invoker)
    }
  }
}
```
