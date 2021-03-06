---
layout: post
title: "JavaScript Functional Programming(II)"
date: 2016-07-20 17:26:20 +0800
author: Yongming
tags:
    - JavaScript
    - functional programming
    
---


##纯函数式编程

#### **纯函数的概念**
> 纯函数是这样一种函数,即相同的输入,永远会得到相同的输出,而且没有任何可观察的副作用

比如``slice``和``splice``,这两个函数的作用并无二致,但是,他们各自的方式却大不相同,但不管怎么说作用还是一样的。
``slice``符合纯函数的定义是因为对相同对输入它保证能返回相同的输出。而``splice``却会嚼烂调用它的那个数组,然后在吐出来,这样就会产生可观察到的副作用,即这个数组永远改变了。

```javascript
var xs = [1,2,3,4,5];

//纯函数
xs.slice(0,3) =>[1,2,3]
xs.slice(0,3) =>[1,2,3]
xs.slice(0,3) =>[1,2,3]

//非纯函数
xs.splice(0,3) =>[1,2,3]
xs.splice(0,3) =>[4,5]
xs.splice(0,3) =>[]
```

在函数式编程中,我们避免这种改变数据的方式。我们追求的是可靠的,每次都能返回同样结果的函数。

看另一个例子

```javascript

//不纯的
var minimum = 21
var checkAge = function(age) {

    return age >= minimum;
};


//纯函数
var checkAge = function(age) {
    
    var minimum = 21;
    return age >= minimum;
    
}
```

在不纯的版本中,``checkAge``的结果取决于``minimum``这个可变量的值。换句话说,它取决于系统状态(system status),这一点很不好,它引入了外部环境,从而增加了认知负荷(cognitive load)。

另一方面,使用纯函数的形式,就要做到自给自足。我们也可以让``minimum``成为一个不可变量。这样就能保证纯粹性,因为状态不会发生变化。要实现这个效果,我们就必须传教一个对象,然后调用``Object.freeze``方法:

```
var immutableState = Object.freeze({
    minimum:21
})
```

####副作用
>副作用是在计算结果的过程中,系统状态的一种变化,或者与外部世界进行的可观察交互
它包括:
- 更改文件系统
- 往数据库中插入记录
- 发送一个http请求
- 可变数据
- 打印/log
- 获取用户输入
- DOM查询
- 访问系统状态

概括来讲,只要函数外部环境发生交互都是副作用——这一点可能会让你怀疑无副作用编程的可行性。函数式编程的哲学就是假定副作用是造成不正当行为的主要原因。这并不是说，要禁止使用一切副作用，而是说，要让它们在可控的范围内发生。后面讲到 functor 和 monad 的时候我们会学习如何控制它们，目前还是尽量远离这些阴险的函数为好。副作用让一个函数变得不纯是有道理的：从定义上来说，纯函数必须要能够根据相同的输入返回相同的输出；如果函数需要跟外部事物打交道，那么就无法保证这一点了。我们来仔细了解下为何要坚持这种「相同输入得到相同输出」原则。注意，我们要复习一些高考数学知识了。

####*回顾下人教版教材数学必修一*

**教材中的定义:** 
> 设A，B是非空的数集，如果按照某种确定的对应关系f，使对于集合A中的任意一个数x，在集合B中都有唯一确定的数y和它对应，那么就称``f:A->B``为从集合A到集合B的一个函数，记作``y=f(x)``或$f(A)={y|f(x)=y,y\inB}$。

> 其中`x`叫作自变量, `y`叫因变量，集合`A`叫做函数的定义域，与`x`对应的`y`叫做函数值，函数值的集合`B`叫做函数的值域。
> 定义域，值域，对应法则称为函数的三要素。一般书写为  。若省略定义域，一般是指使函数有意义的集合。

 》 (未完待续)
 
 
####追求"纯"的理由

#####可缓存性（Cacheable）
     
首先，纯函数总能够根据输入来做缓存。实现缓存的一种典型方式是 memoize 技术：

```javascript
var squareNumber = memoize(function(x){return x*x;});
squareNumber(4);
//=> 16

squareNumber(4); // 从缓存中读取输入值为 4 的结果
//=> 16

squareNumber(5);
//=> 25

squareNumber(5); // 从缓存中读取输入值为 5 的结果
//=> 25
```

下面的代码是一个简单的实现，尽管它不太健壮。

```javascript
var memoize = function(f){
    var cache = {};
    
    return function(){
        var arg_str = JSON.stringify(arguments);
        cache[arg_str] = cache[arg_str] || f.apply(f,arguments);
        return cache[arg_str];
    };
};
```

值得注意的一点是,可以通过延迟执行的方法吧不纯的函数转换为纯函数

```javascript
var pureHttpCall = memize(frunction(url,params){
    return function(){return $.getJSON(url,params);}
});
```

这里有趣的地方在于我们并没有真正发送``http``请求——它只返回来一个函数,当调用它当时候才会发出请求。这个函数之所以有资格成为纯函数,是因为它总是会根据相同当输入返回相同当输出,给定``url``和``params``之后,它就会返回同一个发送``http``请求的函数。

我们的``memoize``函数工作起来没有任何问题,虽然它缓存的并不是http请求所返回的结果, 而是生成的函数。


####可移植性/自文档化(Portable/Self-Documenting)

纯函数是完全自给自足的，它需要的所有东西都能轻易获得。仔细思考思考这一点...这种自给自足的好处是什么呢？首先，纯函数的依赖很明确，因此更易于观察和理解——没有偷偷摸摸的小动作。

```javascript
//不纯的
var signUp = function(attrs){
    var user = saveUser(attrs);
    welcomeUser(user);
}

//纯的
var signUp = function(Db,Email,attrs){
    return function() {
        var user = saveUser(Db, attrs);
        welcomeUser(Email, user);
    };

}

```

这个例子表明, 纯函数对于其依赖必须要诚实,这样我们就能知道它的目的。仅从纯函数版本的``signUp``的签名就可以看出,它将要用到``Db``,``Email``和``attrs``,这在最小程度上给了我们足够多的信息。

其次,通过强迫"注入"依赖, 或者把它们当作参数传递, 我们的的应用也更加灵活, 因为数据库或者邮件客户端等等都参数化了。 如果要使用另一个``Db``, 只需要吧它传给函数就行了。 如果想在一个新应用中使用这个可以靠的函数,尽管吧新的``Db``和``Email``传递过去就好了。

在JavaScript设定中, ``portable``意味着把函数序列化(serlalizing)并通过socket发送, 也可以意味着代码能够在web workers中运行。总之,``portable``是个非常强大的特性。

命令式编程中“典型”的方法和过程都深深地根植于它们所在的环境中，通过状态、依赖和有效作用（available effects）达成；纯函数与此相反，它与环境无关，只要我们愿意，可以在任何地方运行它。

####可测试行(Testable)
纯函数让测试更加容易。我们不需要伪造一个“真实的”支付网关，或者每一次测试之前都要配置、之后都要断言状态（assert the state）。只需简单地给函数一个输入，然后断言输出就好了。

####合理性(Reasonable)
很多人相信使用纯函数最大的好处是``引用透明(referential transparency)。如果一段代码可以替换成它执行所得的结果,而且是在不改变整个程序行为下的前提下替换的,那我们说这段代码说是引用透明的。

由于纯函数总是能够根据相同的输入返回相同的输出，所以它们就能够保证总是返回同一个结果，这也就保证了引用透明性。我们来看一个例子。

```javascript
var decrementHP = function(player){
    return player.set("hp",player.hp-1);
};

var isSameTeam = function(player1,player2){
    return player1.team === player2.team; 
};

var punch = function(player,target){
    if (isSameTeam(player,target)){
        return target;
    }
    else{
        return decrementHP(target);
    }
};


var jobe = Immutable.Map({
    name:"Jobe",
    hp:20, 
    team: "red"
});

var michael = Immutable.Map({
    name: "Michael",
    hp: 20,
    team: "green"

});

punch(jobe,michael); //=>Immutable.Map({name:"Michael", hp:19, team:"green"});

```

``decrementHP``,``isSameTeam``和``punch``都是纯函数, 所以是引用透明。 我们可以使用一种叫做``等式推到``(equational reasoning)的情况下, 手动执行相关代码。我们借助引用透明性来剖析一下这段代码。

首先内联``isSameTeam``函数:

```javascript
var punch = function(player, target) {
  if(player.team === target.team) {
    return target;
  } else {
    return decrementHP(target);
  }
};
```

因为是不可变数据，我们可以直接把``team``替换为实际值：

```javascript
var punch = function(player, target) {
  if("red" === "green") {
    return target;
  } else {
    return decrementHP(target);
  }
};
```

``if``语句执行结果``false``,所以可以把整个``if``语句都删掉:

```javascript
var punch = function(player,target){
    return decrementHP(target);
};
```

如果内联``decrementHP``,我们会发现这种情况,``punch``变成来一个让``hp``的值减1的调用:

```javascript
var punch = function(player,target){
    return target.set("hp",target.hp-1);
}
```

>总之, 等式推导带来的分析代码的能力对重构和理解代码非常重要。[上一篇](2016-07-20-JavaScript Functional Programming)对傻狍子代码重构正是这项技术。

####并行代码
最后, 我们可以并行运行任意纯函数。因为纯函数根本不需要访问共享的内存,而且根据其定义, 纯函数也不会因副作用而进入竞争态。

并行代码在服务端js环境以及使用来 web worker 的浏览器哪里是非常容易实现的,因为它们使用了线程。不过出于对非纯函数复杂度的考虑，当前主流观点还是避免使用这种并行。

--------
##柯里化(curry)


> Can't live if livin' is without you

我爸爸曾经和我说有些东西直到你得到它,才会意识到他的价值。微波炉就是一个很好的例子, 智能手机也一样。老人们在没有互联网之前也过的很充实。对我来说Curry也是这样


curry这个概念很简单: 你可以只传一部分参数来调用一个函数。它会返回一个函数来处理剩下的参数

你可以选择一次性调用curry函数,或者每次只传一个参数分多次调用

```js
var add = function(x) {
  return function(y) {
    return x + y;
  };
};

var increment = add(1);
var addTen = add(10);

increment(2);
// 3

addTen(2);
// 12
```

>这里我们定义来一个``add``函数,它接受一个参数并且返回一个函数。调用``add``函数之后,这个返回的函数会记住第一个参数通过闭包。一次性调用它确实有点繁, 然而我们可以使用一个特殊的辅助函数``curry``来使这类函数定义更加容易

让我们来创建一些``curry``函数爽一把
Let's set up a few curried functions for our enjoyment.(注:"for our enjoyment"出自圣经)

```js
var curry = require('lodash/curry');

var match = curry(function(what, str) {
  return str.match(what);
});

var replace = curry(function(what, replacement, str) {
  return str.replace(what, replacement);
});

var filter = curry(function(f, ary) {
  return ary.filter(f);
});

var map = curry(function(f, ary) {
  return ary.map(f);
});
```

上面的代码我遵循一个简单的,同时也非常重要的模式。即策略性地把要操作的数据(String,Array)放到最后一个参数里。到使用它们的时候你就明白为什么这样了。

```js
match(/\s+/g, 'hello world');
// [ ' ' ]

match(/\s+/g)('hello world');
// [ ' ' ]

var hasSpaces = match(/\s+/g);
// function(x) { return x.match(/\s+/g) }

hasSpaces('hello world');
// [ ' ' ]

hasSpaces('spaceless');
// null

filter(hasSpaces, ['tori_spelling', 'tori amos']);
// ['tori amos']

var findSpaces = filter(hasSpaces);
// function(xs) { return xs.filter(function(x) { return x.match(/\s+/g) }) }

findSpaces(['tori_spelling', 'tori amos']);
// ['tori amos']

var noVowels = replace(/[aeiouy]/ig);
// function(replacement, x) { return x.replace(/[aeiouy]/ig, replacement) }

var censored = noVowels("*");
// function(x) { return x.replace(/[aeiouy]/ig, '*') }

censored('Chocolate Rain');
// 'Ch*c*l*t* R**n'
```

这里所表明的是一种``预加载``函数的能力,同传递一两个参数来接收一个记住来这些参数的新函数

我鼓励你用 `npm install lodash`, 复制上面的代码放到`repl`里run一下。以也可以在能够使用`lodash`或者`ramda`的浏览器里运行。


##不仅仅是双关语／咖喱
  
curry的用途广泛。就像在``hasSpaces``,``findSpaces``和``censored``里面看到的那样,我们传给函数一些参数就可以得到新的函数。

通过``map``函数简单把单个元素的函数包裹下,就可以转换为参数是数组的函数。

```js
var getChildren = function(x) {
  return x.childNodes;
};

var allTheChildren = map(getChildren);
```

只传给函数一部参数通常也被称作``局部调用``,能过大量减少样板文件代码(boiler plate code)。考虑上面的``allTheChilder``函数,如果用``lodash``的普通``map``来写会是什么样的(note:参数的顺序也变了)

```js
var allTheChildren = function(elements) {
  return _.map(elements, getChildren);
};
```

通常我们不定义直接操作数组的函数,因为我们只需要内联``map(getChildren)``就可以。同样适用于``sort``,``filter``,和其他高阶函数。

当我么讨论``纯函数``,我你说一个输入对应一个输出。curry正是如此,每传递一个参数就返回一个新的函数来处理剩下的参数。兄弟,那就是一个输入对应一个输出

无论输出是不是另一函数,它都是纯函数。当然curry函数也允许一次传入多个参数,但这知识处于减少``()``的方便。



## Exercises

A quick word before we start. We'll use a library called [Ramda](http://ramdajs.com) which curries every function by default. Alternatively you may choose to use [lodash/fp](https://github.com/lodash/lodash/wiki/FP-Guide) which does the same and is written/maintained by the creator of lodash. Both will work just fine and it is a matter of preference.

There are [unit tests](https://github.com/DrBoolean/mostly-adequate-guide/tree/master/code/part1_exercises) to run against your exercises as you code them, or you can just copy-paste into a javascript REPL for the early exercises if you wish.

Answers are provided with the code in the [repository for this book](https://github.com/DrBoolean/mostly-adequate-guide/tree/master/code/part1_exercises/answers). Best way to do the exercises is with an [immediate feedback loop](feedback_loop.md).

```js
var _ = require('ramda');


// Exercise 1
//==============
// Refactor to remove all arguments by partially applying the function.

var words = function(str) {
  return _.split(' ', str);
};

var words = split(' ');

// Exercise 1a
//==============
// Use map to make a new words fn that works on an array of strings.

var sentences = undefined;

var sentences = map(words)

// Exercise 2
//==============
// Refactor to remove all arguments by partially applying the functions.

var filterQs = function(xs) {
  return _.filter(function(x) {
    return match(/q/i, x);
  }, xs);
};

var filterQs = filter(match(/q/i,x));


// Exercise 3
//==============
// Use the helper function _keepHighest to refactor max to not reference any
// arguments.

// LEAVE BE:
var _keepHighest = function(x, y) {
  return x >= y ? x : y;
};

// REFACTOR THIS ONE:
var max = function(xs) {
  return _.reduce(function(acc, x) {
    return _keepHighest(acc, x);
  }, -Infinity, xs);
};

var max = reduce(_keepHighest,-Infinity)

// Bonus 1:
// ============
// Wrap array's slice to be functional and curried.
// //[1, 2, 3].slice(0, 2)
var slice = undefined;
var slice = _.curry(function(start,end,xs){
    return xs.slice(start,end)
})

// Bonus 2:
// ============
// Use slice to define a function "take" that takes n elements from the beginning of the string. Make it curried.
// // Result for "Something" with n=4 should be "Some"
var take = undefined;

var take = slice(0)

```
