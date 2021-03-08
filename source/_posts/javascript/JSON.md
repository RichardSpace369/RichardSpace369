---
title: JSON
author: richard
date: 2021/03/05 18:36:00
toc: true
excerpt: 理解 JSON 最关键的一点是要把它当成一种数据格式，而不是编程语言。JSON 不属于 JavaScript，它们只是拥有相同的语法而已。JSON 也不是只能在 JavaScript 中使用，它是一种通用数据格式...
categories:
- javascript
- JSON
---

理解 JSON 最关键的一点是要把它当成一种数据格式，而不是编程语言。JSON 不属于 JavaScript，它们只是拥有相同的语法而已。JSON 也不是只能在 JavaScript 中使用，它是一种通用数据格式

## 语法
JSON 语法支持表示 3 种类型的值。
1. 简单值：字符串、数值、布尔值和 null 可以在 JSON 中出现，就像在 JavaScript 中一样。特殊值 undefined 不可以。
2. 对象：第一种复杂数据类型，对象表示有序键/值对。每个值可以是简单值，也可以是复杂类型
3. 数组：第二种复杂数据类型，数组表示可以通过数值索引访问的值的有序列表。数组的值可以是任意类型，包括简单值、对象，甚至其他数组。JSON 没有变量、函数或对象实例的概念。

### 简单值
JavaScript 字符串与 JSON 字符串的主要区别是，JSON 字符串必须使用双引号

布尔值和 null 本身也是有效的 JSON 值

### 对象
与 JavaScript 对象字面量相比，JSON 主要有两处不同。

首先，没有变量声明（JSON 中没有变量）。

其次，最后没有分号（不需要，因为不是 JavaScript 语句）。

同样，用引号将属性名包围起来才是有效的JSON。


## 解析与序列化
### JSON.stringify

JSON.stringify 方法，除了要序列化的对象，还可以接收两个参数，第一个参数是**过滤器**,第二个参数是用于**缩进结果** JSON 字符串的选项

#### 过滤结果
如果第二个参数是一个数组，那么 JSON.stringify()返回的结果只会包含该数组中列出的对象属性
```
let book = { 
 title: "Professional JavaScript", 
 authors: [ 
 "Nicholas C. Zakas", 
 "Matt Frisbie" 
 ], 
 edition: 4, 
 year: 2017 
}; 
let jsonText = JSON.stringify(book, ["title", "edition"]);//{"title":"Professional JavaScript","edition":4}

```

如果第二个参数是一个函数，则行为又有不同。提供的函数接收两个参数：属性名（key）和属性值（value）。可以根据这个 key 决定要对相应属性执行什么操作。这个 key 始终是字符串，只是在值不属于某个键/值对时会是空字符串。

注意，返回 undefined 会导致属性被忽略

```
let book = {
	title: 'Professional JavaScript',
	authors: ['Nicholas C. Zakas', 'Matt Frisbie'],
	edition: 4,
	year: 2017,
};
let jsonText = JSON.stringify(book, (key, value) => {
	switch (key) {
		case 'authors':
			return value.join(',');
		case 'year':
			return 5000;
		case 'edition':
			return undefined;
		default:
			return value;
	}
}); //{"title":"Professional JavaScript","authors":"Nicholas C. Zakas,Matt Frisbie","year":5000}

```
#### 字符串缩进

JSON.stringify()方法的第三个参数控制缩进和空格。

在这个参数是数值时，表示每一级缩进的空格数

如果缩进参数是一个字符串而非数值，那么 JSON 字符串中就会使用这个字符串而不是空格来缩进。使用字符串，也可以将缩进字符设置为 Tab 或任意字符

#### toJSON()方法
有时候，对象需要在 JSON.stringify()之上自定义 JSON 序列化。此时，可以在要序列化的对象中添加 toJSON()方法，序列化时会基于这个方法返回适当的 JSON 表示

```
let book = {
	title: 'Professional JavaScript',
	authors: ['Nicholas C. Zakas', 'Matt Frisbie'],
	edition: 4,
	year: 2017,
	toJSON: function() {
		return this.title;
	},
};
let jsonText = JSON.stringify(book);
console.log(jsonText);//"Professional JavaScript"

```

注意，**箭头函数不能用来定义 toJSON()方法**。主要原因是箭头函数的词法作用域是全局作用域，在这种情况下不合适



JSON()方法可以与过滤函数一起使用，因此理解不同序列化流程的顺序非常重要。在把对象传给 JSON.stringify()时会执行如下步骤。


(1) 如果可以获取实际的值，则调用 toJSON()方法获取实际的值，否则使用默认的序列化。

(2) 如果提供了第二个参数，则应用过滤。传入过滤函数的值就是第(1)步返回的值。

(3) 第(2)步返回的每个值都会相应地进行序列化。


(4) 如果提供了第三个参数，则相应地进行缩进

### 解析选项 JSON.parse 方法

JSON.parse()方法也可以接收一个额外的参数，这个函数(还原函数)会针对每个键/值对都调用一次。这个函数也接收两个参数，属性名（key）和属性值（value），另外也需要返回值

如果还原函数返回 undefined，则结果中就会删除相应的键。如果返回了其他任何值，则该值就会成为相应键的值插入到结果中。


还原函数经常被用于把日期字符串转换为 Date 对象


## 手动实现 JSON.stringify

```
function jsonStringify(data) {
  let type = typeof data;
  if(type !== 'object') {
    let result = data;
    //data 可能是基础数据类型的情况在这里处理
    if (Number.isNaN(data) || data === Infinity) {
       //NaN 和 Infinity 序列化返回 "null"
       result = "null";
    } else if (type === 'function' || type === 'undefined' || type === 'symbol') {
      // 由于 function 序列化返回 undefined，因此和 undefined、symbol 一起处理
       return undefined;
    } else if (type === 'string') {
       result = '"' + data + '"';
    }
    return String(result);
  } else if (type === 'object') {
     if (data === null) {
        return "null"  // 第01讲有讲过 typeof null 为'object'的特殊情况
     } else if (data.toJSON && typeof data.toJSON === 'function') {
        return jsonStringify(data.toJSON());
     } else if (data instanceof Array) {
        let result = [];
        //如果是数组，那么数组里面的每一项类型又有可能是多样的
        data.forEach((item, index) => {
        if (typeof item === 'undefined' || typeof item === 'function' || typeof item === 'symbol') {
             result[index] = "null";
           } else {
               result[index] = jsonStringify(item);
           }
         });
         result = "[" + result + "]";
         return result.replace(/'/g, '"');
      } else {
         // 处理普通对象
         let result = [];
         Object.keys(data).forEach((item, index) => {
            if (typeof item !== 'symbol') {
              //key 如果是 symbol 对象，忽略
              if (data[item] !== undefined && typeof data[item] !== 'function' && typeof data[item] !== 'symbol') {
                //键值如果是 undefined、function、symbol 为属性值，忽略
                result.push('"' + item + '"' + ":" + jsonStringify(data[item]));
              }
            }
         });
         return ("{" + result + "}").replace(/'/g, '"');
        }
    }
}

```
