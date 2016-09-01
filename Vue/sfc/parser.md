# Vue/sfc/parser.js

只暴露了一个 `parseComponent` 函数，作用是将单文件组件（`.vue`）转化成一个 sfc 对象（可识别的组件对象）。最终的 vue.js 中不会包含这一部分。

## [fn] parseComponent

``` javascript
function parseComponent (
  content: string, // 文件内容
  options?: Object = {} // 配置项
 ): SFCDescriptor {

 const sfc: SFCDescriptor = {
   template: null,
   script: null,
   styles: []
 }

 // code

 return sfc
}

```

使用 `compiler/parser/html-parser` 暴露的 parseHTML 函数转化 content，option 中指定了每个标签开始（start）和结束（end）时要执行的函数。

``` javascript
// ...
  parseHTML(content, { start, end })
  return sfc
}

```
start 函数用来确定三个特殊标签 `<script>, <style>, <template>` 的开始，并将其内容信息存储到 sfc 上

``` javascript
function start (
    tag: string, // 标签名
    attrs: Array<Attribute>, // 属性的数组
    unary: boolean, // 是否为单标签
    start: number, // 标签名（<）开始位置
    end: number // 标签名（>）结束位置
  ) {

  // isSpecialTag: 函数用来检查当前标签 是否为 <script>, <style>, <template> 其中的一个
  // const isSpecialTag = makeMap('script,style,template', true)
  // depth: 代表标签嵌套的深度

  if (isSpecialTag(tag) && depth === 0) {
    currentBlock = {
      type: tag,
      content: '',
      start: end // 以标签名结束位置（/>）作为内容的起始位置
    }

    // checkAttrs 函数用来检测这三个标签上的特殊属性值，🔽🔽🔽下面有细节

    checkAttrs(currentBlock, attrs)

    // 将 <style> 中的值添加到样式数组中
    // <script> 和 <template> 内容则直接添加到 sfc 对象上
    // 由此可以看出 .vue 文件中可以有多个 <style> ，
    // 但 <script> 和 <template> 只能有一个

    if (tag === 'style') {
      sfc.styles.push(currentBlock)
    } else {
      sfc[tag] = currentBlock
    }
  }
  // 不是单标签，则嵌套深度加 1。
  // 在 end 函数中以 depth 是否为 0 来判断是否结束

  if (!unary) depth++
}
```

checkAttrs 接收 start 中的 currentBlock 和标签上的属性数组，如果有预期的值则存储到 currentBlock 上

``` javascript
function checkAttrs (block: SFCBlock, attrs: Array<Attribute>) {
  for (let i = 0; i < attrs.length; i++) {
    const attr = attrs[i]

    // lang: jade/sass/coffee..

    if (attr.name === 'lang') {
      block.lang = attr.value
    }

    // 是否指定 <style> 的样式只作用于当前组件

    if (attr.name === 'scoped') {
      block.scoped = true
    }

    // 外链的资源路径

    if (attr.name === 'src') {
      block.src = attr.value
    }
  }
}
```
接下来是 end 函数，每当一个标签结束时就会执行一次这个函数，而这个函数的功能是用来判断三种特殊标签是否结束的，并将标签内容存储到 sfc 对象上。

``` javascript
function end (
  tag: string, // 标签名
  start: number, // 标签名（<）开始位置
  end: number // 标签名（/>）结束位置
) {
  // 是否为三种特殊标签，当前嵌套的深度是否是1，是否已经初始化
  if (isSpecialTag(tag) && depth === 1 && currentBlock) {

    // 结束标签的开始位置是内容的结束位置
    currentBlock.end = start

    // 引用了第三方库来过滤代码的缩进 npm: de-indent
    let text = deindent(content.slice(currentBlock.start, currentBlock.end))

    // 是否为 <script>, <style> 的起始未知添加空行，
    // 目的是为了在使用 eslint 等代码预处理器的时候能正确的报出行号

    if (currentBlock.type !== 'template' && options.pad) {
      text = padContent(currentBlock) + text
    }
    currentBlock.content = text
    currentBlock = null
  }
  // 减少一层嵌套深度
  // 这里估计单标签结束时应该不会执行 end 函数了
  depth--
}
```
最后看一下 padContent 为标签起始位置添加空行

``` javascript
// const splitRE = /\r?\n/g

function padContent (block: SFCBlock) {
  // 开始标签的结束位置（>）与代码之间有多少个换行
  const offset = content.slice(0, block.start).split(splitRE).length
  // 如果是js 代码并且没有特殊语言类型则添加注释换行，其他添加换行符
  const padChar = block.type === 'script' && !block.lang
    ? '//\n'
    : '\n'
  // 返回 offset 个数的空行
  return Array(offset).join(padChar)
}

```
