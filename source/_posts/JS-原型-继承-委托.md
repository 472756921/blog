---
title: JS对象（原型-继承-委托）
date: 2018-04-28 16:34:55
tags: JS 原型 继承 委托
---
## 记在前面
在JS世界中，原型-继承-委托这三兄弟有点搞笑，与其一个个的拜访，倒不如一起开会，这样子我会更加理解一些。
当然，免不了要单独地简单阐述一下这三兄弟。在谈这三兄弟的时候，会时不时地引申出一些其他的概念。

##  对象
什么是对象？

    “JavaScript中万物皆为对象”，但这句话并不正确，准确地讲：应该是“任何非基本类型的都是对象类型”。

可参考<a>https://segmentfault.com/a/1190000012037062</a>

对象的创建有两种方法

``` JS
    //1
    var obj = {
        name: 'Benson',
        age: '66'
    }
    //2
    var obj2 = new Object();
```
对象，就先了解到这儿

## 原型
想了很久，实在无法用一句准确描述原型，我只能说。他是一个概念。如果非要给原型加上一个定义的话，那么就是，‘原型是一个对象’。
刚刚提到，‘原型’是一个概念，是对JS中一部分特殊对象的一个描述。

    原型是用来描述一个对象内部的[[prototype]]属性所对应对象的对象。

有点绕，全是对象，我们慢慢缩句
1. 原型是对象
2. 原型是用来描述XXX的对象
3. XXX是一个对象
4. XXX是一个对象的[[prototype]]属性对应的对象

## [[prototype]]
[[prototype]]这玩意儿不能直接访问（不能直接console.log()），但是Firefox和Chrome中提供了__proto__这个非标准（不是所有浏览器都支持）的对象（理解成访问器会更好一点儿）。这样子我们就可以愉快地观察[[prototype]]这个家伙了。

每个对象创建的时候都会被赋予一个[[prototype]]，并且还是非空值。

    那么这个家伙能干什么用呢？

都知道，对象都有属性，几遍是JS中的‘对象’也不例外，有对象，自然就有属性。
当我们在执行 . 操作时（[[Get]]），第一步首先检查对象自有属性，如果没有找到，就会去[[prototype]]链中去查找。
那么，到底[[prototype]]的值是什么呢？

    在创建对象时，[[prototype]]就会被赋予一个非空的值，这个值就是默认的Object.prototype。

由于[[prototype]]存在链式调用的情况（继承），[[prototype]]的值可能会发生变化，这个时候就要仔细地判断了。

### <font color=red>特别申明：机械的认为 [[prototype]] 指向的是他的父类，这个观点是及其错误的</font>
这个我们放在后面继承里面慢慢道来
原型写到这里我就打算停下来了，毕竟我只是为了搞清楚三兄弟的关系，没必要扒他的祖宗十八代。

## 继承
在这个部分就会涉及到对[[prototype]]的各种尝试。
首先来看一个原型继承的栗子

``` JS
    function User() {
        this.name = name;
        this.age = age;
    }
    p.prototype = {
        print: function () { console.log(this.name, this.age); }
    };

    var Benson = new User('Benson', 20);
    Benson.print(); // Benson 20
```
由上面的栗子我们可以看到，继承就意味着儿子拥有了老子的一些东西，正所谓：“老子的是我的，我的还是我的”。

    当查找一个对象的属性时，JavaScript 会向上遍历原型链，直到找到给定名称的属性为止。

那么这是不是就意味着，儿子的[[prototype]]指向老子，就是继承了呢？可以暂且这样理解。
关于继承的方式可以参考<a>https://www.cnblogs.com/humin/p/4556820.html</a>
这里我不想讲继承的种种，我只想搞清楚，在继承的过程中儿子的原型链（[[prototype]]）发生了什么事情。
继承的方式有好几种，我们最常用的就是通过 new 关键字。但就是这个 new 关键字，会给我带来一系列的尴尬和困扰。

看一个栗子
``` JS
    function User() {
        this.name = name;
        this.age = age;
    }
    var Benson = new User('Benson', 20);
    console.log(Benson.__proto__.constructor); // User()
```
这里很正常，Benson.__proto__.constructor是指向User的。

    解释一下__proto__.constructor，我们知道__proto__（[[prototype]]）会在function声明的时候就创建，
    它里面会包含一个constructor属性，这个属性引用的是对象关联的函数。

看到这里，是不是就可以认为：由于使用了new，所以Benson.__proto__.constructor获得了User()，而且看起来，User()是一个构造函数。

    可以很负责地告诉你，这个看法是错误的。

让我来详细解释一下原因。
首先，JS中并不存在所谓的 ‘构造函数’。
在通常情况下 User() 都只是一个普通的不能再普通的函数， 但是由于 new 的存在，造成了User()的调用方式变成了 “构造函数调用”。
new 会劫持普通函数的调用，并且用构造函数的方式来调用这个函数。
所以将 ‘构造函数’ 称之为 ‘构造函数的调用’ 更为恰当。
不相信的话可以在User()中添加一句console.log（123）。结果就一目了然了。

还是觉得不可思议？
好，接着举栗子
``` JS
    function Foo() { ... }
    Foo.prototype = { ... }

    var a1 = new Foo();
    a1.constructor === Foo //false
    a1.constructor === object //true
```
依据 “constructor是由XXX构造”的错误理论来讲，new Foo() 相当于构造了一个对象并赋值给 a1 。
那么这个时候 a1.constructor应该指向Foo()才对，但事情并没有这样子发展。
#### 原因
在上上面的栗子中，Benson.constructor === user() 是千真万确的，问题就在这里，Benson.constructor是对User()的引用吗？

    肯定不是

事实上在 Benson 继承 User 的时候，Benson.constructor 已经被 Benson.[[prototype]] 委托给了 User.prototype 而 User.prototype.constructor 默认指向的是 User

    由此引出，继承的准确定义：子类的[[prototype]] === 父类的 prototype。(解决上面关于继承定义买下的包袱)

在上面的栗子中，由于重新的定义了 Foo.prototype ，里面没有包含 constructor ，所以 Benson.[[prototype]].constructor 就没有东西， 怎么办呢？ 通过原型链继续往上找，于是乎找到了Foo.[[prototype]]中的 constructor ，这个 constructor此时是指向 object 的
<font color=red>[[prototype]]机制就是指定对象中的一个内部链接引用另一个对象。
如果第一个对象上没有找到需要的属性/方法，就会在[[prototype]]关联的对象上进行查找，如果也没有找到会继续找他的[[prototype]]，以此类推。
这一系列对象的链接，就叫原型链。
</font>

    原型链的本质就是 对象之间的关联关系。 而这种关系的维护并非上面提及的 ‘继承’和‘类’，是通过 委托行为 来进行的。

委托行为，意味着某些对象在找不到属性/方法的时候，会把这个请求委托给另一个对象。

### 干货

    [[prototype]] == __proto__   默认指向构造函数调用的 prototype。

(注意：所有函数对象的[[prototype]]都指向Function.prototype，它是一个空函数;所有对象的 [[prototype]] 都指向其构造器的 prototype)

    prototype  指向构造函数的原型对象（constructor指向对象）

原型对象（Person.prototype）是 构造函数（Person）的一个实例(除开Function.prototype 它是函数对象，不是原型对象)。
```JS
var Y = new Person();
Person.prototype = Y;
//这个Y其实就是一个普通对象而已。
```

我们创建一个函数A(就是声明一个函数), 那么浏览器就会在内存中创建一个对象B，而且每个函数都默认会有一个属性 prototype 指向了这个对象( 即：prototype的属性的值是这个对象 )。
这个对象B就是函数A的原型对象，简称函数的原型。
这个原型对象B 默认会有一个属性 constructor 指向了这个函数A ( 意思就是说：constructor属性的值是函数A )。

![原型对象](http://o7cqr8cfk.bkt.clouddn.com/public/16-11-10/43031030.jpg)
当把一个函数作为构造函数（使用new）来创建对象的时候，那么这个被创建的对象就会存在一个默认的不可见的属性，来指向了构造函数（函数A）的原型对象（上面提到的对象B）， 这个不可见的属性我们一般用 [[prototype]] 来表示。
![原型对象](http://o7cqr8cfk.bkt.clouddn.com/public/16-11-10/6663492.jpg)

B的[[prototype]]则指向 Object.prototype 又 Object.prototype.[[prototype]] === null。
Object.[[prototype]] 指向 Function.prototype

一个声明函数的原型对象(prototype）指向的是 Object, 它的[[prototype]]指向的是Function.prototype;

## 先写这么多吧
