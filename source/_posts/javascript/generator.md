---
title: 异步编程-generator
author: richard
toc: true
excerpt: generator是一种控制程序原型的方式...
categories:
- javascript
- generator
---

## generator
>Generator 函数是 ES6 提供的一种异步编程解决方案

调用 Generator 函数，**返回一个遍历器对象**，代表 Generator 函数的内部指针。以后，每次调用遍历器对象的next方法，就会返回一个有着value和done两个属性的对象。value属性表示当前的内部状态的值，是yield表达式后面那个表达式的值；done属性是一个布尔值，表示是否遍历结束

**注意**：一种情况就是return

当genertator 函数内部有return 语句的时候，会直接返回{value:[return的值]，done:true},但是for of 循环的时候，是不会循环到这里的

```
function* test() {
	yield 1;
	yield 2;
	return 3;
	yield 4;
}

for (let i of test()) {
	console.log(i);// 这里将打印出1，2 
}
const p = test();
console.log(p.next()); //{ value: 1, done: false }
console.log(p.next()); //{ value: 2, done: false }
console.log(p.next()); //{ value: 3, done: true }
console.log(p.next()); //{ value: undefined, done: true }

```
## next方法的参数

我们可以在执行next的时候，传递一个参数作为上一次yield 表达式返回的值(默认是undefind),但是第一次调用next方法的时候即使传入参数也是undefind

## for of 循环
前面提到Generator 函数会返回一个迭代对象，所以我们可以用for of 来循环Generator函数

## Generator.prototype.throw()
Generator 函数返回的遍历器对象，都有一个throw方法，可以在函数体外抛出错误，然后在 Generator 函数体内捕获。
```
function* test() {
	try {
		yield;
	} catch (e) {
		console.log(e); // 打印出错误
	}
}

const p = test();
p.throw('自定义错误');

```
**注意1：**如果多次使用throw 抛出错误，只有第一次抛出错误会在generator函数内部捕捉，之后抛出的错误就需要在外部使用try catch来包裹了，如果函数内部没有try catch 那么也需要在外部去捕捉

**注意2：**另外 throw方法抛出的错误要被内部捕获，前提是必须至少执行过一次next方法。否则就会在外部捕捉

**注意3：** throw方法被捕获以后，会附带执行下一条yield表达式。也就是说，会附带执行一次next方法。
```
var gen = function* gen() {
	try {
		yield console.log('a');
	} catch (e) {
		// ...
	}
	yield console.log('b');
	yield console.log('c');
};

var g = gen();
g.next(); // a
g.throw(); // b
g.next(); // c

```

## Generator.prototype.return()

Generator 函数返回的遍历器对象，还有一个return()方法，可以返回给定的值，并且终结遍历 Generator 函数。

## generator 函数的异步应用
ES6 诞生以前，异步编程的方法，大概有下面四种。

- 回调函数
- 事件监听
- 发布/订阅
- Promise 对象