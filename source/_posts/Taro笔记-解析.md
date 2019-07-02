---
title: Taro笔记(解析)
date: 2019-04-27 18:38:29
tags:
---
Taro，京东凹凸团队搞的一个多端框架（小程序，rn，h5，）遵循 React 语法规范，组件生命周期与 React 保持一致，同时支持TS。
Taro 还是以小程序为基础，封装了一系列的组件，并且可以集成redux来干状态管理。
安装使用过程看文档：https://nervjs.github.io/taro/docs/GETTING-STARTED.html。
（ps:本人看了写原生小程序开发文档，果断放弃原生的坑）

## 记一下使用心得
1.Taro封装的组件用来开发小程序应该是能满足90%的场景和需求，如果满足不了，请另想办法。
2.路由传参，复杂参数就不要想通过这种方式了。
3.毕竟是对小程序的支持比较好，在H5和RN上你可能会遇到一些没头脑的问题。
4.可以干异步编程，这个是个不错的更新。
5.消息机制用好了，可以解决大部分问题，但这个应该只是个标配。
好吧，我编不出来了。

## 写一点Taro核心
小程序和WEB乃至RN在API和组件上有着巨大的差异，不能简单的通过直译的方式将 <View> 转化成 <Div> 。因此Taro干了一套标准来处理这个问题。
>这一套标准主要以三个部分组成，包括标准运行时框架、标准基础组件库、标准端能力 API，其中运行时框架和 API 对应 @taro/taro，组件库对应 @tarojs/components，通过在不同端实现这些标准，从而达到去差异化的目的。
 —摘自 《Taro 多端开发实现原理与项目实战》第8章

同时作者也在上文中描述了Taro为什么会以小程序作为组件和API标准的原因。

## Taro的灵魂 taro build
少喝水。

taro build 是实现多端编译的核心，我甚至把他看做整个Taro和核心。简单的讲，把一种结构化语言转化成另外一种，要经历至少三个步骤。
1.代码结构化转化（把代码结构转化成抽象语法树，然后进行遍历和替换，最后根据新的规则生成代码）；
2.babel编译转换（Taro也主要使用了这个方式，主要针对于API的语法规范）；
3.解析配置文件；
写的真TM简单。。。接着看Taro源码。

## 先看 taro-cli这个脚手架
我们对Taro的使用还是基于脚手架工具的，我们分析一下它。
我们通过项目package.json文件可以看到，对项目的操作命令实际上都是 taro 来执行。所以至关重要的是分析taro命令。通过阅读 taro-cli 的源码可知，taro命令是我们在安装 taro-cli 工具时，它将taro命令搞到了环境PATH中，以便调用（由于taro-cli的package.json中指定了bin对象，意思就是为其命令和具体的运行体做一个软连接，参考http://www.cnblogs.com/tzyy/p/5193811.html#_h1_11）。
<image src='https://cdn.nlark.com/yuque/0/2019/png/197796/1555669363481-2e8a4ef4-8277-4e57-8141-bef9cbb92f2f.png'/>
我们可以看到核心就一个 taro build 这个命令实际上是调用 taro-cli 的 taro 子命令 build，我们来看一下关键源码。
``` JS
const program = require('commander')
const {getPkgVersion} = require('../src/util')

program.version(getPkgVersion())
  .usage('<command> [options]')
  .command('init [projectName]', 'Init a project with default templete')
  .command('build', 'Build a project with options')
  .command('update', 'Update packages of taro')
  .command('convert', 'Convert weapp to taro')
  .command('info', 'Diagnostics Taro env info')
  .parse(process.argv)
```
commander可以根据子命令自动引导到以特定格式命名的命令执行文件，文件名的格式是 [command]-[subcommand]。例如 taro build 就会去执行 taro-build 这个文件。
so，当 npm run (taro build --type weapp/taro build --type h5等) 命令到达上面文件后会被转发到同级目录下的 taro-build 文件去执行。
接下来我们就要去探究更加详细的过程了。
### 扩展知识
如果看到这里了，可能会和我一样有个疑问。到底是什么在支撑 build 功能的执行。如果看过 taro-build 的源码，会发现（PS：发现个P，作者不看其他文字才不会get到这个点）顶部有一句 #!/usr/bin/env node 。
<image src='https://cdn.nlark.com/yuque/0/2019/png/197796/1555916139023-264121dd-15eb-4742-9bd4-1702d8f6a343.png'/>
OMG，居然是它，居然是一个我从来都没见过的东西，特意查了一下。这个东东分两部分 #! URL，#! 学名叫做 Shebang 是Linux 上的命令。
1.如果脚本文件中没有#!这一行，那么它执行时会默认用当前Shell去解释这个脚本(即：$SHELL环境变量）。
2.如果#!之后的解释程序是一个可执行文件，那么执行这个脚本时，它就会把文件名及其参数一起作为参数传给那个解释程序去执行。
3.如果#!指定的解释程序没有可执行权限，则会报错“bad interpreter: Permission denied”。
4.如果#!指定的解释程序不是一个可执行文件，那么指定的解释程序会被忽略，转而交给当前的SHELL去执行这个脚本。
5.如果#!指定的解释程序不存在，那么会报错“bad interpreter: No such file or directory”。注意：#!之后的解释程序，需要写其绝对路径（如：#!/bin/bash），它是不会自动到$PATH中寻找解释器的。
<image src='https://cdn.nlark.com/yuque/0/2019/png/197796/1555916626897-d7d563b7-05b1-4498-8058-124a9dbb44c0.png'/>
可执行文件？ 卧槽，是node，是 usr/bin/env 下面的node。但是 win 呢？image.png
<image src='https://cdn.nlark.com/yuque/0/2019/png/197796/1555916943905-2852d7c6-fcd7-48fb-b477-9a1bb449221b.png'/>
我推测是直接当前shell就干了吧。

### src/build
tart-bulid文件中有一句 const build = require('../src/build')，我们又去 <a href='https://github.com/NervJS/taro/blob/master/packages/taro-cli/src/build.js'>src/build</a> 下面看。
暂时不看了，里面没啥营养，就是对环境，条件，编译方法做出判断和约定。行吧，搬运一点吧。
``` js
switch (type) {
    case Util.BUILD_TYPES.H5:
      buildForH5({ watch })
switch (type) {
    case Util.BUILD_TYPES.H5:
      buildForH5({ watch })
      break
    case Util.BUILD_TYPES.WEAPP:
      buildForWeapp({ watch })
      break
    case Util.BUILD_TYPES.SWAN:
      buildForSwan({ watch })
      break
    case Util.BUILD_TYPES.ALIPAY:
      buildForAlipay({ watch })
      break
    case Util.BUILD_TYPES.TT:
      buildForTt({ watch })
      break
    case Util.BUILD_TYPES.RN:
      buildForRN({ watch })
      break
    case Util.BUILD_TYPES.UI:
      buildForUILibrary({ watch })
      break
    case Util.BUILD_TYPES.PLUGIN:
      buildForPlugin({
        watch,
        platform: buildConfig.platform
      })
      break
    default:
      console.log(chalk.red('输入类型错误，目前只支持 weapp/h5/rn/swan/alipay/tt 六端类型'))
  }
```
​这里选个buildForWeapp来看看，我们进到 <a herf='https://github.com/NervJS/taro/blob/master/packages/taro-cli/src/weapp.js'>buildForWeapp</a> 文件。
ok，le't go
<image src='https://cdn.nlark.com/yuque/0/2019/png/197796/1555918088534-2c7d60b1-931d-4491-9b08-05ea9fe412de.png'/>
entered 
<image src='https://cdn.nlark.com/yuque/0/2019/png/197796/1555918132671-94a6d61d-53e3-4b43-b944-09d4d0e1694f.png'/>
<image src='https://cdn.nlark.com/yuque/0/2019/png/197796/1555918435424-01f95d64-795b-4f18-a10a-296250072a1c.png'/>
<image src='https://cdn.nlark.com/yuque/0/2019/png/197796/1555918449741-bb3f21c3-745f-40d9-bba2-5eea7cd23d42.png'/>
<image src='https://cdn.nlark.com/yuque/0/2019/png/197796/1555918465445-03960ffb-1491-4d8c-882f-29e8ef3c37f9.png'/>
找到关键了--------------------------------------------------------------
``` JS
async function build ({ watch, adapter, envHasBeenSet = false }) {
  process.env.TARO_ENV = adapter
  if (!envHasBeenSet) isProduction = process.env.NODE_ENV === 'production' || !watch
  buildAdapter = adapter
  outputFilesTypes = Util.MINI_APP_FILES[buildAdapter]
  // 可以自定义输出文件类型
  if (weappConf.customFilesTypes && !Util.isEmptyObject(weappConf.customFilesTypes)) {
    outputFilesTypes = Object.assign({}, outputFilesTypes, weappConf.customFilesTypes[buildAdapter] || {})
  }
  constantsReplaceList = Object.assign({}, constantsReplaceList, {
    'process.env.TARO_ENV': buildAdapter
  })
  buildProjectConfig()
  await buildFrameworkInfo()
  copyFiles()
  appConfig = await buildEntry()
  await buildPages()
  if (watch) {
    watchFiles()
  }
}
​```
首先经历一个工具配置的过程 buildProjectConfig，帮你把你的小程序配置文件生成好，这个一步操作不多，主要是你再开发环境下的配置文件基本是基于微信小程序标准的，差距不会太大。 
然后 buildFrameworkInfo ，这个是对百度小程序的一个处理。生产一个文件，具体不详，暂时也不想了解。
copyFiles应该也是生成配置文件相关。
buildEntry 这个是从入口文件开始打包，里面用了 wxTransformer ，这个应该就是taro定义的小程序转化的配置头文件

## tarojs/components
这里面封装的是Taro的组件，SO用的东西都是这里来的。
``` JS
export { default as View } from './components/view'
export { default as Block } from './components/block'
export { default as Image } from './components/image'
export { default as Text } from './components/text'
export { default as Switch } from './components/switch'
export { default as Button } from './components/button'
export { default as Icon } from './components/icon'
export { default as Radio } from './components/radio'
//  ...  还有很多自行脑补
```

## tarojs/taro
taro这个包大概看下来就是提供运行时的，核心编码采用了nervjs这个玩意。
不好意思，这个包暂时没整明白（也许把后面的看完了，这个就明白了）
taro这个包相当于一个转换器的功能，但是转换规则和核心API是由分包（taro-weapp/taro-h5等）提供的。

## tarojs/taro-webapp
这个包是用来编译小程序的核心
从package.json和index.js中可以看出，他是依赖了tarojs/taro的，因此小程序的打包还是由tarojs/taro参与。
另外在 component 组件中可以看到对小程序的转化操作。(我准备直接贴代码)
我们来细细地研究一下。（代码有点长，我还是不贴了吧）

## 待续。今天先写这么多。。。
<image src='https://cdn.nlark.com/yuque/0/2019/png/197796/1555929880714-9bd885d4-0d57-4253-8c06-037a42da22fc.png'/>