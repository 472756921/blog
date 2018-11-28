---
title: '''webpack读书笔记'''
date: 2018-11-12 17:47:17
tags:
---

## 为什么会有webpack
现代web开发中模块式是一个必备解决方案，因此诞生以一些列模块式工具，诸如AMD，CMD，commonJS，ES6模块等概念和工具。
> 前端模块要在客户端中执行，所以他们需要增量加载到浏览器中。
模块的加载和传输，我们首先能想到两种极端的方式，一种是每个模块文件都单独请求，另一种是把所有模块打包成一个文件然后只请求一次。显而易见，每个模块都发起单独的请求造成了请求次数过多，导致应用启动速度慢；一次请求加载所有模块导致流量浪费、初始化过程慢。这两种方式都不是好的解决方案，它们过于简单粗暴。
引自-《Webpack 中文指南》

> 分块传输，按需进行懒加载，在实际用到某些模块的时候再增量更新，才是较为合理的模块加载方案。要实现模块的按需加载，就需要一个对整个代码库中的模块进行静态分析、编译打包的过程。
引自-《Webpack 中文指南》

> 在编译的时候，要对整个代码进行静态分析，分析出各个模块的类型和它们依赖关系，然后将不同类型的模块提交给适配的加载器来处理。比如一个用 LESS 写的样式模块，可以先用LESS加载器将它转成一个CSS 模块，在通过 CSS 模块把他插入到页面的'style'标签中执行。Webpack 就是在这样的需求中应运而生。
引自-《Webpack 中文指南》

## 什么是 Webpack
Webpack 是一个模块打包器。它将根据模块的依赖关系进行静态分析，然后将这些模块按照指定的规则生成对应的静态资源。
包括：
1. 代码转换
2. 文件优化（压缩文件，合并图片）
3. 代码分割（提取公共代码，让首屏不需要的代码和资源异步加载）
4. 模块合并
5. 代码验证
6. 自动发布（代码更新后，自动构建发布）

## Loader
Webpack 本身只能处理 JavaScript 模块，如果要处理其他类型的文件，就需要使用 loader进行转换。
Loader 可以理解为是模块和资源的转换器，它本身是一个函数，接受源文件作为参数，返回转换的结果。这样，我们就可以通过 require 来加载任何类型的模块或文件，比如CoffeeScript、JSX、 LESS 或图片。
Loader的写法：
``` JS
module: {
    rules: [
        {test: /\.css$/, use:['style-loader', 'css-loader?minimize'] }
    ]
}
```
注意loader的执行顺序是从后向前，先由css-loader读取css文件，再由style-loader将其注入到JS文件中。
每个loader都可以传入参数（URL querysting 的方式），minimize 表示开启css压缩，能够减少css部分 10%-20% 的体积。

## Plugin
Plugin是用来扩展webpack功能的。在 Webpack 构建流程中的特定时机注入扩展逻辑，来改变构建结果或做我们想要的事情。
这是一个Plugin的例子，用于单独打包css
``` JS
module: {
    ...
},
plugins: [
    new MiniCssExtractPlugin({
        filename: `[name]_[contenthash:8].css`,
    })
]
```
## source-map
有了这个东西就可以映射源码。
在config中配置：devtool: 'source-map'（与entry同级） 即可

# webpack 详细
前面讲讲了一大堆，想必对webpack有了一个大致的了解，下面的部分将详细记录webpack的各种工作和功能。


## Entry
Entry是配置的模块的入口，搜寻及递归解析出所有入口依赖的模块（必填）。
Entry有三种配置方式
1. string （搭配 output.library 配置项使用时，只有数组里的最后一个入口文件的模块会被导出）
2. array
3. object（配置多个入口，每个入口生成一个Chunk（代码块））

entry 也可以配置成动态的方式
``` JS
entry : () => {
    return {
        a : ’. / p ages/a ’,
        b :’./pages/ b ’,
    }
}
```

### Chunk
如果entry是一个String/array，只会生成一个chunk，名字是mian
如果entry是一个Obj，就可能出现多个Chunk，名字是obj中的键值

## Output
filename: 输出文件名
如果只有一个chunk的话，filename 可使用静态string，如果是多个chunk的话则可用变量模板，例如 '[name].js'
这里的[name]是chunk内置的name变量去替换（除了name还有id，hash->hash:8，chunkhash->chunkhash:8(指定长度)）

## path
输出的文件存放目录，必须是 string 类型的绝对路径。通常通过Node.js的 path 模块去获取绝对路径。
path: path.resolve(__dirname, './dist')

## publicPath
在复杂的项目里可能会有一些构建出的资源需要异步加载，加载这些异步资源需要对应的URL地址。
publicPath: 配置发布到线上资源的 URL 前缀，为 string 类型。默认值是空字符串 ，即使用相对路径。
