---
title: '''redux解析'''
date: 2018-10-26 09:36:32
tags:
---

## store
存储数据-修改数据-监听数据
因此还需要引入 订阅事件，取消订阅，事件发布 这一套事件管理机制
<font color='red'>Redux 的 store 就是数据管理与事件订阅的组合</font>
dispatch：更新状态、触发回调; 基本上就干了两件事：更新当前状态，触发监听回调
subscribe：注册回调、做好清理; 第一是把回调注册到nextListeners中，供dispatch时调用；第二是返回一个清理注册函数的函数，便于用户进行注册函数的清理。


## 中间件（middleware）
宏观意义上的中间件，定义十分模糊
后来它逐渐的变成了一种对框架的扩展方式，成为一种开放接口，用户可以通过自行编写中间件，控制数据在框架内部的流动方式、数据的内容，或者是函数的应用方式。
<image src='http://img.mp.itc.cn/upload/20170207/e5ab4c2ffe1c495daed3f37fcd5daa62.jpeg'/>
redux中对中间件的解释:
<font color='red'>从一个 action 被调度（dispatch），到这个 action 到达 reducer，中间件在这两个时刻中间提供了一个第三方扩展的植入点</font>
<font color='red'>换句话说，Redux 中间件实际上是对 store.dispatch方法的劫持</font>
 Redux 的中间件机制提供了一个切入点，让用户可以自行控制 action 流向 reducer 的时机和 action 的内容；控制的方式就是实现一个 wrapper 来接替原始的 dispatch方法，并在 wrapper 中可以异步执行原始 dispatch。

## redux的源码组成
主要有几个关键文件：
1. createStore.js
2. applyMiddleware.js
3. bindActionCreators.js
4. compose.js
5. combineReducers.js

### createStore.js

``` JavaScript
import isPlainObject from 'lodash/isPlainObject'
import $$observable from 'symbol-observable'

//redux自己创建的action，用来初始化状态树和reducer改变后初始化状态树
export const ActionTypes = {
  INIT: '@@redux/INIT'
}

export default function createStore(reducer, preloadedState, enhancer) {
  //如果第二个参数为方法且第三个参数为空，则将两个参数交换
  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState
    preloadedState = undefined
  }

  //enhancer和reducer必须为function类型
  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.')
    }
    //将enhancer包装一次createStore方法，再调用无enhancer的createStore方法
    return enhancer(createStore)(reducer, preloadedState)
  }

  if (typeof reducer !== 'function') {
    throw new Error('Expected the reducer to be a function.')
  }

  let currentReducer = reducer //当前的reducer函数
  let currentState = preloadedState //当前的state树
  let currentListeners = []      //监听函数列表
  let nextListeners = currentListeners  //监听列表的一个引用
  let isDispatching = false      //是否正在dispatch

  function ensureCanMutateNextListeners() {
    if (nextListeners === currentListeners) {
      nextListeners = currentListeners.slice()
    }
  }

  //下面的函数被return出去，函数作为返回值，则形成了闭包，currentState等状态会被保存

  //返回当前state树
  function getState() {
       //...
  }

  //添加注册一个监听函数，返回一个可以取消此监听的方法
  function subscribe(listener) {
       //...
  }

  function dispatch(action) {
      //...
  }

  //替换当前reducer
  function replaceReducer(nextReducer) {
      //...
  }

  function observable() {
      //...
  }

  // 当store被创建的时候，初始化状态树
  dispatch({ type: ActionTypes.INIT })

  return {
    dispatch,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
  }
}
```
createStore接受三个参数
1. reducer：这个参数是一个函数，返回下一个 state 树
2. preloadedState：preloadedState 是初始的 state
3. enhancer：第三个参数是一个 store 增强器，函数类型，只能使用 'applyMiddleware' 方法来生成
这个方法最终生成一个对象，对象中包含了一个重要的成员变量，即State树，还暴露了几个成员方法
1. dispatch：redux 中唯一触发 state 树修改的方法，分发一个 action，然后在该 action 对应的处理函数中返回一个新的 state 树
2. subscribe：给 store 的状态树添加监听函数，一旦 dispatch 被调用，所有的监听函数就会被执行
3. getState：返回当前的状态树
4. replaceReducer：替换当前 store 中 reducer 的方法


#### subscribe
``` JavaScript
    function subscribe(listener) {
        if (typeof listener !== 'function') {
          throw new Error('Expected listener to be a function.')
        }

        let isSubscribed = true

        ensureCanMutateNextListeners()
        nextListeners.push(listener)

        return function unsubscribe() {
          if (!isSubscribed) {
            return
          }

          isSubscribed = false

          ensureCanMutateNextListeners()
          const index = nextListeners.indexOf(listener)
          nextListeners.splice(index, 1)
        }
      }
```
这个函数用于给 store 添加监听函数，并返回取消监听的函数。
添加了之后 subscribe 方法会返回一个 unsubscribe() 方法，此方法用于注销刚才添加的监听函数。

#### dispatch
``` JavaScript
function dispatch(action) {
    //action必须是一个纯对象
    if (!isPlainObject(action)) {
      throw new Error(
        'Actions must be plain objects. ' +
        'Use custom middleware for async actions.'
      )
    }

    //action必须是一个包含type的对象
    if (typeof action.type === 'undefined') {
      throw new Error(
        'Actions may not have an undefined "type" property. ' +
        'Have you misspelled a constant?'
      )
    }

    //如果正处于isDispatching状态，报错
    if (isDispatching) {
      throw new Error('Reducers may not dispatch actions.')
    }

    try {
      isDispatching = true
      //这里就是调用我们reducer方法的地方，返回一个新的state作为currentState
      currentState = currentReducer(currentState, action)
    } finally {
      isDispatching = false
    }

    //调用所有的监听函数
    const listeners = currentListeners = nextListeners
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }

    return action
  }
```
关键代码：<font color='red'>currentState = currentReducer(currentState, action)</font>
它就是把action和state搞到reducer里面去摩擦摩擦，得到一个新的state。
然后在执行监听，当state变化了，监听知道了，就启用监听的回调。

### compose.js
``` JavaScript
export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}

```
注意：funcs.reduce((a, b) => (...args) => a(b(...args)))   参看 “扩展运算符”
其作用是把一系列的函数，组装生成一个新的函数，并且从后到前，后面参数的执行结果作为其前一个的参数。
```js
var fn1 = val => 'fn1-' + val
var fn2 = val => 'fn2-' + val
var fn3 = val => 'fn3-' + val
compose(fn1,fn2,fn3)('测试')
输出结果： fn1-fn2-fn3-测试
```
当需要把多个 store 增强器 依次执行的时候，需要用到它。


### applyMiddleware.js
```js
export default function applyMiddleware(...middlewares) {

  //return一个函数，它可以接受createStore方法作为参数，给返回的store的dispatch方法再进行一次包装
  return (createStore) => (reducer, preloadedState, enhancer) => {
    const store = createStore(reducer, preloadedState, enhancer)
    let dispatch = store.dispatch
    let chain = []

    //暴露两个方法给外部函数
    const middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    }

    //传入middlewareAPI参数并执行每一个外部函数，返回结果汇聚成数组
    chain = middlewares.map(middleware => middleware(middlewareAPI))

    //这里用到了刚才说到的compose方法，如果你忘了，返回去看一下
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}

```
createStore方法中的enhancer就是用applyMiddleware函数来创建的。
它是 Redux 的原生方法，作用是将所有中间件组成一个数组，依次执行。并且将getState和dispatch两个方法通过middlewareAPI传递进入中间件内部。
特别注意中间件的加载顺序，在某些情况下，中间件需要按照特定的顺序加载。
而编写中间件需要注意异步中间件的情况，其实也很简单，就是判断action类型，如果是方法的话直接执行他，如果不是的话，就把action传递出去。
所以说，applyMiddleware实际上做了一件事，就是根据外部函数(中间件函数)包装原来的dispatch函数，然后将新的dispatch函数暴露出去。

### bindActionCreators.js
这个方法主要的作用就是将action与dispatch函数绑定，生成直接可以触发action的函数。

### combineReducers.js
combineReducers 辅助函数的作用是，把一个由多个不同 reducer 函数作为 value 的 object，合并成一个最终的 reducer 函数，然后就可以对这个 reducer 调用createStore。
合并后的 reducer 可以调用各个子 reducer，并把它们的结果合并成一个 state 对象。state 对象的结构由传入的多个 reducer 的 key 决定。
