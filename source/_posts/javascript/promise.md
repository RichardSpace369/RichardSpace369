---
title: 异步编程-promise
author: richard
toc: true
excerpt: promise 是一种回调的语法糖，它仅仅是书写上是一个同步的..
categories:
- javascript
- promise
---

## 理解promise
>Promise 对象用于表示一个异步操作的最终完成 (或失败)及其结果值。

### Promise 如何解决回调地狱
回调地狱有两个主要的问题：

1. 多层嵌套的问题；

2. 每种任务的处理结果存在两种可能性（成功或失败），那么需要在每种任务执行结束后分别处理这两种可能性。

Promise 的诞生就是为了解决这两个问题。Promise 利用了三大技术手段来解决回调地狱：**回调函数延迟绑定、返回值穿透、错误冒泡**

#### 延迟绑定

思考如下代码

```
let readFilePromise = filename => {
  return new Promise((resolve, reject) => {
    fs.readFile(filename, (err, data) => {
      if (err) {
        reject(err)
      } else {
        resolve(data)
      }
    })
  })
}
// 这里就是回调函数延迟绑定，先执行，再通过then 来绑定回调函数
readFilePromise('1.json').then(data => {
  return readFilePromise('2.json')
});
```
#### 返回值穿透

我们 将上面的代码改成下面这样

```
let x = readFilePromise('1.json').then(data => {
  return readFilePromise('2.json')  //这是返回的Promise
});
x.then(/* 内部逻辑省略 */)
```

我们根据 then 中回调函数的传入值创建不同类型的 Promise，然后把返回的 Promise 穿透到外层，以供后续的调用。这里的 x 指的就是内部返回的 Promise，然后在 x 后面可以依次完成链式调用。这便是返回值穿透的效果，这两种技术一起作用便可以将深层的嵌套回调写成下面的形式。

#### 错误冒泡

上面的方式解决了多层嵌套的问题，那另外一个问题，即每次任务执行结束后分别处理成功和失败的情况怎么解决的呢？Promise 采用了错误冒泡的方式。其实很容易理解，我们来看看效果。

```
readFilePromise('1.json').then(data => {
    return readFilePromise('2.json');
}).then(data => {
    return readFilePromise('3.json');
}).then(data => {
    return readFilePromise('4.json');
}).catch(err => {
  // xxx
})

```
这样前面产生的错误会一直向后传递，被 catch 接收到，就不用频繁地检查错误了。从上面的这些代码中可以看到，Promise 解决效果也比较明显：实现链式调用，解决多层嵌套问题；实现错误冒泡后一站式处理，解决每次任务中判断错误、增加代码混乱度的问题

## Promise 的原型上的方法

###  Promise.prototype.then(onFulfilled, onRejected)
>添加解决(fulfillment)和拒绝(rejection)回调到当前 promise, 返回一个新的 promise, 将以回调的返回值来resolve.

接收参数：

这个方法，将在期约函数执行完毕之后执行，这个方法可以接收两个参数都不是必须的，第一个参数是用于处理 onResovled 情况，其接收的参数是 promise 内部调用 resolve 函数传递的参数，第二个参数是处理 onRejected 情况。其接收的参数是 promise 内部调用 reject 函数的参数。这个方法返回一个新的期约对象

如果在调用这个参数的时候传递了一个非函数类型的值，会直接忽略这个值,例如

```
const p = Promise.resolve(1);
p.then(2).then((x) => {
	console.log(x); // 接收的是1
});

```

如果接收的函数返回一个期约对象将原封不动的返回这个期约对象

```
const p = Promise.resolve(1);
p.then(() => Promise.resolve(2)).then((x) => {
	console.log(x); // 接收的是2
});
这个函数的then 方法接收一个返回期约对象的函数
```

如果接收的函数返回的是其他数据，那么程序将会调用 Promise.resolve 包装这个返回的值，如果没有显示的返回语句，将使用 Promise.resolve 包装一个 undefind 返回

如果**接收的函数抛出错误**，那么就会返回拒绝的期约即使用 Promise.reject 进行包裹错误

```
const p = Promise.resolve(1);
p.then(() => {
	throw new Error('p 产生了错误');
})
	.then((x) => {
		console.log(x); // 接收的是2
	})
	.catch((error) => {
		console.error('error:', error);
	});

```

如果**接收的函数返回错误，而不是抛出错误**，那么将继续使用 Promise.resolve 包裹这个返回的对象

```
let p11 = p1.then(() => Error('qux'));
```

需要注意的一点是：then 方法的第二个参数是代表处理 onRejected 的情况，这个函数返回的值同样也会被 Promise.resolve 进行包裹

```
const p = Promise.reject('错误');
p.then(null, () => {
	return 'onRejected 返回的值';
}).then((x) => {
	console.log(x); // 'onRejected 返回的值';
});

```

###  Promise.prototype.catch()

Promise.prototype.catch()方法用于给期约添加拒绝处理程序。这个方法只接收一个参数：onRejected 处理程序。事实上，这个方法就是一个语法糖，调用它就相当于调用 Promise.prototype.then(null, onRejected)

### Promise.prototype.finally

无论是 rejected 状态还是 resolved 状态都会执行

## Promise 的静态方法

### Promise.all(iterable) 

这个方法返回一个新的promise对象，该promise对象在iterable参数对象里所有的promise对象都成功的时候才会触发成功，一旦有任何一个iterable里面的promise对象失败则立即触发该promise对象的失败。这个新的promise对象在触发成功状态以后，会把一个包含iterable里所有promise返回值的数组作为成功回调的返回值，顺序跟iterable的顺序保持一致；如果这个新的promise对象触发了失败状态，它会把iterable里第一个触发失败的promise对象的错误信息作为它的失败错误信息。Promise.all方法常被用于处理多个promise对象的状态集合

```
const promiseSuccess = new Promise((resolve, reject) => {
	setTimeout(resolve, 1000, 1);
});
const promiseFail = new Promise((resolve, reject) => {
	setTimeout(reject, 1000, 'error');
});
// 全部执行成功将会在then里面第一个参数中返回一个数组
Promise.all([promiseSuccess, promiseSuccess]).then((res) => {
	console.log(res); //[1,1]
});
// 有一个执行失败则触发reject
Promise.all([promiseSuccess, promiseFail])
	.then((res) => {
		console.log(res); //[1,1]
	})
	.catch((error) => {
		console.log(error);
	});
```
**注意1**:如果传入得是参数不包含任何promise，则返回一个异步完成
```
const test = Promise.all([1, 2, 3]).then((result) => {
	console.log('result', result); //[1,2,3]
});
console.log(test); // promise pending 状态

```
**注意2:**如果接收得是一个空的可迭代得对象则返回一个已完成得同步状态(注意是同步状态)
```
const test = Promise.all([]);
console.log(test); // Promise { [] }
```

**注意3：**，如果作为参数的 Promise 实例，自己定义了 catch 方法，那么它一旦被 rejected，并不会触发 Promise.all()的 catch 方法。

```
const p1 = new Promise((resolve, reject) => {
  resolve('hello');
})
.then(result => result)
.catch(e => e);

const p2 = new Promise((resolve, reject) => {
  throw new Error('报错了');
})
.then(result => result)
.catch(e => e);

Promise.all([p1, p2])
.then(result => console.log(result))
.catch(e => console.log(e));
// ["hello", Error: 报错了]
```
###  Promise.race 方法

###  Promise.resolve 方法

promise.resolve 接收的参数有多种情况

1. 如果接收的是一个 Promise 实例，则原封不动的返回这个实例
2. 如果接收的参数是一个 thenable 对象,Promise.resolve()方法会将这个对象转为 Promise 对象，然后就立即执行thenable对象的then()方法。
3. 如果参数是一个原始值，或者是一个不具有 then()方法的对象，则 Promise.resolve()方法返回一个新的 Promise 对象，状态为 resolved。
4. Promise.resolve()方法允许调用时不带参数，直接返回一个 resolved 状态的 Promise 对象。

另外要注意的是 Promise.resolve 和 Promise.reject 总是异步执行的

```
try {
	throw Error();
} catch (err) {
	console.log('err1', err); // 打印
}
try {
	Promise.reject(Error()); // 直接报错，这个错误捕捉不到
} catch (err) {
	console.log('err2', err); // 未打印
}

```

###  Promise.reject

###  Promise.allSettled;

等到所有promises都已敲定（settled）（每个promise都已兑现（fulfilled）或已拒绝（rejected））。返回一个promise，该promise在所有promise完成后完成。并带有一个对象数组，每个对象对应每个promise的结果。



该方法返回的新的 Promise 实例，一旦结束，状态总是 fulfilled，不会变成 rejected。状态变成 fulfilled 后，Promise 的监听函数接收到的参数是一个数组，每个成员对应一个传入 Promise.allSettled()的 Promise 实例。

Promise.allSettled()的返回值 allSettledPromise，状态只可能变成 fulfilled。它的监听函数接收到的参数是数组 results。该数组的每个成员**都是一个对象**，对应传入 Promise.allSettled()的两个 Promise 实例。每个对象都有 status 属性，该属性的值只可能是字符串 fulfilled 或字符串 rejected。fulfilled 时，对象有 value 属性，rejected 时有 reason 属性，对应两种状态的返回值

**接收参数得情况和promise.all 是保持一致得**


**注意**：catch函数永远不会执行，即使抛出错误
```
const promise1 = Promise.resolve(3);
const promise2 = new Promise((resolve, reject) =>
	setTimeout(reject, 100, 'foo')
);
const promises = [promise1, promise2];

Promise.allSettled(promises)
	.then((results) => {
		console.log(results); //[{ status: 'fulfilled', value: 3 },{ status: 'rejected', reason: 'foo' }]
	})
	.catch((error) => {
		console.log(error); // 不会执行
	});

```
### Promise.any

该方法接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例返回。只要参数实例有一个变成 fulfilled 状态，包装实例就会变成 fulfilled 状态；如果所有参数实例都变成 rejected 状态，包装实例就会变成 rejected 状态。

### Promise.try


不知道或者不想区分，函数f是同步函数还是异步操作，但是想用 Promise 来处理它。因为这样就可以不管f是否包含异步操作，都用then方法指定下一步流程，用catch方法处理f抛出的错误

## Promise 的缺点以及解决方案

期约不具备的两个特性(期约取消和状态进度)

ES6 不支持取消期约和进度通知，一个主要原因就是这样会导致期约连锁和期约合成
过度复杂化。比如在一个期约连锁中，如果某个被其他期约依赖的期约被取消了或者发出了
通知，那么接下来应该发生什么完全说不清楚。毕竟，如果取消了 Promise.all()中的一个
期约，或者期约连锁中前面的期约发送了一个通知，那么接下来应该怎么办才比较合理呢？

###  期约取消

1. 使用bluebird 提供的promise来做
2. 使用原生promise增加cancle状态来实现

```
function updateUser() {
	let resolveFn,
		rejectFn,
		cancled = false;
	// 延迟返回状态
	const promise = new Promise((resolve, reject) => {
		resolveFn = resolve;
		reject = rejectFn;
	});
	//在这里真正执行函数和判断是否已经取消
	function wrapWithCancel(fn) {
		return (data) => {
			if (!cancled) {
				fn(data);
			}
		};
	}
	fetchUser()
		.then(wrapWithCancel(updateUserData))
		.then(wrapWithCancel(updateUserAddress))
		.then(wrapWithCancel(updateMarketingData))
		.then(resolveFn)
		.catch(rejectFn);
	return {
		promise,
		cancle: () => {
			cancled = true;
			rejectFn({ reson: '取消' });
		},
	};
}
const { promise, cancle } = updateUser();
// later some time
cancle();// 执行取消操作

```
### 状态进度



## promise面试题

### 实现一个串行调用的promise

```
function start(tasks) {
	var result = [];
	return tasks.reduce((accumulator, item, index) => {
		return accumulator.then((res) => {
			result.push(res);
			return index == tasks.length - 1
				? item.then((res) => {
						result.push(res);
						return result;
				  })
				: item;
		});
	}, Promise.resolve(0));
}
function delay(time) {
	return new Promise(function(resolve, reject) {
		setTimeout(function() {
			resolve(time);
		}, time);
	});
}

start([delay(2000), delay(2000), delay(1000)]).then((res) => {
	console.log(res); //[undefined,undefined,1000]
});

```
### 实现多个Promise 异步执行并拿到返回结果

```
function execter(arr) {
    return new Promise((resolve) => {
        let result = [];
        function setData(data) {
            result.push(data);
            if (arr.length === result.length) {
                resolve(result);
            }
        }
        arr.forEach(promise => {
            promise
                .then(data => {
                    setData(data);
                })
                .catch(err => {
                    setData(null);
                });
        });
    });
}
```
### 使用Promise实现每隔1秒输出1,2,3

注意这个方案是直接创建了三个settimeout函数然后设置了不同的时间而已

```
function outpUT(arr) {
	arr.reduce((pre, next) => {
		return pre.then(() => {
			setTimeout(console.log, next * 1000, next);
		});
	}, Promise.resolve());
}
outpUT([1, 2, 3]);

```

### 使用Promise实现红绿灯交替重复亮

>红灯3秒亮一次，黄灯2秒亮一次，绿灯1秒亮一次；如何让三个灯不断交替重复亮灯？（用Promise实现）三个亮灯函数已经存在：

```
function red() {
	console.log('red');
}
function green() {
	console.log('green');
}
function yellow() {
	console.log('yellow');
}
/**
 * @description 创建真正执行打印的函数
 * @param {*} time
 * @param {*} fn
 * @returns
 */
function light(time, fn) {
	return new Promise((resolve) => {
		setTimeout(() => {
			fn();
			resolve();
		}, time);
	});
}
/**
 * @description 执行函数，依次执行红黄绿，然后递归调用
 * @returns
 */
function step() {
	return Promise.resolve()
		.then(() => {
			return light(3000, red);
		})
		.then(() => {
			return light(2000, yellow);
		})
		.then(() => {
			return light(1000, green);
		})
		.then(() => {
			return step();
		});
}
step();

```
### 限制异步操作的并发个数并尽可能快的完成全部

有8个图片资源的url，已经存储在数组urls中。
urls类似于['https://image1.png', 'https://image2.png', ....]
而且已经有一个函数function loadImg，输入一个url链接，返回一个Promise，该Promise在图片下载完成的时候resolve，下载失败则reject。
但有一个要求，任何时刻同时下载的链接数量不可以超过3个。
请写一段代码实现这个需求，要求尽可能快速地将所有图片下载完成。

实现方法如下：
1. 先取出Limit(假设是3)个元素来形成一个数组，并发出请求
2. 迭代length-limit个元素，在迭代的每一次中，拿到上面最快执行完毕的下标，并且用下一个元素来替代这个下标的位置
3. 最后执行(promise.all)剩下的三个元素

这个方案：有个问题就是假设我们需要拿到执行的请求结果那么这个方法就是有缺陷的，可以思考一下


改进思路：就是建立一个Map然后改造hander，然后在Handler里面去设置


```
var urls = [
			'https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/AboutMe-painting1.png',
			'https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/AboutMe-painting2.png',
			'https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/AboutMe-painting3.png',
			'https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/AboutMe-painting4.png',
			'https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/AboutMe-painting5.png',
			'https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/bpmn6.png',
			'https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/bpmn7.png',
			'https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/bpmn8.png',
		];
		function loadImg(url) {
			return new Promise((resolve, reject) => {
				const img = new Image();
				img.onload = function() {
					console.log('一张图片加载完成');
					resolve(img);
				};
				img.onerror = function() {
					reject(new Error('Could not load image at' + url));
				};
				img.src = url;
			});
		}
		/**
		 * @ 思路：第一次先一次执行limit个，然后在里面每个执行完毕了再从url中取一个出来去执行
		 * @param {*} urls
		 * @param {*} limit
		 */
		function limitLoad(urls, handler, limit) {
			// 赋值URLs
			let sequence = [].concat(urls);
			// 先取前三个元素发出请求，但是这里是没有处理请求结果的
			let promises = sequence.splice(0, limit).map((url, index) => {
				return handler(url).then(() => {
					return index; // 注意：返回的是下标元素
				});
			});
			// 迭代剩下的未发出请求的图片
			return (
				sequence
					.reduce((pCollect, url) => {
						return pCollect
							.then(() => {
								// 使用promise race 是保证尽快拿到已经请求结束的图片的下标
								return Promise.race(promises);
							})
							.then((fastestIndex) => {
								// 拿到前面请求最快的元素下标然后发出这个请求，并且让这个请求占据原来的位置
								promises[fastestIndex] = handler(url).then(() => {
									return fastestIndex; // 要继续将这个下标返回，以便下一次变量
								});
							})
							.catch((err) => {
								console.error(err);
							});
					}, Promise.resolve()) // 初始化传入
					// 因为上面最多只调用7次，但是已经补充了请求到promises里面，则把剩余的执行完毕
					.then(() => {
						// 最后三个用.all来调用
						return Promise.all(promises);
					})
			);
		}
		limitLoad(urls, loadImg, 3)
			.then((res) => {
				console.log('图片全部加载完毕');
				console.log(res);
			})
			.catch((err) => {
				console.error(err);
			});

```

### 手动实现一个promise