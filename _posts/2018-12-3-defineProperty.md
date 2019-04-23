---
layout:     post
title:      "理解Object.defineProperty()笔记"
subtitle:   ""
author:     "csy"
header-img: "img/common/banner.jpg"
header-mask:  0.5
catalog: true
tags:
    - JavaScript
    - 笔记
---

### 介绍
Object.defineProperty(obj, prop, descriptor)方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回这个对象。
这个方法允许修改默认的额外选项（或配置）。默认情况下，使用 Object.defineProperty() 添加的属性值是不可修改的。

Object.defineProperty()和点运算符为对象的属性赋值时，数据描述符中的属性默认值是相反的。

### 属性描述符
> 不能同时是两者

- 数据描述符
- 存取描述符

### 示例

```js
// 创建一个新对象
var obj = {
  test: "Data"
}
//数据描述demo
//对象已有的属性添加特性描述
Object.defineProperty(obj, "test", {
  configurable: true | false, //默认为false
  enumerable: true | false,//默认为false
  value: '任意类型的值', //默认为undefined
  writable: true | false //默认为false
});

console.log(obj) //{test: "任意类型的值"}

//存取器描述demo
//对象新添加的属性的特性描述
var initValue = 'hello'
Object.defineProperty(obj, "newKey", {
  get: function() {
    //当获取值的时候触发的函数
    return initValue
  },
  set: function(newValue) {
    //当设置值的时候触发的函数,设置的新值通过参数value拿到
    if (initValue != newValue) {
      initValue = newValue
    }
  },
  configurable: true | false, //默认为false
  enumerable: true | false,//默认为false
});
console.log(obj) //{test: "任意类型的值"}
console.log(obj.newKey) //hello
obj.newKey = 1
console.log(obj.newKey)
```

### 继承

```js
function myClass() {}
var value;
Object.defineProperty(myClass.prototype,"x",{
  get() {
    return value
    // return this.value;
  },
  set(x) {
    value = x
    // this.value = x;
  },
  configurable: true
})

var a = new myClass()
var b = new myClass()
a.x = 1
console.log(a.x);
console.log(b.x);

var descriptor = Object.create(null) //Object.create(null)将__proto__属性指向null
Object.defineProperty(myClass.prototype, 'x',descriptor)

console.log(b.x);

```
最后一个b.x输出的还是1，Object.create(null)，默认没有 enumerable，没有 configurable，没有 writable，但不会重置get和set。
