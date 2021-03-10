---
title: 集合数据类型-map,weakMap,set,weakSet
author: richard
toc: true
excerpt: 掌握集合数据类型map,weakMap,set,weakSet...
tags:
- Map
- weakMap
- Set
- WeakSet
categories:
- javascript
- 集合数据类型
---


## Map
Map 是一种新的集合类型，为这门语言带来了真正的键/值存储机制。


使用 new 关键字和 Map 构造函数可以创建一个空映射：
```
const m = new Map();
```

如果想在创建的同时初始化实例，可以给 Map 构造函数**传入一个可迭代对象，需要包含键/值对数组**。可迭代对象中的每个键/值对都会按照**迭代顺序**插入到新映射实例中
```
// 使用嵌套数组初始化映射
const m1 = new Map([ 
 ["key1", "val1"], 
 ["key2", "val2"], 
 ["key3", "val3"] 
]);
// 使用自定义迭代器初始化映射
const m2 = new Map({ 
 [Symbol.iterator]: function*() { 
 yield ["key1", "val1"]; 
 yield ["key2", "val2"]; 
 yield ["key3", "val3"]; 
 } 
}); 
alert(m2.size); // 3 
// 映射期待的键/值对，无论是否提供
const m3 = new Map([[]]); 
alert(m3.has(undefined)); // true 
alert(m3.get(undefined)); // undefined
```
### Map方法

- get()
- has()
- set() 方法返回映射实例
- delete()
- clear()
- size 属性

### 顺序和迭代


映射实例可以提供一个迭代器（Iterator），能以插入顺序生成[key, value]形式的数组。

- 可以通过 entries()方法（或者 Symbol.iterator 属性，它引用 entries()）取得这个迭代器

```
const m = new Map([ 
 ["key1", "val1"], 
 ["key2", "val2"], 
 ["key3", "val3"] 
]); 
alert(m.entries === m[Symbol.iterator]); // true 
for (let pair of m.entries()) { 
 alert(pair); 
} 
// [key1,val1] 
// [key2,val2] 
// [key3,val3] 
for (let pair of m[Symbol.iterator]()) { 
 alert(pair); 
} 
// [key1,val1] 
// [key2,val2] 
// [key3,val3]
```
entries()是默认迭代器，所以可以直接对映射实例使用扩展操作，把映射转换为数组
  ```
  const m = new Map([ 
    ["key1", "val1"], 
    ["key2", "val2"], 
    ["key3", "val3"] 
    ]); 
  console.log([...m]); // [[key1,val1],[key2,val2],[key3,val3]]
  ```
- forEach(callback, opt_thisArg)方法并传入回调，依次迭代每个键/值对
```
const m = new Map([ 
 ["key1", "val1"], 
 ["key2", "val2"], 
 ["key3", "val3"] 
]); 
m.forEach((val, key) => alert(`${key} -> ${val}`)); 
// key1 -> val1 
// key2 -> val2 
// key3 -> val3
```
- keys()和 values()分别返回以插入顺序生成键和值的迭代器

```
const m = new Map([ 
 ["key1", "val1"], 
 ["key2", "val2"], 
 ["key3", "val3"] 
]); 
for (let key of m.keys()) { 
 alert(key); 
} 
// key1 
// key2 
// key3 
for (let key of m.values()) { 
 alert(key); 
} 
// value1 
// value2 
// value3
```

- 键和值在迭代器遍历时是可以修改的，但映射内部的引用则无法修改。当然，这并不妨碍修改作为键或值的对象内部的属性，因为这样并不影响它们在映射实例中的身份

```
const m1 = new Map([ 
 ["key1", "val1"] 
]); 
// 作为键的字符串原始值是不能修改的
for (let key of m1.keys()) { 
 key = "newKey"; 
 alert(key); // newKey 
 alert(m1.get("key1")); // val1 
}
```
### 选择 Object 还是 Map

1. 内存占用
Object 和 Map 的工程级实现在不同浏览器间存在明显差异，但存储单个键/值对所占用的内存数量都会随键的数量线性增加。批量添加或删除键/值对则**取决于各浏览器对该类型内存分配的工程实现**。不同浏览器的情况不同，但给定固定大小的内存，Map 大约可以比 Object 多存储 50%的键/值对。
2. 插入性能
向 Object 和 Map 中插入新键/值对的消耗大致相当，不过插入 Map 在所有浏览器中一般会稍微快一点儿。对这两个类型来说，插入速度并不会随着键/值对数量而线性增加。如果代码涉及大量插入操作，那么显然 Map 的性能更佳。

3. 查找速度
与插入不同，从大型 Object 和 Map 中查找键/值对的性能差异极小，但如果只包含少量键/值对，则 Object 有时候速度更快。在把 Object 当成数组使用的情况下（比如使用连续整数作为属性），浏览器引擎可以进行优化，在内存中使用更高效的布局。这对 Map 来说是不可能的。对这两个类型而言，
查找速度不会随着键/值对数量增加而线性增加。如果代码涉及大量查找操作，那么某些情况下可能选择 Object 更好一些。

4. 删除性能
使用 delete 删除 Object 属性的性能一直以来饱受诟病，目前在很多浏览器中仍然如此。为此，出现了一些伪删除对象属性的操作，包括把属性值设置为 undefined 或 null。但很多时候，这都是一种讨厌的或不适宜的折中。而对大多数浏览器引擎来说，Map 的 delete()操作都比插入和查找更快。如果代码涉及大量删除操作，那么毫无疑问应该选择 Map。

## WeakMap
WeakMap 是 Map 的“兄弟”类型，其 API 也是 Map 的子集。WeakMap 中的“weak”（弱），描述的是 **JavaScript 垃圾回收程序对待“弱映射”中键的方式。**


**弱映射中的键只能是 Object 或者继承自 Object 的类型**，尝试使用非对象设置键会抛出TypeError。值的类型没有限制

如果想在初始化时填充弱映射，则构造函数可以接收一个可迭代对象，其中需要包含键/值对数组。可迭代对象中的每个键/值都会按照迭代顺序插入新实例中：
```
const key1 = {id: 1}, 
 key2 = {id: 2},
 key3 = {id: 3}; 
// 使用嵌套数组初始化弱映射
const wm1 = new WeakMap([ 
 [key1, "val1"], 
 [key2, "val2"], 
 [key3, "val3"] 
]); 
alert(wm1.get(key1)); // val1 
alert(wm1.get(key2)); // val2 
alert(wm1.get(key3)); // val3
```

### 方法
- get()
- set()
- has()
- delete()

### 弱键

WeakMap 中“weak”表示弱映射的键是“弱弱地拿着”的。意思就是，这些键不属于正式的引用，不会阻止垃圾回收。但要注意的是，弱映射中值的引用可不是“弱弱地拿着”的。只要键存在，键/值对就会存在于映射中，并被当作对值的引用，因此就不会被当作垃圾回收

### 使用场景
1. 私有变量
2. DOM 节点元数据


## Set
使用 new 关键字和 Set 构造函数可以创建一个空集合：
```
const m = new Set();
```
如果想在创建的同时初始化实例，则可以给 Set 构造函数传入一个可迭代对象，其中需要包含插入到新集合实例中的元素：
```
// 使用数组初始化集合 
const s1 = new Set(["val1", "val2", "val3"]); 
alert(s1.size); // 3 
// 使用自定义迭代器初始化集合
const s2 = new Set({ 
 [Symbol.iterator]: function*() { 
 yield "val1"; 
 yield "val2"; 
 yield "val3"; 
 } 
}); 
alert(s2.size); // 3
```

### 方法
1. add()增加值
2. has()查询
3. delete()和 clear()删除元素：

### 顺序与迭代

Set 会维护值插入时的顺序，因此支持按顺序迭代。


- 集合实例可以提供一个迭代器（Iterator），能以插入顺序生成集合内容。可以通过 values()方法及其别名方法 keys()（或者 Symbol.iterator 属性，它引用 values()）取得这个迭代器：
```
const s = new Set(["val1", "val2", "val3"]); 
alert(s.values === s[Symbol.iterator]); // true 
alert(s.keys === s[Symbol.iterator]); // true 
for (let value of s.values()) { 
 alert(value); 
} 
// val1 
// val2 
// val3 
for (let value of s[Symbol.iterator]()) { 
 alert(value); 
} 
// val1 
// val2 
// val3
```
因为 values()是默认迭代器，所以可以直接对集合实例使用扩展操作，把集合转换为数组：
```
const s = new Set(["val1", "val2", "val3"]); 
console.log([...s]); // ["val1", "val2", "val3"]
```
- 集合的 entries()方法返回一个迭代器，可以按照插入顺序产生包含两个元素的数组，这两个元素是集合中每个值的重复出现：
```
const s = new Set(["val1", "val2", "val3"]); 
for (let pair of s.entries()) { 
 console.log(pair); 
} 
// ["val1", "val1"] 
// ["val2", "val2"] 
// ["val3", "val3"]
```
- 如果不使用迭代器，而是使用回调方式，则可以调用集合的 forEach()方法并传入回调，依次迭代每个键/值对。传入的回调接收可选的第二个参数，这个参数用于重写回调内部 this 的值
  
```
const s = new Set(["val1", "val2", "val3"]); 
s.forEach((val, dupVal) => alert(`${val} -> ${dupVal}`)); 
// val1 -> val1 
// val2 -> val2 
// val3 -> val3
```

## WeakSet

可以使用 new 关键字实例化一个空的 WeakSet：
```
const ws = new WeakSet();
```

弱集合中的值只能是 Object 或者继承自 Object 的类型，尝试使用非对象设置值会抛出 TypeError。


如果想在初始化时填充弱集合，则构造函数可以接收一个可迭代对象，其中需要包含有效的值。可迭代对象中的每个值都会按照迭代顺序插入到新实例中：

```
const val1 = {id: 1}, 
 val2 = {id: 2}, 
 val3 = {id: 3}; 
// 使用数组初始化弱集合
const ws1 = new WeakSet([val1, val2, val3]); 
alert(ws1.has(val1)); // true 
alert(ws1.has(val2)); // true 
alert(ws1.has(val3)); // true
```

### 方法
- add()
- has()查询
- delete()删除

### 使用场景

1.  DOM 节点元数据
