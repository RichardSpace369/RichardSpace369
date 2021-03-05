---
title: js中的array
author: richard
toc: true
excerpt: js中的数据是一种特殊的对象，它具有如下方法...
categories:
- javascript
- 数组
---

## 理解数组
>数组（Array）是**有序的**元素序列。若将有限个**类型相同**的变量的集合命名，那么这个名称为数组名。组成数组的各个变量称为数组的分量，也称为数组的元素，有时也称为下标变量。用于区分数组的各个元素的数字编号称为下标

上面是百度百科对数组的解释：我们可以看出**数组在内存中是一段连续的同类型的数据空间**


### js中数组的表现了？

我们先看MDN 对数组的定义

>数组是一种类列表对象，它的原型中提供了遍历和修改元素的相关操作。JavaScript 数组的长度和元素类型都是非固定的。因为数组的长度可随时改变，并且其数据在内存中也可以不连续，所以 JavaScript 数组不一定是密集型的，这取决于它的使用方式
```
const arr=[1,2,3]
arr.name="数组"
console.log(arr.name)//"数组"
```
从上面的代码我们可以看出来并不满足上面数组的定义。 其实这是因为**js中的数组是一个特殊的对象**

##  创建数组

在js中创建一个数组一共有二种方法，分别是
1. 字面量创建
2. Array构造函数创建和new Array()

当我们使用Array构造函数创建数组的时候，**当我们传入的参数只有一个数值(n)的时候，代表需要创建长度为n的数组，如果传入的参数是一系列参数，代表创建长度为参数的个数，且只包含这些参数**

```
new Arrray(5)//[empty × 5]
new Array(5,1)//[5,1]
new Array("a")//["a"]
console.log(new Array({ length: 3 })); //[{length:3}]
```
**注意：这里有一个细节问题，就是当我们传入的参数为一个数值的时候，会带来数组空位的问题**
## 数组静态方法

###   Array.from
>Array.from() 方法从一个类似数组或可迭代对象创建一个新的，浅拷贝的数组实例。
它的语法如下：
```
Array.from(arrayLike[, mapFn[, thisArg]])
arrayLike: 想要转换成数组的伪数组对象(包含length)或可迭代对象。
mapFn:可选， 如果指定了该参数，新数组中的每个元素会执行该回调函数。也就是支持一个映射函数
thisArg:可选参数，执行回调函数 mapFn 时 this 对象。 
```
下面是针对他的使用案例

```
	console.log(Array.from('abc')); //["a", "b", "c"]
	console.log(Array.from('abc', (val) => val + 2)); // ["a2", "b2", "c2"]
```
### Array.of
>Array.of() 方法创建一个具有可变数量参数的新数组实例，而不考虑参数的数量或类型。

```
	console.log(Array.of(1, 2, 3)); // [1, 2, 3]
	console.log(Array.of(7))//[7]
    //polyfill方法
    if (!Array.of) {
        Array.of = function() {
            return Array.prototype.slice.call(arguments);
        };
    }
```
### Arrray.from 
>Array.isArray() 用于确定传递的值是否是一个 Array。

如果对象是 Array ，则返回true，否则为false。

#### instanceof 和 isArray
当检测Array实例时, Array.isArray 优于 instanceof,因为Array.isArray能检测iframes.

## 关于数组的length
>length 属性的值是一个 0 到 232-1 的整数。

数组的长度在属性描述符里面是，可修改不可删除和遍历的


数组的length在使用上有一个需要注意的点，当我们进行增删数组的时候，数组长度会自动变化，当我们将Length长度变大的时候，数组元素会自动扩充，当我们改小Length的时候，会移除部分元素
```
const a = [];
a.length = 5;
console.log(a);//<5 empty items> 
a.length=3
console.log(a)//[ <3 empty items> ]

```
## 理解数组空位

>数组的空位指，数组的某一个位置没有任何值。比如，Array构造函数返回的数组都是空位

```
Array(3) // [, , ,]
```
**注意，空位不是undefined，一个位置的值等于undefined，依然是有值的。空位是没有任何值，in运算符可以说明这一点。**
```
0 in [, , ,] // false
0 in [undefined, undefined, undefined] // true
```
ES5 对空位的处理，已经很不一致了，大多数情况下会忽略空位。


- forEach(), filter(), reduce(), every() 和some()都会跳过空位。
- map()会跳过空位，但会保留这个值
- join()和toString()会将空位视为undefined，而undefined和null会被处理成空字符串。
```
// forEach方法
[,'a'].forEach((x,i) => console.log(i)); // 1

// filter方法
['a',,'b'].filter(x => true) // ['a','b']

// every方法
[,'a'].every(x => x==='a') // true

// reduce方法
[1,,2].reduce((x,y) => x+y) // 3

// some方法
[,'a'].some(x => x !== 'a') // false

// map方法
[,'a'].map(x => 1) // [,1]

// join方法
[,'a',undefined,null].join('#') // "#a##"

// toString方法
[,'a',undefined,null].toString() // ",a,,"
```
ES6 则是明确将空位转为undefined。


for...of循环也会遍历空位。
```
let arr = [, ,];
for (let i of arr) {
  console.log(1);
}
// 1
// 1
```
entries()、keys()、values()、find()和findIndex()会将空位处理成undefined。
```
// entries()
[...[,'a'].entries()] // [[0,undefined], [1,"a"]]

// keys()
[...[,'a'].keys()] // [0,1]

// values()
[...[,'a'].values()] // [undefined,"a"]

// find()
[,'a'].find(x => true) // undefined

// findIndex()
[,'a'].findIndex(x => true) // 0
```

## Array 原型上的方法

### 修改器方法

#### copywithin
>copyWithin() 方法浅复制数组的一部分到同一数组中的另一个位置，并返回它，不会改变原数组的长度。
语法如下：

arr.copyWithin(target[, start[, end]])
target
0 为基底的索引，复制序列到该位置。如果是负数，target 将从末尾开始计算。
如果 target 大于等于 arr.length，将会不发生拷贝。如果 target 在 start 之后，复制的序列将被修改以符合 arr.length。

start
0 为基底的索引，开始复制元素的起始位置。如果 start 被忽略，copyWithin 将会从0开始复制。如果 start 为负，则其指定的索引位置等同于 length+start，length 为数组的长度。end 也是如此。


end
0 为基底的索引，开始复制元素的结束位置。copyWithin 将会拷贝到该位置，但不包括 end 这个位置的元素。如果 end 被忽略，copyWithin 方法将会一直复制至数组结尾（默认为 arr.length）

copyWithin 方法不要求其 this 值必须是一个数组对象；除此之外，copyWithin 是一个可变方法，它可以改变 this 对象本身，并且返回它，而不仅仅是它的拷贝。

```
[1, 2, 3, 4, 5].copyWithin(-2)
// [1, 2, 3, 1, 2]

[1, 2, 3, 4, 5].copyWithin(0, 3)
// [4, 5, 3, 4, 5]

[1, 2, 3, 4, 5].copyWithin(0, 3, 4)
// [4, 2, 3, 4, 5]

[1, 2, 3, 4, 5].copyWithin(-2, -3, -1)
// [1, 2, 3, 3, 4]

```
#### fill
>fill() 方法用一个固定值填充一个数组中从起始索引到终止索引内的全部元素。不包括终止索引。
语法如下

arr.fill(value[, start[, end]])

value 用来填充数组元素的值。

start 可选 起始索引，默认值为0。

end 可选 终止索引，默认值为 this.length。

如果 start 是个负数, 则开始索引会被自动计算成为 length+start, 其中 length 是 this 对象的 length 属性值。如果 end 是个负数, 则结束索引会被自动计算成为 length+end。

fill 方法故意被设计成通用方法, 该方法不要求 this 是数组对象。

fill 方法是个可变方法, 它会改变调用它的 this 对象本身, 然后返回它, 而并不是返回一个副本。

当一个对象被传递给 fill方法的时候, 填充数组的是这个对象的引用。
```
[1, 2, 3].fill(4);               // [4, 4, 4]
[1, 2, 3].fill(4, 1);            // [1, 4, 4]
[1, 2, 3].fill(4, 1, 2);         // [1, 4, 3]
[1, 2, 3].fill(4, 1, 1);         // [1, 2, 3]
[1, 2, 3].fill(4, 3, 3);         // [1, 2, 3]
[1, 2, 3].fill(4, -3, -2);       // [4, 2, 3]
[1, 2, 3].fill(4, NaN, NaN);     // [1, 2, 3]
[1, 2, 3].fill(4, 3, 5);         // [1, 2, 3]
Array(3).fill(4);                // [4, 4, 4]
[].fill.call({ length: 3 }, 4);  // {0: 4, 1: 4, 2: 4, length: 3}

// Objects by reference.
var arr = Array(3).fill({}) // [{}, {}, {}];
// 需要注意如果fill的参数为引用类型，会导致都执行都一个引用类型
// 如 arr[0] === arr[1] 为true
arr[0].hi = "hi"; // [{ hi: "hi" }, { hi: "hi" }, { hi: "hi" }]
```
#### push
#### pop
#### shift
#### unshift
>unshift() 方法将一个或多个元素添加到数组的开头，并返回该数组的新长度(该方法修改原有数组)。


#### sort
>sort() 方法用原地算法对数组的元素进行排序，并返回数组。默认排序顺序是在将元素转换为字符串，然后比较它们的UTF-16代码单元值序列时构建的

如果没有指明 compareFunction ，那么元素会按照转换为的字符串的诸个字符的Unicode位点进行排序。例如 "Banana" 会被排列到 "cherry" 之前。当数字按由小到大排序时，9 出现在 80 之前，但因为（没有指明 compareFunction），比较的数字会先被转换为字符串，所以在Unicode顺序上 "80" 要比 "9" 要靠前。


**使用映射改善排序**


compareFunction 可能需要对元素做多次映射以实现排序，尤其当 compareFunction 较为复杂，且元素较多的时候，某些 compareFunction 可能会导致很高的负载。使用 map 辅助排序将会是一个好主意。基本思想是首先将数组中的每个元素比较的实际值取出来，排序后再将数组恢复
```
// 需要被排序的数组
var list = ['Delta', 'alpha', 'CHARLIE', 'bravo'];

// 对需要排序的数字和位置的临时存储
var mapped = list.map(function(el, i) {
  return { index: i, value: el.toLowerCase() };
})

// 按照多个值排序数组
mapped.sort(function(a, b) {
  return +(a.value > b.value) || +(a.value === b.value) - 1;
});

// 根据索引得到排序的结果
var result = mapped.map(function(el){
  return list[el.index];
});
```

####  reverse方法
>reverse() 方法将数组中元素的位置颠倒，并返回该数组。数组的第一个元素会变成最后一个，数组的最后一个元素变成第一个。该方法会改变原数组。

reverse方法是特意类化的；此方法可被 called 或 applied于类似数组对象。对象如果不包含反映一系列连续的、基于零的数值属性中的最后一个长度的属性，则该对象可能不会以任何有意义的方式运行。

如：颠倒类数组中的元素
```
const a = { 0: 1, 1: 2, 2: 3, length: 3 };
Array.prototype.reverse.call(a);
console.log(a); // {0: 3, 1: 2, 2: 1, length: 3}

```
#### Array.prototype.splice()
>splice() 方法通过删除或替换现有元素或者原地添加新的元素来修改数组,并以数组形式返回被修改的内容。此方法会改变原数组。
语法：
array.splice(start[, deleteCount[, item1[, item2[, ...]]]])

start​指定修改的开始位置（从0计数）。如果超出了数组的长度，则从数组末尾开始添加内容；如果是负值，则表示从数组末位开始的第几位（从-1计数，这意味着-n是倒数第n个元素并且等价于array.length-n）；如果负数的绝对值大于数组的长度，则表示开始位置为第0位。

deleteCount 可选整数，表示要移除的数组元素的个数。如果 deleteCount 大于 start 之后的元素的总数，则从 start 后面的元素都将被删除（含第 start 位）。如果 deleteCount 被省略了，或者它的值大于等于array.length - start(也就是说，如果它大于或者等于start之后的所有元素的数量)，那么start之后数组的所有元素都会被删除。如果 deleteCount 是 0 或者负数，则不移除元素。这种情况下，至少应添加一个新元素。

item1, item2, ... 可选要添加进数组的元素,从start 位置开始。如果不指定，则 splice() 将只删除数组元素。

### 访问方法
>下面的这些方法绝对不会改变调用它们的对象的值，只会返回一个新的数组或者返回一个其它的期望值。
####  concat 方法
>concat()方法可以在现有数组全部元素基础上创建一个新数组。它首先会创建一个当前数组的副本，然后再把它的参数添加到副本末尾，最后返回这个新构建的数组

如果传入的一组参数中，包含的有数组就将数组打平展开(双重嵌套的数组不会进行此操作)
```
	console.log([1, 2, 3].concat(4, [5, 6])); //[1,2,3,4,5,6]
	console.log([1, 2, 3].concat(4, [5, [6]])); //[1,2,3,4,5,[6]]
```

concat方法不会改变this或任何作为参数提供的数组，而是返回一个浅拷贝，它包含与原始数组相结合的相同元素的副本。 原始数组的元素将复制到新数组中，如下所示：

- 对象引用（而不是实际对象）：concat将对象引用复制到新数组中。 原始数组和新数组都引用相同的对象。 也就是说，如果引用的对象被修改，则更改对于新数组和原始数组都是可见的。 这包括也是数组的数组参数的元素。
- 数据类型如字符串，数字和布尔（不是String，Number 和 Boolean 对象）：concat将字符串和数字的值复制到新数组中。



**注意：可以阻止打平数组**

打平数组参数的行为可以重写，方法是在参数数组上指定一个特殊的符号：Symbol.isConcatSpreadable。这个符号能够阻止 concat()打平参数数组。相反，把这个值设置为 true 可以强制打平类数组对象

```
let colors = ["red", "green", "blue"]; 
let newColors = ["black", "brown"]; 
let moreNewColors = { 
 [Symbol.isConcatSpreadable]: true, 
 length: 2, 
 0: "pink", 
 1: "cyan" 
}; 
newColors[Symbol.isConcatSpreadable] = false; 
// 强制不打平数组
let colors2 = colors.concat("yellow", newColors); 
// 强制打平类数组对象
let colors3 = colors.concat(moreNewColors);
```
#### inclueds
>includes() 方法用来判断一个数组是否包含一个指定的值，根据情况，如果包含则返回 true，否则返回false。
语法如下：
arr.includes(valueToFind[, fromIndex])
valueToFind： 需要查找的元素值。
fromIndex: 从fromIndex 索引处开始查找 valueToFind。如果为负值，则按升序从 array.length + fromIndex 的索引开始搜,如果计算出来的值还是小于0，那么整个数组都会被搜索。默认为 0。如果 fromIndex 大于等于数组的长度，则会返回 false，且该数组不会被搜索。

```
var arr = ['a', 'b', 'c'];

arr.includes('a', -100); // true
arr.includes('b', -100); // true
arr.includes('c', -100); // true
arr.includes('a', -2); // false
```
#### JOIn方法
>join() 方法将一个数组（或一个类数组对象）的所有元素连接成一个字符串并返回这个字符串。如果数组只有一个项目，那么将返回该项目而不使用分隔符。

```
ar a = ['Wind', 'Rain', 'Fire'];
var myVar1 = a.join();      // myVar1的值变为"Wind,Rain,Fire"
var myVar2 = a.join(', ');  // myVar2的值变为"Wind, Rain, Fire"
var myVar3 = a.join(' + '); // myVar3的值变为"Wind + Rain + Fire"
var myVar4 = a.join('');    // myVar4的值变为"WindRainFire"
```
**如果数组中某一项是 null 或 undefined，则在 join()、toLocaleString()、toString()和 valueOf()返回的结果中会以空字符串表示**
#### slice 方法
>slice() 方法返回一个新的数组对象，这一对象是一个由 begin 和 end 决定的原数组的浅拷贝（包括 begin，不包括end）。原始数组不会被改变。

语法

arr.slice([begin[, end]])

begin 可选
提取起始处的索引（从 0 开始），从该索引开始提取原数组元素。
如果该参数为负数，则表示从原数组中的倒数第几个元素开始提取，slice(-2) 表示提取原数组中的倒数第二个元素到最后一个元素（包含最后一个元素）。
如果省略 begin，则 slice 从索引 0 开始。
如果 begin 超出原数组的索引范围，则会返回空数组。

end 可选
提取终止处的索引（从 0 开始），在该索引处结束提取原数组元素。slice 会提取原数组中索引从 begin 到 end 的所有元素（包含 begin，但不包含 end）。
slice(1,4) 会提取原数组中从第二个元素开始一直到第四个元素的所有元素 （索引为 1, 2, 3的元素）。
如果该参数为负数， 则它表示在原数组中的倒数第几个元素结束抽取。 slice(-2,-1) 表示抽取了原数组中的倒数第二个元素到最后一个元素（不包含最后一个元素，也就是只有倒数第二个元素）。
如果 end 被省略，则 slice 会一直提取到原数组末尾。
如果 end 大于数组的长度，slice 也会一直提取到原数组末尾。

#### indexOf
>indexOf()方法返回在数组中可以找到一个给定元素的第一个索引，如果不存在，则返回-1。

语法如下

arr.indexOf(searchElement[, fromIndex])

fromIndex 可选
开始查找的位置。如果该索引值大于或等于数组长度，意味着不会在数组里查找，返回-1。如果参数中提供的索引值是一个负值，则将其作为数组末尾的一个抵消，即-1表示从最后一个元素开始查找，-2表示从倒数第二个元素开始查找 ，以此类推。 注意：如果参数中提供的索引值是一个负值，并不改变其查找顺序，查找顺序仍然是从前向后查询数组。如果抵消后的索引值仍小于0，则整个数组都将会被查询。其默认值为0.
#### lastIndexOf
>lastIndexOf() 方法返回指定元素（也即有效的 JavaScript 值或变量）在数组中的最后一个的索引，如果不存在则返回 -1。从数组的后面向前查找，从 fromIndex 处开始。
语法:

arr.lastIndexOf(searchElement[, fromIndex])

fromIndex 可选

从此位置开始逆向查找。默认为数组的长度减 1(arr.length - 1)，即整个数组都被查找。如果该值大于或等于数组的长度，则整个数组会被查找。如果为负值，将其视为从数组末尾向前的偏移。即使该值为负，数组仍然会被从后向前查找。如果该值为负时，其绝对值大于数组长度，则方法返回 -1，即数组不会被查找。


### 迭代方法
在下面的众多遍历方法中，有很多方法都需要指定一个回调函数作为参数。在每一个数组元素都分别执行完回调函数之前，数组的length属性会被缓存在某个地方，所以，如果你在回调函数中为当前数组添加了新的元素，那么那些新添加的元素是不会被遍历到的。此外，如果在回调函数中对当前数组进行了其它修改，比如改变某个元素的值或者删掉某个元素，那么随后的遍历操作可能会受到未预期的影响
#### entries，keys,values 方法

#### forEach
>forEach() 方法对数组的每个元素执行一次给定的函数。

语法：
arr.forEach(callback(currentValue [, index [, array]])[, thisArg])

callback：为数组中每个元素执行的函数，该函数接收一至三个参数：

	currentValue:数组中正在处理的当前元素。

	index: 可选数组中正在处理的当前元素的索引。
	
	array 可选forEach() 方法正在操作的数组。

thisArg 可选:可选参数。当执行回调函数 callback 时，用作 this 的值。如果 thisArg 参数有值，则每次 callback 函数被调用时，this 都会指向 thisArg 参数。如果省略了 thisArg 参数，或者其值为 null 或 undefined，this 则指向全局对象。按照函数观察到 this 的常用规则，callback 函数最终可观察到 this 值，注意：如果使用箭头函数表达式来传入函数参数， thisArg 参数会被忽略，因为箭头函数在词法上绑定了 this 值。



注意：**除了抛出异常以外，没有办法中止或跳出 forEach() 循环。如果你需要中止或跳出循环，forEach() 方法不是应当使用的工具。**


forEach() 遍历的范围在第一次调用 callback 前就会确定。调用 forEach 后添加到数组中的项不会被 callback 访问到。如果已经存在的值被改变，则传递给 callback 的值是 forEach() 遍历到他们那一刻的值。已删除的项不会被遍历到。如果已访问的元素在迭代时被删除了（例如使用 shift()），之后的元素将被跳过


使用forEach需要注意以下几点
1. 使用forEach遍历包含空位的数组(会跳过空位)
```
const arraySparse = [1, 3, , 7];
let numCallbackRuns = 0;

arraySparse.forEach(function(element, index) {
	console.log(element, index); // index还是正常的哦
	numCallbackRuns++;
});

console.log('numCallbackRuns: ', numCallbackRuns);

// 1 0
// 3 1
// 7 3
// numCallbackRuns: 3

```
2. 在遍历的时候操作了原数组
```
// 移除一个元素
var words = ['one', 'two', 'three', 'four'];
words.forEach(function(word) {
  console.log(word);
  if (word === 'two') {
    words.shift();
  }
});
// one
// two
// four

// 新增一个元素(并不会导致多遍历一次)

var words = ['one', 'two', 'three', 'four'];
words.forEach(function(word, index) {
	console.log(word);
	if (word === 'two') {
		words.push(1231);
	}
});
// one;
// two;
// three;
// fou;

```

#### every 方法
>every() 方法测试一个数组内的所有元素是否都能通过某个指定函数的测试。它返回一个布尔值。

**若收到一个空数组，此方法在一切情况下都会返回 true。**

语法：
arr.every(callback(element[, index[, array]])[, thisArg])



实现 every Polyfill
```
Array.prototype.myEvery = function(fn, context) {
				if (typeof callback !== 'function') {
					throw Error('错误');
				}
				let passed = true;
				for (let i = 0; i < len; i++) {
					// 判断这个属性是否在对象上
					if (Object.prototype.hasOwnProperty.call(this, i)) {
						if (!passed) {
							break;
						}
						passed = !!callback.call(context, this[i], i, this);
					}
				}
				return passed;
			};
```


####  map()
> map() 方法创建一个新数组，其结果是该数组中的每个元素是调用一次提供的函数后的返回值。

语法：
```
var new_array = arr.map(function callback(currentValue[, index[, array]]) {
 // Return element for new_array 
}[, thisArg])
```
map 方法处理数组元素的范围是在 callback 方法第一次调用之前就已经确定了。调用map方法之后追加的数组元素不会被callback访问。如果存在的数组元素改变了，那么传给callback的值是map访问该元素时的值。在map函数调用后但在访问该元素前，该元素被删除的话，则无法被访问到。




实现Map Polyfill

```
Array.prototype.myMap = function(callback, context) {
				if (typeof callback !== 'function') {
					throw Error('错误');
				}
				const result = [];
				const len = this.length;
				for (let i = 0; i < len; i++) {
					// 判断这个属性是否在对象上
					if (Object.prototype.hasOwnProperty.call(this, i)) {
						// 使用call 的原因主要是可能改变context
						result.push(callback.call(context, this[i], i, this));
					}
				}
				return result;
			};
```


#### some()
>some() 方法测试数组中是不是至少有1个元素通过了被提供的函数测试。它返回的是一个Boolean类型的值。

**如果用一个空数组进行测试，在任何情况下它返回的都是false。**


语法：
arr.some(callback(element[, index[, array]])[, thisArg])


```

    Array.prototype.mysome = function(fn, context) {
				if (typeof callback !== 'function') {
					throw Error('错误');
				}
				let passed = false;
				for (let i = 0; i < len; i++) {
					// 判断这个属性是否在对象上
					if (Object.prototype.hasOwnProperty.call(this, i)) {
						if (passed) {
							break;
						}
						passed = !!callback.call(context, this[i], i, this);
					}
				}
				return passed;
			};  
```
#### filter
>filter() 方法创建一个新数组, 其包含通过所提供函数实现的测试的所有元素。 

语法：
var newArray = arr.filter(callback(element[, index[, array]])[, thisArg])


#### find 和 findIndex()
> find() 方法返回数组中满足提供的测试函数的第一个元素的值。否则返回 undefined。


> findIndex()方法返回数组中满足提供的测试函数的第一个元素的索引。若没有找到对应元素则返回-1。


语法：
arr.find(callback(element[, index[, array]])[, thisArg])

在第一次调用 callback函数时会确定元素的索引范围，因此在 find方法开始执行之后添加到数组的新元素将不会被 callback函数访问到。如果数组中一个尚未被callback函数访问到的元素的值被callback函数所改变，那么当callback函数访问到它时，它的值是将是根据它在数组中的索引所访问到的当前值。被删除的元素仍旧会被访问到，但是其值已经是undefined了。

```
var a = [1, 2, 3, 4, 5];
a.find((element) => {
	a.push(6);
	console.log(element);// 1 2 3 4 5
});

```
#### reduce和reduceRight
>reduce() 方法对数组中的每个元素执行一个由您提供的reducer函数(升序执行)，将其结果汇总为单个返回值。
语法
arr.reduce(callback(accumulator, currentValue[, index[, array]])[, initialValue])

callback: 执行数组中每个值 (如果没有提供 initialValue则第一个值除外)的函数，包含四个参数：
	accumulator: 累计器累计回调的返回值; 它是上一次调用回调时返回的累积值，或initialValue（见于下方）。
	currentValue: 数组中正在处理的元素。
	index 可选 数组中正在处理的当前元素的索引。 如果提供了initialValue，则起始索引号为0，否则从索引1起始。
	array 可选 调用reduce()的数组
initialValue可选 作为第一次调用 callback函数时的第一个参数的值。 如果没有提供初始值，则将使用数组中的第一个元素。 在没有初始值的空数组上调用 reduce 将报错。

回调函数第一次执行时，accumulator 和currentValue的取值有两种情况：如果调用reduce()时提供了initialValue，accumulator取值为initialValue，currentValue取数组中的第一个值；如果没有提供 initialValue，那么accumulator取数组中的第一个值，currentValue取数组中的第二个值。

注意：如果没有提供initialValue，reduce 会从索引1的地方开始执行 callback 方法，跳过第一个索引。如果提供initialValue，从索引0开始。

如果数组为空且没有提供initialValue，会抛出TypeError 。如果数组仅有一个元素（无论位置如何）并且没有提供initialValue， 或者有提供initialValue但是数组为空，那么此唯一值将被返回并且callback不会被执行。
### 其他方法

#### flat
>flat() 方法会按照一个可指定的深度递归遍历数组，并将所有元素与遍历到的子数组中的元素合并为一个新数组返回。
指定要提取嵌套数组的结构深度，默认值为 1。
1. flat() 方法会移除数组中的空项:
```
var arr4 = [1, 2, , 4, 5];
arr4.flat();
// [1, 2, 4, 5]
```

需要思考flat的各种替代方案
[参考链接](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/flat)
## 数组转为其他类型
1. 数组转boolean(永远转为true)
2. 数组转字符串(先调用valueof再调用toString)
3. 数组转Number(先调用valueOf 再调用toString)
 
## 知识拓展
1. [理解什么是关联数组](https://www.netinbag.com/cn/internet/what-is-an-associative-array.html)

