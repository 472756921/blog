---
title: '''JS中的This'''
date: 2018-05-02 11:03:48
tags:
---
## THIS在JS里面是比较'神秘'和'诡异'的一个角色，它变化莫测，喜怒无常。
    首先由一个错误的观点引出
<font color=red>this指向其所在位置的函数，this指向词法作用域</font>
这里就将js的词法作用域和动态作用域混淆了。
this指向谁完全是由JS的调用栈决定的，换句话说，this的绑定是在函数调用的时候完成的，而不是在声明的时候。
### this绑定规则
this的绑定是有由以下4个规则指定的（But有时候有例外），优先级由低到高。
    特别注意在严格模式下声明的this会受到一些影响。
#### 默认绑定
#### 隐式绑定
#### 显认绑定
#### NEW绑定


## call bind apply 三兄弟
这三个兄弟都是改变函数运行时的 this 指向
call 和 apply 相似，区别在于
#### call
接受1+N个参数：第一个参数是要绑定给this的值，后面传入的是一个参数列表；（this, a1,a2,a3,....,an）;当第一个参数为null、undefined的时候，默认指向window。
#### apply
接受两个参数：第一个参数是要绑定给this的值，第二个参数是一个参数数组；（this, []）;第一个参数为null、undefined的时候，默认指向window。
#### bind
和call很相似，第一个参数是this的指向，从第二个参数开始是接收的参数列表。区别在于bind方法返回值是函数以及bind接收的参数列表的使用。
``` js
// bind返回值是函数
var obj = {
    name: 'Dot'
}

function printName() {
    console.log(this.name)
}

var dot = printName.bind(obj)
console.log(dot) // function () { … }
dot()  // Dot

// 参数的使用
function fn(a, b, c) {
    console.log(a, b, c);
}
var fn1 = fn.bind(null, 'Dot');

fn('A', 'B', 'C');            // A B C
fn1('A', 'B', 'C');           // Dot A B
fn1('B', 'C');                // Dot B C
fn.call(null, 'Dot');      // Dot undefined undefined

```
由此可见：call 是把第二个及以后的参数作为 fn 方法的实参传进去，而 fn1 方法的实参实则是在 bind 中参数的基础上再往后排。
### 总结
call、apply和bind函数存在的区别:
bind返回对应函数, 便于稍后调用； apply, call则是立即调用。
除此外, 在 ES6 的箭头函数下, call 和 apply 将失效
