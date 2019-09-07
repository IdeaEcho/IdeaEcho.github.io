---
layout:     post
title:      "call、apply、bind的实现"
subtitle:   ""
author:     "csy"
header-img: "img/home-bg2.jpg"
header-mask:  0.5
catalog: true
tags:
- JavaScript
- 笔记
---

## call、apply、bind
- 都用来改变函数体内部的this指向
- 第一个参数都是this，如果没有第一个参数，this 的值绑定为全局对象
- 都可以利用后续参数传参
  - call：fn.call(this,1,2,3)
  - apply:fn.apply(this,[1,2,3])
  - bind:fn.bind(this)(1,2,3)

## call的实现
思路：
1、将函数设为对象的属性
2、执行该函数
3、删除该函数

```javascript
Function.prototype.myCall = function (context = window, ...rest) {
  context.fn = this
  let result = context.fn(...rest)
  delete context.fn
  return result
}
```

## apply的实现

```javascript
Function.prototype.myApply = function (context = window, params=[]) {
  context.fn = this
  let result
  if(params.length) {
    result = context.fn(...params)
  } else {
    result = context.fn()
  }
  delete context.fn
  return result
}
```

## bind的实现
思路：
1、返回一个函数
2、可以传入参数
3、bind 返回的函数作为构造函数的时候，bind 时指定的 this 值会失效，但传入的参数依然生效。

```javascript
Function.prototype.myBind = function (context, ...rest) {
    if (typeof this !== 'function') {
        throw new TypeError('invalid invoked!')
    }
    var self = this
    function F(...args) {
        if (this instanceof F) { //当作为构造函数时，this 指向实例，此时结果为 true
            return new self(...rest, ...args)
        }
        return self.apply(context, rest.concat(args))
    }
    delete F.prototype
    return F
}
```

