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

多态看起来像是从子类引用父类，但是本质上就是引用。继承的本质是重写原型对象，代之以一个新类型的实例。

混入模式（显式混入、隐式混入）可以用来模拟类的复制行为，但是很丑陋。而且显式混入无法完全模拟类的复制行为，因为对象只能复制对共享函数对象的引用。

## 1.1 类式继承（构造函数继承）
即在子类构造函数的内部调用父类构造函数，使得自身获得父类的方法和属性。
- 优点：
  - 可以定义私有属性方法
- 缺点：
  - 因为继承的是对象本身，每次实例化都保存内存中，有性能问题
  - 不能定义共享属性方法/或写在外面失去了封装性

```javascript
function Father() {
  this.name = name
  this.friends = friends // 👍 可以定义私有 引用类型不会被共享
  this.share = share // ❌ 可以定义公有 但需要放在外部
  this.log = log // ❌ 避免重复声明，为了复用需要放在外面。
}
// ❌ 公有属性和方法定义在外面失去了封装性
let share = [1, 2, 3]
function log() {
    return this.name
}

function Child(name) {
  Father.call(this, name)// 👍 可以在子类传递参数给父类
}

var a = new Child('a')
```

## 1.2 原型继承
即子类型从超类型的原型对象里继承方法  
- 优点：  
  - 父类的方法得到了复用，可以定义公有属性方法
- 缺点：
  - 不能定义私有属性方法
  - 如果父类包含引用类型的属性，那么所有子类的实例都会共享该属性
  - 在创建子类实例时，不能向父类的构造函数传递参数
  - 封装性一般

```javascript
function Father() {}
Father.prototype = {
    constructor: Father, 
    name: 'csy', // ❌ 不能定义私有属性，全部都是公有
    friends: ['alice', 'troy'], // 👍 可以定义公有属性 所有实例都引用这个
    log: function() { // 👍 方法被共享了
        return this.name
    }
}

function Child() {}

//ES6之前的写法
Child.prototype = Object.create(Father.prototype) //原型继承
Child.prototype.constructor = Child
//ES6写法 不过ES6就直接用class语法糖了，一般不用写原型继承
//Object.setPrototypeOf(Child.prototype, Father.prototype)

var a = new Child('a')
console.log(a.myName());
```

> 网上很多博客原型继承的方式是 `Child.prototype = new Father()`。<<你不知道的javascirpt>>书中说道这种用法有副作用。虽然会创建一个关联到 Child.prototype 的新对象，但是它使用了 Father(..) 的“构造函数调用”，如果函数 Father 有一些副作用（比如写日志、修改状态、注册到其他对象、给 this 添加数据属性，等等）的话，就会影响到 Child() 的“后代”。

## 1.3 组合继承
结合类式继承和组合继承，用类式继承属性，而原型继承方法。
- 优点：
  - 可以定义私有属性，引用属性不会被共享。私有的写在构造函数，公有的写在原型
  - 可以向父类传递参数
- 缺点：
  - 调用两次父类，性能损耗
  - 封装性一般

```javascript
function Father(name, friends) {
  // 😀 私有的写这里
  this.name = name // 👍 可以定义私有属性
  this.friends = friends // 👍 定义公有引用属性不会被共享
}

Father.prototype = {
  // 😀 公有的写这里
  share: [1, 2, 3], // 👍 这里定义的公有属性会被共享
  myName: function() { // 👍 方法被共享了
    return this.name
  }
}

function Child(name, friends) {
  Father.call(this, name, friends) // 👍 可以向父类传递参数 ⚡ 这里又调用了一次 Father
}

Child.prototype = new Father() //使用 new 操作符创建并重写 prototype ⚡ 这里调用了一次 Father
Child.prototype.constructor = Child

var a = new Child('a')
console.log(a.myName());
```
## 1.4 寄生组合继承
- 优点：
  - 可以定义私有属性，引用属性不会被共享。私有的写在构造函数，公有的写在原型
  - 可以向父类传递参数
  - 不会重复调用父类
- 缺点：
  - 封装性一般

```javascript
function Father(name, friends) {
  this.name = name
  this.friends = friends
}

Father.prototype = {
  share: [1, 2, 3],
  myName: function () {
    return this.name
  }
}

function Child(name, friends, gender) {
  Father.call(this, name, friends)
  this.gender = gender
}

// 上半部分和组合继承一样
Child.prototype = Object.create(Father.prototype) //原型继承
Child.prototype.constructor = Child
```

## ES6 class
看完前面的，再看es6的语法糖真是太甜了。
```javascript 
class Father {
    constructor(name, friends) { // 该属性在构造函数上，不共享
        this.name = name
        this.friends = friends
    }
    log() { // 该方法在原型上，共享
        return this
    }
}
Father.prototype.share = [1, 2, 3] // 原型上的属性，共享

class Child extends Father {
    constructor(name, friends, gender) {
        super(name, friends)
        this.gender = gender
    }
}
```

# 2. 原型链
[[Prototype]]机制就是对象中的一个内部链接引用另外一个对象。如果在第一个对象上没有找到需要的属性或者方法引用，引擎就会继续在[[Prototype]]关联的对象上进行查找，以此类推，这一系列对象的链接被称为"原型链"。

对于默认的 [[Get]] 操作来说，如果无法在对象本身找到需要的属性，就会继续访问对象的 [[Prototype]] 链。

for..in 遍历，in 操作符来检查属性在对象中是否存在时，会查找对象的整条原型链（无论属性是否可枚举）。

```javascript
var anotherObject = { a:2 };
var myObject = Object.create(anotherObject)
for(key in myObject) {
  console.log(key);
}
console.log("a" in myObject);
```

## 属性设置和屏蔽
> 应尽量避免使用屏蔽，以及注意隐式屏蔽的情况

- 原型链上的已有xxx非只读属性，会直接在myObject中添加一个新的属性xxx，它是屏蔽属性。
- 原型链上的已有xxx只读属性，则无法创建同名的xxx屏蔽属性。
- 原型链上的已有xxx但它是一个setter，则会调用这个setter，xxx不会被添加到myObject，也不会重新定义xxx。

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


## 参考
<<你不知道的javascirpt>>