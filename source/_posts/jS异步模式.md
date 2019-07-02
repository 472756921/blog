---
title: jS异步模式
date: 2019-07-02 16:50:27
tags:
---

之前一直想学习一下JS的 异步机制 和不断进化的 异步方案，现在整理记录一下
<a name="OBYhp"></a>
# 什么是异步
AJAX是异步？对也不对。<br />异步任务指的是，不进入主线程、而进入任务队列（task queue）的任务（JS是单线程，同步任务只能在主线程中顺序执行），只有任务队列通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。
<a name="uOgcb"></a>
# 由AJAX开启异步之旅
我们知道网络请求是一个耗时操作，同步网络请求会阻塞主线程，影响后续任务执行。因此，AJAX（异步请求）概念的出现很大程度上解决了当时遇到的问题（当然AJAX还解决了交互和其他的一些问题）。<br />AJAX的任务会进入任务队列执行，完成后会去通知主线程，我准备好了，调用我吧。于是乎我们传入的 function就开始执行。
```javascript
// jQ AJAX
$.ajax({
  url: myUrl,
  type: 'get',
  dataType: 'json',
  timeout: 1000,
  success: function (data, status) {
    console.log(data)
  },
  fail: function (err, status) {
    console.log(err)
  }
})
```
上面代码中，我们可以看到 success 和 fail ，分别传入了对应的 function 我们把它叫做回调函数（CallBack-function）。因此引入我们对于 callback 的一些学习。
<a name="HVPoj"></a>
# Callback
我们今天讲异步，但不是所有的 callback 都是异步执行的，比如数组方法 map 中的 callback 就是同步执行的。<br />我们看一个经典的例子：
```javascript
setTimeout(function cb() {
  output('callback')
}, 1000)
```
这个例子有两个不是很明显但重要的问题

1. setTimeout由谁提供，浏览器对吧。所以他不是JS的API，对宿主环境有依赖有要求。
1. 不能控制 CB 的调用。

我们再看一个例子：
```javascript
setTimeout(function() {
  output('one')
  setTimeout(function() {
    output('two')
    setTimeout(function() {
      output('three')
    }, 1000)
  }, 1000)
}, 1000)
```
是不是有种头疼的感觉（其实还好，后面会更加头疼），我们一直讲，callback 有个比较大的问题叫做地狱回调，但实际上我没有理解到其真正的含义。
<a name="51Wp8"></a>
## 地狱回调
从根本上来讲，回调地狱并非仅仅指在代码的编写上使用多层级回调函数嵌套，本质上应该是 回调函数 对于时间的控制，又或者说 同步任务 和 异步任务 的结合。<br />举个例子，我们有4个网络请求同时发出，然后需要按照顺序来显示结果，这个时候我们怎么保证顺序输出呢?，为了保证请求的并发，使用请求嵌套的方式满足不了需求，我们需要写一套非常复杂的控制方法来实现。<br />但是我们可以换个角度想，我们去不关心请求的过程，只关心得到请求结果后的展示。及我们得想办法将时间概念排除在编程之外。由此引入 thunk 这个概念
<a name="2vRHF"></a>
# Thunk
我们来看一下 thunk 的使用。<br />thunk是一个比较老的概念，详情可以参见阮老师的[博客](http://www.ruanyifeng.com/blog/2015/05/thunk.html)
```javascript
function getFile(file) {
  let resp

  fakeAjax(file, function(text) {
    if (!resp) resp = text
    else resp(text)
  })

  return function th(cb) {
    if (resp) cb(resp)
    else resp = cb
  }
}

const th1 = getFile('file1')
const th2 = getFile('file2')
const th3 = getFile('file3')

th1(function ready(text) {
  output(text)
  th2(function ready(text) {
    output(text)
    th3(function ready(text) {
      output(text)
      output('Complete!')
    })
  })
})
```
我们来研究一下上面一段代码，请求文件（getFile）后返回了一个接收callback函数的函数，我们可以简单地理解为：请求我做了，但是后续由用户自行决定。比起纯 Callback 方案，利用 thunk，我们的代码好理解很多。thunk 的本质其实是使用闭包来管理状态，因此我们不再管理复杂的时间维度。
> 异步 thunk 让时间不再是问题，如果我们换个角度看 ，它就好似是给一个未来的值添加了展位符。有没有觉得这种说法似曾相识，没错，Promise 也是如此。在 Promise 中，时间也被抽离了出去。
> 虽然异步 thunk 抽离出时间后，我们的代码稍微更好理解了，但是回调的另外一个问题 — 依赖反转，通过 thunk 却难以克服。如果用人话来说「依赖反转」，其实这是一种信任问题，回调函数的调用其实是受外界控制的，其会不会被调用，会被调用几次都不能完全受我们控制。为了解决这个问题，Promise 粉墨登场。
> ——[https://mp.weixin.qq.com/s/xrdEwB4uxIFhNIg5xr-X7w](https://mp.weixin.qq.com/s/xrdEwB4uxIFhNIg5xr-X7w)

<a name="tvmF2"></a>
# Promise
初次接触promise，只是为了取代 callback 的回调地狱，但当我们了解回调地狱的本质后会发现，promise解决的并非 多层级嵌套造成的不便，而是对于回调函数执行的控制（Promise 的核心在于其通过一种协议保障了then后注册的函数只会被执行一次）。普通的 callback 实际上是第三方直接调用我们的函数，这个第三方不一定是完全可信的，我们的回调函数可能会被调用，也可能不会调用，还可能会调用多次。Promise 则将代码的执行控制在我们自己手里，要么成功要么失败，then后面的函数只会执行一次。<br />但是promise依旧存在一个问题，链式调用就像一列火车，一旦开启就无法控制让其停下。为此还专门讨论了一些中断Promise的方法。<br />也正是由于Promise的不可控特性，generator 光荣出厂
<a name="OWVrZ"></a>
# Generator 
Generator 的中文名称是生成器，它是ECMAScript6中提供的新特性。在过去，封装一段函数。函数只存在“没有被调用”或者“被调用”的情况，不存在一个函数被执行之后还能暂停的情况，而Generator的出现让这种情况成为可能。<br />归纳起来 generator 函数具有以下特点：

- 函数可暂停和继续；<br />
- 可返回多个值给外部；<br />
- 在继续的时候，外面也可以再传入值；<br />
- 通过 Generator 写的异步代码看起来就像是同步的；

generator除了return语句外，可以用yield返回多次，也可以看成generator遇到yield是就返回一个值，并且generator暂停在这，当需要继续执行是只用调用generator.next()就可以继续执行，遇到yield又会暂停，再调用generator.next()后继续执行……<br />每次调用 next 方法，会返回一个对象，表示当前阶段的信息（ value 属性和 done 属性）。<br />value 属性是 yield 语句后面表达式的值，表示当前阶段的值；<br />done 属性是一个布尔值，表示 Generator 函数是否执行完毕，即是否还有下一个阶段。<br />由此，generator的作用已经很明显了。具体可以参加阮老师[博客](http://www.ruanyifeng.com/blog/2015/04/generator.html)

generator 把我们的代码分割成了独立可阻塞的部分，局部的阻塞不会导致全局的阻塞，每当遇到yield就会暂停，就需要我们手动调用 .next()来继续执行后面的内容。这个方法在任何地方都可能被调用，因此又出现了在 callback 中出现过的「控制反转」问题。我们完全不知道谁会在什么地方调用.next()，结合 Promise 我们可以比较轻松的解决「控制反转」的问题，一些人把 Promise + Generator 当作是异步最好的解决方案之一。<br />看一个例子
```javascript
// 这里封装的 runGenerator 可以让 generator 自动运行起来
function runGenerator(g) {
  const it = g()
  let ret
  (function iterate(val) {
    ret = it.next(val)
    if (!ret.done) {
      if ('then' in ret.value) {
        ret.value.then(iterate)
      } else {
        setTimeout(function() {
          iterate(ret.value)
        }, 0)
      }
    }
  })()
}

function makeAjaxCall(url, cb) {
  setTimeout(cb(url), 1000)
}

function request(url) {
  return new Promise(function(resolve, reject) {
    makeAjaxCall(url, function(err, text) {
      if (err) reject(err)
      else resolve(text)
    })
  })
}

runGenerator(function* main() {
  let result1
  try {
    result1 = yield request('http://some.url.1')
  } catch (err) {
    output('Error: ' + err)
    return
  }
  const data = JSON.parse(result1)

  let result2
  try {
    result2 = yield request('http://some.url.2?id=' + data.id)
  } catch (err) {
    output('Error: ' + err)
    return
  }
  const resp = JSON.parse(result2)
  output('The value you asked for: ' + resp.value)
})
```
上述代码具体来说，我们可以在 Promise 中调用 .next()，Promise 机制保证了.next() 的调用是受控制的。<br />但是对于generator来讲，每次封装一个generator函数无疑是很复杂的一件事情，因此async的出现解决了generator函数声明的复杂问题。
<a name="o3j77"></a>
# Async
一句话，async 函数就是 Generator 函数的语法糖。但是，我们应当明确，async 函数并非一种让 generator 更便于使用的语法糖。async 函数只有在结束时，才会返回的是一个 Promise。我们无法控制其中间状态，而 generator 返回的是迭代器，迭代器让你有充分的控制权。
