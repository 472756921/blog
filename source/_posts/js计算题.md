---
title: js计算题
date: 2018-04-26 19:08:36
tags: js计算 js面试题
---
## 回调函数的应用
```JS
    function fun(n, o) {
      console.log(o);
      return {
        fun: function(m) {
          return fun(m, n)
        }
      }
    }
    var a = fun(1).fun(2).fun(4).fun(8)
```
### 解析：
首先拆解 fun(1).fun(2).fun(4).fun(8) 这个结构，让其变成
fun(1);
fun(1).fun(2);
fun(1).fun(2).fun(4);
以此类推...
可知：
fun(1) ==> function fun(1, undefined) { console.log(undefined); return{ ... } }   故打印出 undefined。

fun(1).fun(2) ==> function(m) { return fun(m, n) }  ==> function(2) { return fun(2, 1) } 其中由于fun(1)的调用，使得 n = 1 故将打印 undefined， 1

fun(1).fun(2).fun(3) ==> function(m) { return fun(m, n) }  ==> function(3) { return fun(3, 2) } 其中由于fun(2)的调用，使得 n = 2 故将打印 undefined，1，2

以此类推可得出：//undefined,1,2,4

## 闭包作用域
```JS
    var User = {
      count: 1,
      getCount: function() {
        return this.count
      }
    }
    var fn = User.getCount
    console.log(fn());
```
### 解析：
第一眼看上去貌似应该打印 1，but就这样掉进坑里了。
首先，要知道 fn TMD只是一个函数表达式，解释出来就是 fn = getCount: function(){...}。
SO：fn() = return this.count
WTF：this.count 是个什么鬼？知不道，自然就是undefined。
大叫一声：“我不甘心，一定要让打印的内容等于1”
那我们就得对fn动手术了，搞成fn = User，console.log(fn.getCount());

在题干中，fn指向的作用于是getCount这个方法，而在手术后，fun的指向变成User，所以this的环境是不一样的。

出现this先看作用域，没有this作用域仅限于自己（什么显式绑定，隐式绑定另说）

## 原型链和作用域
```JS
        function Animal(name) {
          this.name = name;
        }

        Animal.prototype.sayName = function() {
          console.log(this.name)
        }

        function Cat(name) {
          Animal.call(this, name)
        }

        var kat = new Cat('Jim')

        kat.sayName();
```
### 解析：
首先给cat一个name，然后将cat的this指向给Animal，当调用sayName的时候惊奇的发现cat原型上并没有sayName这个方法。故输出方法未定义
除非cat原型拥有sayName，可以选择继承Animal，或者直接设置。

## 优先级
```JS
        function Foo() {
            getName = function () {
                console.log('1');
            };
            return this;
        }
        Foo.getName = function () {
            console.log('2');
        };
        Foo.prototype.getName = function () {
            console.log('3');
        };
        var getName = function () {
            console.log('4');
        };
        function getName() {
            console.log(5);
        }

        Foo.getName();
        getName();
        Foo().getName();
        getName();
        new Foo.getName();
        new Foo().getName();
        new new Foo().getName();
```
### 解析
Foo.getName();  很简单  2
getName(); 以为等于5实际等于4 原因在于函数声明优先级高于函数表达式。不明白？接着说

```JS
    function a() {
        console.log(111)
    }
    a();
    function a() {
        console.log(222)
    }
    a();
```
以上代码输出什么呢？
肯定不是 111,222
而是 222， 222
为啥呢？
因为 function a(){} 这个函数会自动提升。就是不管你这个函数啥时候调用（a()），js就先把function a(){}给准备好，所以在第一个a()调用之前，要先准备好啊function a(){}，理所当然的第二个function就覆盖了第一个function
继续看
```JS
        function a() {
            console.log(111)
        }
        a();
        var a = function() {
            console.log(222)
        }
        a();
```
这个时候这么打印呢？
答案是 111，222没有问题
那再改一下

```JS
        function a() {
            console.log(111)
        }
        var a = function() {
            console.log(222)
        }
        a();
        a();
```
这个时候就会打印出 222,222这个结果
原因很简单，JS首先把function a(){}准备好，等待调用，但是还没有等到调用后面就出现了 a = X 这个玩意，这下子就坏事儿了，a就不在是一个函数声明了，而变成了赋值表达式，自然 a()就会变成打印222了
然后是Foo().getName()，这个很简单输出 1，但是你以为他很简单的话，你就错了额。

依旧是拆分大法，Foo()和getName()。
首先执行Foo()，他返回了this，后面再执行和getName(),就是执行this.getName()。
那么this是谁呢？期初我以为this是执行的function后来发现是错误的，this正常情况下应该是谁调用就指向谁，那么调用Foo()的是Window
这里很明显this属于Window
好，Window.getName()应该调用谁呢？在通常印象中应该是调用覆盖之后的getName函数表达式，也就是输出4
但是特别注意一点，也是坑，在Foo()中，惊奇的发现getName = function() {...} 这个玩意儿他没有var声明，就意味着他变成了全局的一个函数表达式，成了Window的一个属性(由于Foo()的调用造成getName被定义，后续的window.getName都将会是它)。
故Window.getName()会打印 1

跟着getName();又是getName();如果以为是等于4的话，恭喜你，又错了
原因在于上面执行过的Foo().getName()这个鬼，上面讲了，这位选手把他自己的getName搞成了全局的getName，那你说getName()应该执行什么，输出什么？
new Foo.getName();一看有点懵逼，继续拆分大法，问题来了，怎么拆呢？
new Foo和getName() 还是 new 和 Foo.getName()
这里就要考虑  new 和 . 的优先级了，在手册里可以发现 . 优先级高于 new 所以就是 new(Foo.getName())
所以输出 2

new Foo().getName();
依据上面的模式我们推测，应该拆分成 new (Foo().getName())。如果这样子，恭喜你已经成功地掌握了上面的知识，只是我要悲凉地说，你又掉坑里面了
new Foo 和 new Foo()优先级是不一样的 new Foo() 的优先级高于 new Foo 且 与 . 是相同的。
所以这个地方就成了 new Foo() 和 getName() 按照优先级相同，先干左边的条件，执行方式就变成了  (new Foo()).getName();
(new Foo()) 又由于括号优先级高于new 所以这个地方返回了一个this，成了new this，那这个this是什么呢，肯定不是Window。
照理来说new一个对象，不应该出现返回值，但是在js实例化对象可有可没有返回值。
如果没有返回值，那么将返回一个实例化对象，和java等一样。
如果有返回值的话，就要分两种情况来看了。
1，返回的是非引用类型，如基本类型（string,number,boolean,null,undefined），这个时候就装作不认识返回值（无返回值），直接返回一个是实例化对象
2，返回值是个引用类型，则实际返回值为这个引用类型。
题中返回的是个this，而this在构造函数中本来就代表当前实例化对象（this是个引用类型，引用的自己个儿），遂最终Foo函数返回实例化对象。
好，总算理清楚了。
在Foo对象中没有getName（）这个方法，所以回去原型链上查找，就找到了打印3这个东东

同上new new Foo().getName(); => new((new Foo()).getName()) 打印3

```JS
        var length = 10;
        function fn() {
            console.log(this.length)
        };
        var obj = {
            length: 5,
            method: function (fn) {
                fn();
                arguments[0]();
                fn.call(obj, 12);
            }
        };
        obj.method(fn, 1);
```
## 总结
计算题除开算法外，最重要的就是要理解JS的词法作用域和动态作用域，要清楚地了解this在各种情况下指向
