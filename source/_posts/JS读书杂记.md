---
title: '''JS读书杂记'''
date: 2018-10-15 17:06:16
tags:
---

## JS 的七种类型------类型
null， undefined, string, boolean, number, object, symbol
除了object之外，其他都叫做基本类型。

## 为啥 typeof null === 'object';  //true
这是一个JS的BUG，可能永远不会被修复。
原因：
1.对象在底层都表示为二进制
2.在 JavaScript 中二进制前三位都为 0 的话会被判断为 object 类型
3.null 的二进制表示是全 0，自然前三位也是 0

## 为啥 typeof function a(){} === 'function'; //true
因为函数实际上是object的一个子类，是可调用的对象（ps，由于内部有 [[call]] 属性保证可以被调用 ）
另外 数组也是object的一个子类型

## 变量是没有类型的，只有值才有
JavaScript 中的变量是没有类型的，只有值才有，在对变量执行 typeof 操作时，得到的结果并不是该变量的类型，而是该变量持有的值的类型。

## undefined 和 undeclared(未声明)
``` js
var a;
console.log(a)   //undefined
console.log(b)   //undeclared  (b is not defined)

typeof(a)   //undefined
typeof(b)   //undefined
```
## 为啥typeof(b)是undefined --（typeof undeclared）
typeof的安全机制作用
如果要判断一个变量是否已经被声明；
``` js
if(testVar){
    ...   //该变量未被声明 ,
}
// 上述做法肯定报错 ReferenceError
// 用typeof安全机制做
if(typeof testVar !== 'undefined' ){
    ...   //该变量已被声明 ,
}
```

## 字符串反转
例如 var a = 'abcdefg';
怎么样反转它，循环当然可以，不过有点low
我们知道字符串是没法被改变的（字符串的成员函数无法改变其原始值，而是创建一个新的值返回），but数组是可以的。
并且数组本身就拥有一个reverse（）函数用于反转数组
所以
1.先将字符串转成数组 (.split(''))
2.reverse()
3.join('')

但是注意，如果包含了星号等复杂字符的字符串是没有办法用这个方式反转的

## 0.1 + 0.2 === 0.3  //false
因为在二进制浮点数中 0.1和0.2表示并不是十分精确的。

## 不是值的值
undefined 和 null 他们既是类型又是值
null指空值，undefined指没有值

## 不是数字的数字
数学运算中含有不是数字类型（无法解析为16、10进制），就返回一个NaN（not a number）
注意这个Nan 他是唯一一个不自返，即 NaN == NaN, NaN === NaN 均为false
可用isNaN方法检测，but
``` js
    var a = 'foo';
    window.isNaN(a);    // true
    //显然  a 不是 NaN
    //推荐使用ES6中的 NUmber.isNaN()
```

## 值 复制 和 引用
简单值(包括 null、undefined、字符串、数字、布尔和 symbol)总是通过值复制(value-copy)的方式来赋值/传递；
复合型(对象，包括数组、函数、封装对象)总是通过引用复制(reference-copy)的方式来赋值/传递。

我们无法自行决定使用值复制还是引用复制（且二者在语法上没有区别），一切由值的类型决定。
``` js
    var a = 2;
    var b = a; // b 是 a 的值的一个副本(值复制)
    b++;
    a; // 2
    b; // 3

    var c = [1, 2, 3];
    var d = c; // d 是 [1, 2, 3] 的一个引用(引用复制)
    d.push(4);
    c; // [1, 2, 3, 4]
    d: // [1, 2, 3, 4]
```
再看一个例子
``` js
    function foo(x) {
        x.push(4);
        x; // [1, 2, 3, 4]

        x = [4, 5, 6];
        x.push(7);
        x; // [4, 5, 6, 7];
    }

    var a = [1, 2, 3];
    foo(a);
    a; // [1, 2, 3, 4] 不是 [4,5,6,7]

    // x 只会改变 a 和 x 共同指向的值，而不会改变 a 的指向，当 x 被赋新值时，指向发生了改变。

    // 如果想要通过值复制的方式传递复合值，就需要为它创建一个副本，这样子传递的不再是原始值
    // 可以通过 slice() 做一个浅副本

```

## 基本类型 和 封装对象
思考一个问题，var a = 'aaa'; 和 var b = new String('aaa') 和 var c = String('aaa') 有什么区别；
其实 var a = 'aaa' 和 var c = String('aaa') 是一样的东西， 即 a === c
这里主要是区别 a 和 b
1. a 是基本类型 String
2. b 是对象 Object 创建的是字符串'aaa'的封装对象，而非基本类型值'aaa'

主要说一下 var a = 'aaa'；
由于基本类型值没有 .toString(),.length 这样的方法和属性， 所以JS引擎自动为a（基本类型值）包装一个 '封装对象'
注意 a 是js引擎自动包装的封装对象，其本质还是一个基本类型，不能和手动封装的 b 一样，b 实际上还是一个引用类型。

有个疑问，自动包装是在访问属性方法的时候进行的 还是在 LHS 进行的？？？

