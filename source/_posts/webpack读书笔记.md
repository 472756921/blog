---
title: '''webpack读书笔记'''
date: 2018-11-12 17:47:17
tags:
---

# 为什么会有webpack
现代web开发中模块式是一个必备解决方案，因此诞生以一些列模块式工具，诸如AMD，CMD，commonJS，ES6模块等概念和工具。
> 前端模块要在客户端中执行，所以他们需要增量加载到浏览器中。
模块的加载和传输，我们首先能想到两种极端的方式，一种是每个模块文件都单独请求，另一种是把所有模块打包成一个文件然后只请求一次。显而易见，每个模块都发起单独的请求造成了请求次数过多，导致应用启动速度慢；一次请求加载所有模块导致流量浪费、初始化过程慢。这两种方式都不是好的解决方案，它们过于简单粗暴。
引自-《Webpack 中文指南》

> 分块传输，按需进行懒加载，在实际用到某些模块的时候再增量更新，才是较为合理的模块加载方案。要实现模块的按需加载，就需要一个对整个代码库中的模块进行静态分析、编译打包的过程。
引自-《Webpack 中文指南》

> 在编译的时候，要对整个代码进行静态分析，分析出各个模块的类型和它们依赖关系，然后将不同类型的模块提交给适配的加载器来处理。比如一个用 LESS 写的样式模块，可以先用LESS加载器将它转成一个CSS 模块，在通过 CSS 模块把他插入到页面的'style'标签中执行。Webpack 就是在这样的需求中应运而生。
引自-《Webpack 中文指南》

# webpack 的身世

## webpack本地（局部）安装如何运行
在本地安装webpack后，由于没有环境变量，所以无法直接使用webpack，这个时候我们可以借助npm实现webpack的使用
如在package.json中配置：
"dev": "webpack --mode development".
"build": "webpack --mode production".( --mode production表示生产环境)

## 什么是 Webpack
Webpack 是一个模块打包器。它将根据模块的依赖关系进行静态分析，然后将这些模块按照指定的规则生成对应的静态资源。
包括：
1. 代码转换
2. 文件优化（压缩文件，合并图片）
3. 代码分割（提取公共代码，让首屏不需要的代码和资源异步加载）
4. 模块合并
5. 代码验证
6. 自动发布（代码更新后，自动构建发布）
***
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
***
## Plugin
Plugin是用来扩展webpack功能的。在 Webpack 构建流程中的特定时机注入扩展逻辑，来改变构建结果或做我们想要的事情。
这是一个Plugin的例子，用于单独打包css
``` JS
module: {
    rules: [
        {
            test: /\.css$/, //这个时候就不需要style-loader把CSS注入到js中了
            loader:[MiniCssExtractPlugin.loader,'css-loader']
        }
    ]
},
plugins: [
    new MiniCssExtractPlugin({
        filename: `[name]_[contenthash:8].css`,
    })
]
```
***
## source-map
有了这个东西就可以映射源码。
在config中配置：devtool: 'source-map'（与entry同级） 即可
***
## webpack中的各种概念
module：模块，一个模块对应一个文件，Webpack会从entry开始，递归找出所有依赖的模块。
loader：模块转换器，将一个模块按照要求转化成新的内容。
plugins：插件，在构建流中特定的时机（取决于插件开发者，我们一般是被动的使用）注入扩展逻辑，来改变结构达到我们想要的效果。
# webpack 详细
前面讲讲了一大堆，想必对webpack有了一个大致的了解，下面的部分将详细记录webpack的各种工作和功能。

## Content
Content是寻找文件的根目录，默认启动webpack时所在的目录，可设置指定
content:path.resolve(__dirname, 'XXX'),   （xxx绝对路径）
## Entry
Entry是配置的模块的入口，搜寻及递归解析出所有入口依赖的模块（必填）。
Entry有三种配置方式
1. string
2. array（搭配 output.library 配置项使用时，只有数组里的最后一个入口文件的模块会被导出）
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
可以理解为‘代码块’
如果entry是一个String/array，只会生成一个chunk，名字是mian
如果entry是一个Obj，就可能出现多个Chunk，名字是obj中的键值(key)
***
## Output
filename: 输出文件名
如果只有一个chunk的话，filename 可使用静态string，如果是多个chunk的话则可用变量模板，例如 '[name].js'
这里的[name]是chunk内置的name变量去替换（除了name还有id，hash->hash:8，chunkhash->chunkhash:8(指定长度)）

### path
输出的文件存放目录，必须是 string 类型的绝对路径。通常通过Node.js的 path 模块去获取绝对路径。
path: path.resolve(__dirname, './dist')

### publicPath
在复杂的项目里可能会有一些构建出的资源需要异步加载，加载这些异步资源需要对应的URL地址。
publicPath: 配置发布到线上资源的 URL 前缀，为 string 类型。默认值是空字符串 ，即使用相对路径。
可配置IP地址，发布到CDN上

### library 和 libraryTarget
这两兄弟是构建可以被其他模块导入的库时要用到的东西。
略过

### libraryExport
配置导出模块中哪些子模块需要被导出
同上略过

开发组件库时需要用到上面两个东西
***
## module
配置模块处理规则

### rules
创建模块时，匹配请求的规则 对象数组。
这些规则能够修改模块的创建方式。这些规则能够对模块(module)应用 loader，或者修改解析器(parser)。
#### loader
loader 用于对模块的源代码进行转换。loader 可以使你在 import 或"加载"模块时预处理文件。
loader 可以将文件从不同的语言（如 TypeScript）转换为 JavaScript，或将内联图像转换为 data URL。
一个 loader 主要有三个步骤(同级)
1. 匹配文件，主要使用 test（正则匹配，匹配文件格式或名称），include（配置匹配目录-缩小范围：path.resolve(__dirname, 'src')），exclude（配置排除目录） 三项配置。
以上三项配置还支持数组形式，它们是或条件。
2. 使用何种手段（loader）处理
    use：['xxx_loader','yyy_loader']
    or
    use:[ { loader: xxx_loader, options: { xxx: xxx }, enforce: 'post' } ]
3. 设置执行循序
默认use中的 loader 执行循序是从后往前，我们可以通过 enforce 来设置 第一个（pre）或最后一个（post）执行。
注意这个配置只能在 loader 使用对象的时候添加。

### noParse
防止 webpack 解析那些任何与给定正则表达式相匹配的文件。
noParse: /jquery|chartjs/    不使用webpack处理匹配到的这两个文件。
特别注意：被排除的文件中不应该含有 import, require, define 的调用，或任何其他导入机制，否知浏览器直接调用将会无法解析。

### parser
控制哪些模块语法被解析，哪些不被解析（相比noParse对文件的控制，这个家伙是在语法层面上做控制）
``` js
parser: {
  amd: false, // 禁用 AMD
  commonjs: false, // 禁用 CommonJS
  system: false, // 禁用 SystemJS
  harmony: false, // 禁用 ES2015 Harmony import/export
  requireInclude: false, // 禁用 require.include
  requireEnsure: false, // 禁用 require.ensure
  requireContext: false, // 禁用 require.context
  browserify: false, // 禁用特殊处理的 browserify bundle
  requireJs: false, // 禁用 requirejs.*
  node: false, // 禁用 __dirname, __filename, module, require.extensions, require.main 等。
}
```
***
## resolve
能设置模块如何被解析（就是如果找到模块）。
我们在开发中会用到 import， require 等方法来导入模块，resolve 就是用来设置如何找到对应模块的配置。
它大概需要这么几个属性
### alias
路径映射：创建 import 或 require 的别名，来确保模块引入变得更简单。例如，一些位于 src/ 文件夹下的常用模块：
``` js
alias: {
  Utilities: path.resolve(__dirname, './src/utilities/'),
  components: path.resolve(__dirname, './src/components/'),
}

// 引用 before
import Utility from '../../utilities/utility';
import Button from './src/components/button';

// after
import Utility from 'Utilities/utility';
import Button from 'components/button';

```
### mainFields
配置优先使用的代码版本
mainFiels: [ 'jsnext: main', 'browser', 'main' ];
其中, jsnext:main 代表使用 ES6 的代码， main 使用 ES5 的代码。
只要按顺序找到其中的第一个就使用，并停止查找

### extensions
补偿 import 或 require 等方式引入文件的后缀
默认 extensions: ['js', 'json']
***
## plugin
扩展webpack功能的, 这玩意儿主要依赖社区开发，所有要了解各种 plugin 的方法参数，带入写法都是一致的。
```JS
plugins: [
    new MiniCssExtractPlugin({
        filename: `[name]_[contenthash:8].css`,
    }),
    new CommonChunkPlugin({
        name: 'common',
        chunks: [ 'a', 'b' ],
    }),
]
```
了解更多的 plugin 用法，需要详细到各个 plugin 的开发文档中去了解。
***

## DevServer
这个也可以写入配置文件，BUT 这个东西只能是通过 DevServer 启动 webpack 的时候才会生效。

### hot
热更新，注意实现方式
<font color='red'>他是通过用新的模块来替换旧的模块来实现实时预览功能的，不是刷新</font>

在hot:false的情况系下，DevServer 默认是通过刷新整个页面来实现实时预览的。

### historyApiFallback
单页应用路由命中时都要求返回一个 index.html 文件，然后将 JS 代码注入。
如果我们的应用有多个单页组成，那么根据路由的不同，要求返回不同的 index.html，这个时候我们就要使用这个家伙了。
```JS
    historyApiFallback: {
        rewrites: [
            { from: /^/user/, to: '/user.html' },
            { from: /^/list/, to: '/list.html' },
            { from: /^/test/, to: '/test.html' },
            { from: /./, to: '/index.html' },    //其他的路由都返回 index.html
        ]
    }
```

### host, port
设置地址（默认 127.0.0.1）和端口（默认 8080，如果被占用，向上加 1）

### disableHostCheck
访问 DevServer 时会对访问者的 HOST 进行检查，默认只接受本地访问。
关闭这个选型，能够让其他设备访问自己的本地服务，服务器就不会再去检查host，如果不关闭的话，由于其他设备是通过 IP 访问， 一般情况下是不会在host列表中，就会造成不能访问。

### https
DevServer 会自动生成一套证书。
甚至于可以配置证书
```JS
    devServer: {
        https: true,
        https: {
            key: '',
            cert: '',
            ca: '',
        },
    }

    // 以上二选一
```

### compress
开启 Gzip 压缩，默认false

### open openPage
open 开启打开浏览器
openPage 打开的URL

### proxy
代理到后端的接口
```JS
    proxy: {
        '/api': 'http:xxxx.com/xxx:30'
    }
```

### profile
开启构建性能信息，分析构建性能

### cache
开启缓存来提升构建速度。

### Watch WatchOption
监听文件更新，DevServer 默认开启
```JS
WatchOption:{
    ignored: /XXXX/,         // 不监听的文件夹
    aggregateTimeOut: 400，  // 400MS后再去重新编译，本质是截流
    poll: 1000               // 每秒 1000 次询问更新
}
```

***
## 其他配置
### target
构建不同环境的代码
1. web
2. node
3. async-node  异步加载 Chunk 代码
4. webworker
由于不同环境，打包过程会有一些不同。

### Externals
打包过程排除掉的模块，通常用于第三方库。
举个栗子： jQuery
如果使用了 import $ from 'jquery'，
在最后打包的文件中，会将 JQ 的内容完整的注入进去，但显然我们并不想这样子。
```JS
externals: {
  jquery: 'jQuery'
},
// 这样子我们就把 Jq 排除在了打包文件之外。
```
再来说打包后对 JQuery 的使用。
在引用 JQuery 后，会在全局（global）生成一个 jQuery 全局对象（注意大小写），externals 的工作实际上就是当 require 的参数是 jquery 的时候，使用 jQuery 这个全局变量引用它。
打包后的源码中会有这样一句
```JS
function(...) {
    // 注意这里的 jQuery 指全局变量
    // 因此我们 打包后源码使用 require 来引入 jquery 时，会直接使用全局变量 jQuery。
    module.exports = jQuery;
},
```
***
# webpack 多种配置写法

## function 写法
```JS
module.exports = function (env = {}, argv) {
  const plugins = [];

  const isProduction = env['production'];

  // 在生成环境才压缩
  if (isProduction) {
    plugins.push(
      // 压缩输出的 JS 代码
      new UglifyJsPlugin({
        // 最紧凑的输出
        beautify: false,
        // 删除所有的注释
        comments: false,
        compress: {
          // 在UglifyJs删除没有用到的代码时不输出警告
          warnings: false,
          // 删除所有的 `console` 语句，可以兼容ie浏览器
          drop_console: true,
          // 内嵌定义了但是只用到一次的变量
          collapse_vars: true,
          // 提取出出现多次但是没有定义成变量去引用的静态值
          reduce_vars: true,
        }
      })
    )
  }

  return {
    // JS 执行入口文件
    entry: './main.js',
    output: {
      // 把所有依赖的模块合并输出到一个 bundle.js 文件
      filename: 'bundle.js',
      // 输出文件都放到 dist 目录下
      path: path.resolve(__dirname, './dist'),
    },
    plugins: plugins,
    // 在生成环境不输出 Source Map
    devtool: isProduction ? undefined : 'source-map',
  };
}


// script 配置
"scripts": {
    "build:dev": "webpack",
    "build:dist": "webpack --env.production"
},
```
<font color='red'>特别注意：在webpack4 中不再自定义环境参数，需要使用 --mode 作为开发/生成环境的区别</font>
***
# 自动化生成 index.html
根据打包后的文件，自动生成index.html，并引入相关文件。
这里需要用到插件 web-webpack-plugin 。

## 单页应用
配置也很简单
```JS
entry: {
    app: './entry.js',
},
plugins: [
    new WebPlugin({
        template: './index.html',     // 指定本地模板
        filename: 'index.html'        // 指定打包后的文件名
    })
],
```
<font color='red'>要点：</font>
在本地模板中配置加载的 css 和 js
``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <link rel="stylesheet" href="app">
</head>
<body>
    <div id="app"></div>
    <script src="app"></script>
</body>
</html>
```

注意其中的 src 和 href 都使用了 'app'， 这个是根据 webpack 打包入口配置chunk名称。

## 多个单页应用
在实际开发中，也许会将应用依据功能不同划分成多个单页应用。
为避免手动配置 webpack （当然也可以手动配置），将使用 web-webpack-plugin 插件的 AutoWebPlugin 功能来解决问题。
先上一个手动多入口的配置
```JS
entry: {
    home: './src/home.js',
    user: './src/user.js',
},

plugins: [
    new WebPlugin({
        template: './index.html',
        filename: 'Home.html',
        requires: ['home'],
    }),
    new WebPlugin({
        template: './index.html',
        filename: 'User.html',
        requires: ['user'],
    })
],
```
下面使用 AutoWebPlugin 进行自动打包，先看配置文件
```JS
   const { AutoWebPlugin } = require('web-webpack-plugin');

   const autoWebPlugin = new AutoWebPlugin('src', {
       template: './index.html',
       postEntrys: [ './src/common.css' ],
       commonsChunk: {
           name: 'common',
       }
   });
   module.exports = {
       entry: autoWebPlugin.entry(),
       output: {
           path: path.resolve(__dirname, 'dist'),
           filename: `[name]_[chunkhash:8].js`
       },
       plugins: [
           autoWebPlugin,
       ],
   };
```

<font color='red'>注意几个要点：</font>
1.AutoWebPlugin 对入口的目录是约定式的，要求入口文件必须是 'src' (可配置，上面new AutoWebPlugin的第一个参数)文件夹下，文件夹（这个文件夹名称就是Chunk的名称），中的 index.js(不可更改)。
2.postEntrys 是将 common.css 文件作为公共样式。打包后会为每一个 chunk 生成一个独立的css文件，AutoWebPlugin会将 common.css 的内容添加进入这个css文件。
3.使用该插件后，无需再 模板 index.html 中再指定 css 和 js 的路径，它将自动注入。

