---
layout:     post
title:      "理解Object.defineProperty()笔记"
subtitle:   ""
author:     "csy"
header-img: "img/common/banner.jpg"
header-mask:  0.5
catalog: true
tags:
    - js
---

### 介绍
Object.defineProperty(obj, prop, descriptor)方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并返回这个对象。
这个方法允许修改默认的额外选项（或配置）。默认情况下，使用 Object.defineProperty() 添加的属性值是不可修改的。


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
