---
title: Object
author: richard
toc: true
excerpt: 构造函数函数Object
categories:
- javascript
- Object
---

##  理解Object

在JavaScript中，几乎所有的对象都是Object类型的实例，它们都会从Object.prototype继承属性和方法。Object 构造函数为给定值创建一个对象包装器。Object构造函数，会根据给定的参数创建对象，具体有以下情况：

1. 如果给定值是 null 或 undefined，将会创建并返回一个空对象
2. 如果传进去的是一个基本类型的值，则会构造其包装类型的对象
3. 如果传进去的是引用类型的值，仍然会返回这个值，经他们复制的变量保有和源对象相同的引用地址

当以非构造函数形式被调用时，Object 的行为等同于 new Object()。

##  Object 构造函数的方法
###  Object.assign()
>Object.assign() 方法用于将所有可枚举属性的值从一个或多个源对象分配到目标对象。它将返回目标对象。

如果目标对象中的属性具有相同的键，则属性将被源对象中的属性覆盖。后面的源对象的属性将类似地覆盖前面的源对象的属性。

Object.assign 方法只会拷贝源对象自身的并且可枚举的属性到目标对象。该方法使用源对象的[[Get]]和目标对象的[[Set]]，所以它会调用**相关 getter 和 setter**。因此，它分配属性，而不仅仅是复制或定义新的属性。如果合并源包含getter，这可能使其不适合将新属性合并到原型中为了将属性定义（包括其可枚举性）复制到原型，应使用Object.getOwnPropertyDescriptor()和Object.defineProperty() 。


###  Object.create()
>Object.create()方法创建一个新对象，使用现有的对象来提供新创建的对象的__proto__。

```
Object.create(proto，[propertiesObject])
```

- proto  新创建对象的原型对象。

- propertiesObject 可选。需要传入一个对象(参考Object.defineProperties()的第二个参数),如果propertiesObject参数是 null 或非原始包装对象，则抛出一个 TypeError 异常。


###  Object.defineProperty()
>Object.defineProperty() 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回此对象

### Object.defineProperties()

### Object.entries()
### Object.keys()
>返回一个包含所有给定对象**自身可枚举属性名称的数组**。

### Object.values()
>返回给定对象自身可枚举值的数组。

###  Object.getOwnPropertyNames()

###  Object.getOwnPropertySymbols()

###  Object.preventExtensions()和Object.isExtensible()
>防止对象的任何扩展。

###  Object.freeze()和 Object.isFrozen()
>冻结对象：其他代码不能删除或更改任何属性。
注意:Object.freeze 只能冻结一层对象
```
const person={
    age:26,
    address:{
        prefix:"xxx"
    }
}
const frezzPerson=Object.freeze(person)
frezzPerson.age=28// 修改不成功
frezzPerson.address.prefix="yyyy"// 修改成功
```
### Object.getOwnPropertyDescriptors()
>Object.getOwnPropertyDescriptors() 方法用来获取一个对象的所有自身属性的描述符。


###  Object.getPrototypeOf()和Object.setPrototypeOf()
>Object.getPrototypeOf() 方法返回指定对象的原型 ( 即, 内部[[Prototype]]属性）。
###  Object.is()
>比较两个值是否相同。所有 NaN 值都相等（这与==和===不同）。

### Object.isSealed()
>Object.isSealed() 方法判断一个对象是否被密封。

##  Object 原型上的方法

###  Object.prototype.hasOwnProperty()
>返回一个布尔值 ，表示某个对象是否含有指定的属性，而且此属性非原型链继承的。

###  Object.prototype.isPrototypeOf()
>返回一个布尔值，表示指定的对象是否在本对象的原型链中。

### Object.prototype.propertyIsEnumerable()
>判断指定属性是否可枚举