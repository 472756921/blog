---
title: JS模块化
date: 2019-07-02 17:00:07
tags:
---
## CommonJS
服务器端js模块化的规范，同步的模块加载–(代表：NodeJS)
## AMD
浏览器端js模块化的规范，提前执行（异步加载：依赖先执行）–(代表：RequireJS)
## CMD
也是浏览器端js模块化的规范，延迟执行（运行到需加载，根据顺序执行）–(代表：SeaJS)
## ES6模块化
ES6模块化出现后，将逐步地取代commonJS和AMD，成为浏览器和服务器的通用模块化解决方案。
但是ES6模块无法直接运行在浏览器环境，需要转成ES5后才能正常运行。

# 接下来我们详细地解析一下

## CommonJS 实现原理
<font color=red>在nodeJS环境下</font>
A.js
```js
    module.exports = '欢迎使用 CommonJS';
```
B.js
```js
    let fs = require('fs'); //node 文件读取
    let show = require('./a.js');
    // 重写实现 require 方法
    function requireByBenson(moduleName) {
        let content = fs.readFileSync(moduleName, 'utf8');
        let fn = new Fucntion('export', 'module', 'require', '__dirname', '__filename', content + '\n return module.exports');
        let module = { exports: {} };
        return fn(module.exports, module, req, __dirname, __filename);
    }

    let temp = requireByBenson('./a.js');
    console.log(temp);  // 欢迎使用 CommonJS
```

## AMD
浏览器端（也可以允许与node端），异步加载模块
A.js
```js
    define('name', [], function() {return 'require'});

    //重写 define 方法

    let factorys = {};   //全局变量
    function define(moduleName, dependencies, factoryFunction) {
        factoryFunction.dependencies = dependencies;
        factorys[moduleName] = factoryFunction;
    }
```
B.js
```js
    require(['name'], function(name){console.log(name)});

    //重写 require 方法
    function require(name, callBack) {
        let res = name.map(function(item) {
            let exports;
            require(factoryFunction.dependencies, function() {
                exports = factorys[name].apply(null, arguments);
            });
            return exports ;
        })
        callBack.apply(null, res);
    }
```

## CMD
同步模块定义
...待续