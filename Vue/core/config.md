# Vue/core/config.js

``` javascript
const config: Config = {
  // user

  /**
   * 选项合并策略，（用于core/util/options）
   * @type { [key: string]: Function }
   */
  optionMergeStrategies: Object.create(null),

  /**
   * 是否屏蔽警告
   * @type boolean
   */
  silent: false,

  /**
   * 是否启动开发工具
   * @type boolean
   */
  devtools: process.env.NODE_ENV !== 'production',

  /**
   * watcher 错误处理器
   * @type ?Function
   */
  errorHandler: null,

  /**
   * 忽略的自定义元素
   * @type ?Array<string>
   */
  ignoredElements: null,

  /**
   * 为绑定（v-on）键设置别名
   * @type { [key: string]: number }
   */
  keyCodes: Object.create(null),

  // platform
  /**
   * 检查一个标签是否已经被转换了
   * 被转换的标签不能注册成一个组件 - 依赖平台并有可能被覆盖
   * @type (x?: string) => boolean
   */
  isReservedTag: no,

  /**
   * 检查一个标签是否为位置元素 - 依赖平台
   * @type (x?: string) => boolean
   */
  isUnknownElement: no,

  /**
   * 获取一个元素的命名空间
   * @type (x?: string) => string | void
   */
  getTagNamespace: noop,

  /**
   * 检查一个 attribute 是否必须使用属性绑定 e.g. value - 依赖平台
   * @type (x?: string) => boolean
   */
  mustUseProp: no,

  // internal

  /**
   * 组件可以拥有的资源类型
   * @type Array<string>
   */
  _assetTypes: [
    'component',
    'directive',
    'filter'
  ],

  /**
   * 生命周期钩子函数名列表
   * @type Array<string>
   */
  _lifecycleHooks: [
    'beforeCreate',
    'created',
    'beforeMount',
    'mounted',
    'beforeUpdate',
    'updated',
    'beforeDestroy',
    'destroyed',
    'activated',
    'deactivated'
  ],

  /**
   * 在一次 scheduler 刷新循环中允许的最大循环更新数量
   * @type number
   */
  _maxUpdateCount: 100,

  /**
   * 在一次 scheduler 刷新循环中允许的最大循环更新数量
   * @type boolean
   */
  _isServer: process.env.VUE_ENV === 'server'
}

export default config

```

- [no, noop](../shared/util.md#fn-no)
