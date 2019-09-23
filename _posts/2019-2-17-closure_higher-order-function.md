---
layout:     post
title:      "闭包和高阶函数"
subtitle:   ""
author:     "csy"
header-img: "img/home-bg2.jpg"
header-mask:  0.5
catalog: true
tags:
    - JavaScript
    - 笔记
---

## 1. 高阶函数
什么是高阶函数？
- 函数可以作为参数传递
- 函数可以作为返回值输出

### 1.1 函数作为参数传递
即经常用到的回调函数
### 1.2 函数可以作为返回值输出
"无论通过何种手段将内部函数传递到所在的词法作用域以外，它都会持有对原始定义作用域的引用，无论在何处执行这个函数都会使用闭包。" -- 《你不知道的javascript》  

举例函数作为返回值输出：
##### :chestnut:1:偏函数
返回了一个包含预处理参数的新函数，以便之后可以调用。比如下面这个例子：简单的类型判断。

```js
function isType(type) {
  return function(obj) {
    return Object.prototype.toString.call(obj) === `[object ${type}]`
  }
}
let isArray = isType('Array')
let isString = isType('String')
console.log(isArray([1, 2, [3,4]]) // true
console.log(isString({})
```

##### :chestnut:2:预置函数
原理：当达到条件时再执行回调函数

```js
function after(time, cb) {
    return function() {
        if (--time === 0) {
            cb();
        }
    }
}
// 举个栗子吧，吃饭的时候，我很能吃，吃了三碗才能吃饱
let eat = after(3, function() {
    console.log('吃饱了');
});
eat();
eat();
eat();
```
