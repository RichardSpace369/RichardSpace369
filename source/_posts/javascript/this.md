---
title: this
author: richard
toc: true
excerpt: 日常我们在开发中，经常使用this,所以需要掌握的它原理...
categories:
- javascript
- this
---

## 为什么要用this

我们思考如下代码，就是利用this来实现的

```
function identify() {
	return this.name.toUpperCase();
}
function speak() {
	var greeting = 'hello ,I am ' + identify.call(this);
	console.log(greeting);
}
var me = {
	name: 'Kyle',
};
var you = {
	name: 'reader',
};
identify.call(me);
identify.you(me);
speak.call(me);
speak.call(you);
```
这段代码可以在不同的上下文对象（me和you）中重复使用函数identify()和speak()，不用针对每个对象编写不同版本的函数。如果不使用this，那就需要给identify()和speak()显式传入一个上下文对象

**this提供了一种更优雅的方式来隐式“传递”一个对象引用，因此可以将API设计得更加简洁并且易于复用**
##  this 到底是什么

**this是在运行时进行绑定的**，并不是在编写时绑定，它的上下文取决于函数调用时的各种条件。this的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式。

当一个函数被调用时，会创建**一个活动记录（有时候也称为执行上下文）。这个记录会包含函数在哪里被调用（调用栈）、函数的调用方式、传入的参数等信息。this就是这个记录的一个属性，会在函数执行的过程中用到**

## this 全面解析

### 绑定规则

#### 默认绑定

独立函数调用

```
function foo(){
    console.log(this.a)
}
var a=2;
foo()// 2
```
因为在本例中，函数调用时应用了this的默认绑定，因此this指向全局对象。


**如果使用严格模式，则不能将全局对象应用于默认绑定，this会绑定到undefind**

那么我们怎么知道这里应用了默认绑定呢？可以通过分析调用位置来看看foo()是如何调用的。在代码中，foo()是直接使用不带任何修饰的函数引用进行调用的，因此只能使用默认绑定，无法应用其他规则

```
function foo(){
    "use strict"
    console.log(this.a)// this 指向undefind
}
var a=2;
foo()// 报错
```

这里有一个微妙但是非常重要的细节，**虽然this的绑定规则完全取决于调用位置，但是只有foo()运行在非strict mode下时，默认绑定才能绑定到全局对象；在严格模式下调用foo()则不影响默认绑定**

```
function foo(){
    console.log(this.a)// this 指向undefind
}
var a=2;
(function(){
  "use strict"
   foo()
})()

```
#### 隐式绑定

当函数引用有上下文对象时，隐式绑定规则会把函数调用中的this绑定到这个上下文对象

```
function foo(){
    console.log(this.a)
}
var obj={
    a:2,
    foo,
}
obj.foo()//2
```

**对象属性引用链中只有上一层或者说最后一层在调用位置中起作用**

```
function foo(){
    console.log(this.a)
}
var obj2={
    a:42,
    foo,
}
var obj1={
    a:2,
    obj2:obj2
}
obj1.obj2.foo() //2
```

#### 隐式丢失

1. 一个最常见的this绑定问题就是被隐式绑定的函数会丢失绑定对象，也就是说它会应用默认绑定，从而把this绑定到全局对象或者undefined上，取决于是否是严格模式。
```
function foo() {
	console.log(this.a);
}
var obj = {
	a: 2,
	foo,
};
var bar = obj.foo;
var a = 'oop';
bar(); //"oop"

```

2. 一种更微妙、更常见并且更出乎意料的情况发生在传入回调函数时
   
```
function foo() {
	console.log(this.a);
}
function doFoo(fn) {
	fn();
}
var obj = {
	a: 2,
	foo,
};
var a = 'oop';
doFoo(obj.foo); // "oop"

```
3. 调用回调函数的函数可能会修改this

在一些流行的JavaScript库中事件处理器常会把回调函数的this强制绑定到触发事件的DOM元素上

#### 显示绑定

JavaScript提供的绝大多数函数以及你自己创建的所有函数都可以使用call(..)和apply(..)方法。

它们的第一个参数是一个对象，是给this准备的，接着在调用函数时将其绑定到this。因为你可以直接指定this的绑定对象，因此我们称之为显式绑定。


**注意：**
如果你传入了一个原始值（字符串类型、布尔类型或者数字类型）来当作this的绑定对象，这个原始值会被转换成它的对象形式（也就是new String(..)、newBoolean(..)或者new Number(..)）。这通常被称为“装箱”


可惜，显式绑定仍然无法解决我们之前提出的丢失绑定问题。


1.  硬绑定

```
function foo(something){
    console.log(this.a,something)
    return this.a+something
}
var obj={
    a:2
}
var bar=function(){
    return foo.apply(obj,arguments)// 硬绑定
}
var b=bar(3)
console.log(b)//5

```

2.  API调用的“上下文”

第三方库的许多函数，以及JavaScript语言和宿主环境中许多新的内置函数，都提供了一个可选的参数，通常被称为“上下文”（context），其作用和bind(..)一样，确保你的回调函数使用指定的this。

#### new绑定

使用new来调用函数，或者说发生构造函数调用时，会自动执行下面的操作。

1. 创建（或者说构造）一个全新的对象。
2. 这个新对象会被执行[[Prototype]]连接。
3. 这个新对象会绑定到函数调用的this。
4. 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。


### 优先级
1. 显式绑定优先级更高，也就是说在判断时应当先考虑是否可以存在显式
2. new和显示绑定，new 的优先级更高


### 绑定例外

#### 被忽略的this

如果你把null或者undefined作为this的绑定对象传入call、apply或者bind，这些值在调用时会被忽略，实际应用的是默认绑定规则


然而，总是使用null来忽略this绑定可能产生一些副作用。如果某个函数确实使用了this（比如第三方库中的一个函数），那默认绑定规则会把this绑定到全局对象（在浏览器中这个对象是window），这将导致不可预计的后果（比如修改全局对象）。

#### 更安全的this

一种“更安全”的做法是传入一个特殊的对象(空对象)，把this绑定到这个对象不会对你的程序产生任何副作用。


在JavaScript中创建一个空对象最简单的方法都是Object.create(null)

#### 间接引用
另一个需要注意的是，你有可能（有意或者无意地）创建一个函数的“间接引用”，在这种情况下，调用这个函数会应用默认绑定规则。
```
function foo(){
    console.log(this.a)
}
var a=2;
var o={
    a:3,
    foo,
}
var p={
    a:4
}
o.foo()//3
(p.foo=o.foo)()//2
```
赋值表达式p.foo = o.foo的返回值是目标函数的引用，因此调用位置是foo()而不是p.foo()或者o.foo()。根据我们之前说过的，这里会应用默认绑定。

**注意**：对于默认绑定来说，决定this绑定对象的并不是调用位置是否处于严格模式，而是函数体是否处于严格模式。如果函数体处于严格模式，this会被绑定到undefined，否则this会被绑定到全局对象。

### 箭头函数

箭头函数不使用this的四种标准规则，而是根据外层（函数或者全局）作用域来决定this。

箭头函数的绑定无法被修改。（new也不行！）