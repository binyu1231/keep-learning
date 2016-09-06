闲下来学习一下 Vue 源码。自省用不保证对。

目录中

- ★ 代表已经完整
- ☆ 代表完成一部分
- 没有星代表还没看

**注意**：这里只是将源码拆分成细碎部分进行学习，一些细节不一定能很好的展示出来（例如变量的作用域），
建议对照源码。版本：2.0.0-rc3，`npm i vue@next`。

文件中导出的变量会用标题（加粗）写出来，并标明类型。内部函数没用加粗字体表示

由于代码比较多，大家可以根据下面的目录结构在其他文章中找到对应的代码。

建议阅读顺序：

``` diff
+ shared/
    |- util.js                            // 共用的一些工具函数
+ sfc/
    |- parser.js                          // 解析 .vue 文件
+ core/
+   |- util/
    |   |- debug.js                       // warn formatComponentName 两个调试函数
    |   |- env.js                         // 检测环境，实现兼容的 nextTick 和 _Set
    |   |- lang.js                        // 作用于变量的工具函数
    |   |- options.js                     // 对配置项进行操作
    |   |- props.js                       // 验证组件的属性
    |   |- index.js                       // 导出
+   |- global-api/
    |   |- assets.js                      // Vue.component/directive/filter
    |   |- extend.js                      // Vue.extend
    |   |- mixin.js                       // Vue.mixin
    |   |- use.js                         // Vue.use
    |   |- index.js                       // 初始化全局 api
+   |- instance/
    |   |- events.js                      // $on $off $once $emit
    |   |- init.js                        // Vue.prototype._init 初始化
    |   |- lifecycle.js                   // 实现生命周期
    |   |- proxy.js
    |   |- render.js                      // render 选项
    |   |- state.js                       // props/data/computed/methods/watch
    |   |- index.js                       // 导出 Vue 将各文件实例方法合并
+   |- observer/
    |   |- array.js                       // 实现一个能通知变化的新的 Array
    |   |- dep.js                         // Dep 类, 监视变动的依赖项
    |   |- watcher.js                     // Watcher 类
    |   |- scheduler.js                   
    |   |- index.js                       // Observer 类
+   |- vdom/
+   |   |- modules/
    |   |   |- directives.js
    |   |   |- ref.js
    |   |   |- index.js
    |   |- create-component.js
    |   |- create-element.js
    |   |- helpers.js
    |   |- patch.js
    |   |- vnode.js
    |- index.js
+   |- components/
    |   |- keep-alive.js                  // 指令 keep-alive 的实现
    |   |- index.js                       // 导出   
+ compiler/
+   |- codegen/
    |   |- events.js
    |   |- index.js
+   |- directives/
    |   |- bind.js
    |   |- index.js
+   |- parser/
    |   |- entity-decoder.js
    |   |- filter-parser.js
    |   |- html-parser.js
    |   |- text-parser.js
    |   |- index.js
    |- error-detector.js
    |- helpers.js
    |- optimizer.js
    |- optimizer.js

+ entries/
    |- web-compiler.js
    |- web-runtime-with-compiler.js
    |- web-runtime.js
    |- web-server-renderer.js
+ server/
    |- create-bundle-renderer.js
    |- create-renderer.js
    |- render-stream.js
    |- render.js
    |- run-in-vm.js
    |- write.js
+ platforms/
+   |- web/
+   |   |- compiler/
+   |   |   |- directives/
    |   |   |   |- html.js
    |   |   |   |- model.js
    |   |   |   |- text.js
    |   |   |   |- index.js
+   |   |   |- modules/
    |   |   |   |- class.js
    |   |   |   |- style.js
    |   |   |   |- index.js
    |   |   |- index.js
+   |   |- runtime/
+   |   |   |- directives/
    |   |   |   |- model.js
    |   |   |   |- show.js
    |   |   |   |- index.js
+   |   |   |- modules/
    |   |   |   |- attrs.js
    |   |   |   |- class.js
    |   |   |   |- dom-props.js
    |   |   |   |- events.js
    |   |   |   |- style.js
    |   |   |   |- transition.js
    |   |   |   |- index.js
+   |   |   |- components/
    |   |   |   |- transition-group.js
    |   |   |   |- transition.js
    |   |   |   |- index.js
    |   |   |- class-util.js
    |   |   |- node-ops.js
    |   |   |- path.js
    |   |   |- transition-util.js
+   |   |- server/
+   |   |   |- directives/
    |   |   |   |- show.js
    |   |   |   |- index.js
+   |   |   |- modules/
    |   |   |   |- attrs.js
    |   |   |   |- class.js
    |   |   |   |- dom-props.js
    |   |   |   |- style.js
    |   |   |   |- index.js
+   |   |- util/
    |   |   |- attrs.js
    |   |   |- class.js
    |   |   |- element.js
    |   |   |- index.js
```
