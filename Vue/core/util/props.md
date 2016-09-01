# Vue/core/util/props.js

## ☆ [fn] validateProp

验证 props 中的属性，
（待解）

``` javascript
function validateProp (
  key: string,
  propOptions: Object,
  propsData: ?Object,
  vm?: Component
): any {
  /* istanbul ignore if */
  if (!propsData) return
  const prop = propOptions[key]
  const absent = !hasOwn(propsData, key)
  let value = propsData[key]
  // handle boolean props
  if (getType(prop.type) === 'Boolean') {
    if (absent && !hasOwn(prop, 'default')) {
      value = false
    } else if (value === '' || value === hyphenate(key)) {
      value = true
    }
  }
  // check default value
  if (value === undefined) {
    value = getPropDefaultValue(vm, prop, key)
    // since the default value is a fresh copy,
    // make sure to observe it.
    const prevShouldConvert = observerState.shouldConvert
    observerState.shouldConvert = true
    observe(value)
    observerState.shouldConvert = prevShouldConvert
  }
  if (process.env.NODE_ENV !== 'production') {
    assertProp(prop, key, value, vm, absent)
  }
  return value
}
```

``` javascript
// 用构造函数检查类型做类型检查以防在不同的 vms / iframes 下检查失败
function getType (fn) {
  const match = fn && fn.toString().match(/^\s*function (\w+)/)
  return match && match[1]
}

// eg:
getType(String) // => 'String'
```

``` javascript
// assertType 函数用于检测类型
assertType(1234, String) // => { valid: false, expectedType: 'string' }
assertType(1234, Number) // => { valid: true, expectedType: 'number' }
assertType(1234, Array)  // => { valid: true, expectedType: 'Array' }
```
