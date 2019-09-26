---
layout:     post
title:      "深入理解混合对象“类”和原型"
subtitle:   ""
author:     "csy"
header-img: "img/home-bg2.jpg"
header-mask:  0.5
catalog: true
tags:
- javascript
- 笔记
---

# 1. 混合对象“类”
javascript中类和其他语言中的类完全不同。传统面向类的语言中父类和子类、子类和实例之间其实是复制操作，但是在javascript中并没有复制，对象之间是通过内部的 [[Prototype]] 链委托关联的，这样一个对象就可以通过委托访问另一个对象的属性和函数。

多态看起来像是从子类引用父类，但是本质上就是引用。

混入模式（显式混入、隐式混入）可以用来模拟类的复制行为，但是很丑陋。而且显式混入无法完全模拟类的复制行为，因为对象（还有函数）只能复制对共享函数对象的引用。


# 2. 原型链
[[Prototype]]机制就是对象中的一个内部链接引用另外一个对象。如果在第一个对象上没有找到需要的属性或者方法引用，引擎就会继续在[[Prototype]]关联的对象上进行查找，以此类推，这一系列对象的链接被称为"原型链"。

对于默认的 [[Get]] 操作来说，如果无法在对象本身找到需要的属性，就会继续访问对象的 [[Prototype]] 链。

for..in 遍历， in 操作符来检查属性在对象中是否存在时，会查找对象的整条原型链（无论属性是否可枚举）。

```javascript
var anotherObject = { a:2 };
var myObject = Object.create(anotherObject)
for(key in myObject) {
  console.log(key);
}
console.log("a" in myObject);
```
## 属性设置和屏蔽
- 原型链上的已有xxx非只读属性，会直接在myObject中添加一个新的属性xxx，它是屏蔽属性。
- 原型链上的已有xxx只读属性，则无法创建同名的xxx屏蔽属性。
- 原型链上的已有xxx但它是一个setter，则会调用这个setter，xxx不会被添加到myObject，也不会重新定义xxx。

应尽量避免使用屏蔽，以及注意隐式屏蔽的情况

```javascript
var anotherObject = { a:2 };
var myObject = Object.create(anotherObject)
console.log(anotherObject.hasOwnProperty("a")) //true
console.log(myObject.hasOwnProperty("a")) //false
myObject.a++ //隐式屏蔽
console.log(anotherObject.a) //2
console.log(myObject.a) //3
console.log(myObject.hasOwnProperty("a")) //true
```
## 误解
```javascript
function Foo(name) {
  this.name = name;
}
Foo.prototype.myName = function () {
  return this.name;
};
var a = new Foo("a");
var b = new Foo("b");
a.myName(); // "a" 
b.myName(); // "b"
```
这段代码，看起来好像创建 a 和 b 时会把 Foo.prototype 对象复制到这两个对象中，然而事实并不是这样。
在创建的过程中，a 和 b 的内部 [[Prototype]] 都会关联到 Foo.prototype 上。
当 a 和 b 中无法找到 myName 时，它会通过委托，在 Foo.prototype 上找到。

```javascript
function Foo() { /* .. */ }
Foo.prototype = { /* .. */ }; // 创建一个新原型对象
var a1 = new Foo();
a1.constructor === Foo; // false! 
a1.constructor === Object; // true! 
```
构造函数只是通过默认的 [[prototype]] 委托指向Foo
Foo.prototype的 .constructor 属性只是Foo函数在声明时的默认属性
但是这个对象也没有.constructor 属性（不过默认的 Foo.prototype 对象有这个属性！），所以它会继续委托，这次会委托给委托链顶端的 Object.prototype。这个对象有 .constructor 属性，指向内置的 Object(..) 函数。

## 关联原型
```javascript
// ES6 之前需要抛弃默认的 Bar.prototype 
Bar.ptototype = Object.create( Foo.prototype ); 
// ES6 开始可以直接修改现有的 
Bar.prototype Object.setPrototypeOf( Bar.prototype, Foo.prototype );
```

## 找到继承祖先
`a instanceof Foo` 
a在整条[[prototype]]链中是否有指向 Foo.prototype 的对象。instanceof只能判断对象和函数之间的关系。无法判断两个对象间的关系

`a.isPrototypeOf(b)`
a是否出现在b的[[prototype]]链中


