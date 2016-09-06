# Vue/core/util/props.js


## [fn] validateProp

验证 props 中的属性的值，并返回一个经过校验的合理的值。

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
  // 缺席，属性未赋值
  const absent = !hasOwn(propsData, key)
  let value = propsData[key]
  // 处理布尔属性的 prop
  if (getType(prop.type) === 'Boolean') {
    if (absent && !hasOwn(prop, 'default')) {
      // 属性未赋值 && 没有默认值
      value = false
    } else if (value === '' || value === hyphenate(key)) {
      // 赋值了 || 有默认值 && （值为空 || 值与属性转化为连字符形式的字符串相等）
      value = true
    }
  }
  // 检验默认值
  if (value === undefined) {
    // 没有默认值 🔽🔽🔽
    value = getPropDefaultValue(vm, prop, key)
    // 由于默认值是新的拷贝，需要添加观察
    const prevShouldConvert = observerState.shouldConvert
    // 需要转化
    observerState.shouldConvert = true
    observe(value)
    // 设置回原来的值
    observerState.shouldConvert = prevShouldConvert
  }
  if (process.env.NODE_ENV !== 'production') {
    // 判断属性值是否合法 🔽🔽🔽
    assertProp(prop, key, value, vm, absent)
  }
  return value
}
```

- [hasOwn](../../shared/util.md#fn-hasown)
- [hyphenate](../../shared/util.md#fn-hyphenate)
- [observe](../observer/index.md#fn-observe)
- [observerState](../observer/index.md#object-observerstate)

_[fn] getPropDefaultValue_

为 prop 获取一个默认值，没有指定 default 则返回 undefined，非函数的值调用工厂函数生成默认值，其他则返回值本身。

``` javascript
function getPropDefaultValue (vm: ?Component, prop: PropOptions, name: string): any {

  // 没有 default 属性，则返回 undefined
  // props: { 'aaa' }
  if (!hasOwn(prop, 'default')) return undefined

  // or props: { aaa: { default: String } }

  const def = prop.default
  // 作为工厂函数的 default 不能是 Object || Array 类型
  if (isObject(def)) {
    process.env.NODE_ENV !== 'production' && warn(
      'Invalid default value for prop "' + name + '": ' +
      'Props with type Object/Array must use a factory function ' +
      'to return the default value.',
      vm
    )
  }
  // 调用不是函数类型的工厂函数
  return typeof def === 'function' && prop.type !== Function
    ? def.call(vm)
    : def
}

```

- [warn](../util/debug.md#fn-warn)
- [hasOwn](../../shared/util.md#fn-hasown)
- [isObject](../../shared/util.md#fn-isobject)

_[fn] assertProp_

判断一个 prop 是否合法

``` javascript
function assertProp (
  prop: PropOptions,
  name: string,
  value: any,
  vm: ?Component,
  absent: boolean
) {
  // 缺少必填属性
  if (prop.required && absent) {
    warn('Missing required prop: "' + name + '"', vm)
    return
  }

  // 非必填项值为逻辑假，不报错
  if (value == null && !prop.required) return


  let type = prop.type
  let valid = !type
  const expectedTypes = []

  // 指定了类型
  if (type) {
    if (!Array.isArray(type)) {
      // 非数组转化为数组，@多类型参数实现
      type = [type]
    }
    // 满足数组中某一种类型即为合法
    for (let i = 0; i < type.length && !valid; i++) {
      const assertedType = assertType(value, type[i])
      expectedTypes.push(assertedType.expectedType)
      valid = assertedType.valid
    }
  }
  if (!valid) {
    // 指定了类型 && 不合法
    warn(
      'Invalid prop: type check failed for prop "' + name + '".' +
      ' Expected ' + expectedTypes.map(capitalize).join(', ') +
      ', got ' + Object.prototype.toString.call(value).slice(8, -1) + '.',
      vm
    )
    return
  }
  // 使用自定义验证器验证
  const validator = prop.validator
  if (validator) {
    if (!validator(value)) {
      warn(
        'Invalid prop: custom validator check failed for prop "' + name + '".',
        vm
      )
    }
  }
}
```

- [warn](../util/debug.md#fn-warn)
- [capitalize](../../shared/util.md#fn-capitalize)


_[fn] assertType_

判断参数类型

``` javascript
function assertType (value: any, type: Function): {
  valid: boolean,
  expectedType: string
} {
  let valid
  let expectedType = getType(type) // 🔽🔽🔽
  if (expectedType === 'String') {
    valid = typeof value === (expectedType = 'string')
  } else if (expectedType === 'Number') {
    valid = typeof value === (expectedType = 'number')
  } else if (expectedType === 'Boolean') {
    valid = typeof value === (expectedType = 'boolean')
  } else if (expectedType === 'Function') {
    valid = typeof value === (expectedType = 'function')
  } else if (expectedType === 'Object') {
    valid = isPlainObject(value)
  } else if (expectedType === 'Array') {
    valid = Array.isArray(value)
  } else {
    valid = value instanceof type
  }
  return { valid, expectedType }
}

assertType(1234, String) // => { valid: false, expectedType: 'string' }
assertType(1234, Number) // => { valid: true, expectedType: 'number' }
assertType(1234, Array)  // => { valid: true, expectedType: 'Array' }
```

- [isPlainObject](../../shared/util.md#fn-isplainobject)


_[fn] getType_

用构造函数检查类型做类型检查以防在不同的 vms / iframes 下检查失败

``` javascript
function getType (fn) {
  const match = fn && fn.toString().match(/^\s*function (\w+)/)
  return match && match[1]
}

// eg:
getType(String) // => 'String'
```
