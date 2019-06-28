---
layout:     post
title:      "js事件循环"
subtitle:   ""
author:     "csy"
header-img: "img/home-bg2.jpg"
header-mask:  0.5
catalog: true
tags:
- JavaScript
- 笔记
---

看了一些有关事件循环中函数执行顺序的问题，这里记录一下。


## 事件循环运行机制：
Microtasks 和 Macrotasks 是异步任务的一种类型，Microtasks 的优先级要高于 Macrotasks，下面是它们所包含的api：
microtasks主要包含：process.nextTick、promise、MutationObserver
macrotask主要包含：script(整体代码)、setTimeout、setInterval、I/O、UI交互事件、postMessage、MessageChannel、setImmediate(Node.js 环境)

事件循环是js实现异步的核心，每轮事件循环分为以下步骤：
1. 执行一个宏任务（栈中没有就从事件队列中获取）
2. 执行过程中如果遇到微任务，就将它添加到微任务的任务队列中
3. 宏任务执行完毕后，立即执行当前微任务队列中的所有微任务（依次执行）
4. 当前事件循环执行完毕，开始检查渲染，然后GUI线程接管渲染
5. 渲染完毕后，JS线程继续接管，开始下一个宏任务（从事件队列中获取）

### Promise和async中的立即执行

```js
new Promise(resolve => {
    resolve(1);
    Promise.resolve().then(() => console.log(2));
    console.log(4)
}).then(t => console.log(t));
console.log(3);
```

这段代码的流程大致如下：

* script 任务先运行。首先遇到 Promise 实例，构造函数首先执行，所以首先输出了 4。此时 microtask 的任务有 t2 和 t1。
* script 任务继续运行，输出 3。至此，第一个宏任务执行完成。
* 执行所有的微任务，先后取出 t2 和 t1，分别输出 2 和 1。
* 代码执行完毕。

为什么 t2 会先执行呢？理由如下：
> Promise.resolve 方法允许调用时不带参数，直接返回一个resolved 状态的 Promise 对象。立即 resolved 的 Promise 对象，是在本轮“事件循环”（event loop）的结束时，而不是在下一轮“事件循环”的开始时。

总结：  
Promise中的异步体现在then和catch中，所以写在Promise中的代码是被当做同步任务立即执行的。  
在async/await中，await出现之前，其中的代码也是立即执行的。那么出现了await时候发生了什么呢？


### await做了什么

很多人以为 await 会一直等待之后的表达式执行完之后才会继续执行后面的代码，实际上await是一个让出线程的标志。async await 本身就是 promise + generator 的语法糖。所以 await 后面的代码是 microtask 。await 后面的表达式会先执行一遍，将 await 后面的代码加入到microtask中，然后就会跳出整个 async 函数来执行后面的代码。

```js
async function async1() {
	console.log('async1 start');
	await async2();
	console.log('async1 end');
}
```
等价于
```js
async function async1() {
	console.log('async1 start');
	Promise.resolve(async2()).then(() => {
    console.log('async1 end');
  })
}
```

### 视图渲染的时机
浏览器为了能够使得JS内部 macrotask 与DOM任务能够有序的执行，会在一个 macrotask 执行结束后，在下一个 macrotask 执行开始前，循环的 microtask 队列被执行完之后，对页面进行重新渲染，也就是说执行任务的耗时会影响视图渲染的时机。流程如下：
macrotask ->渲染-> macrotask ->...

但也不是每轮事件循环都会执行视图更新，浏览器有自己的优化策略，例如把几次的视图更新累积到一起重绘，重绘之前会通知 requestAnimationFrame 执行回调函数，也就是说 requestAnimationFrame 回调的执行时机是在一次或多次事件循环的 UI render 阶段。requestAnimationFrame 既不属于 macrotask, 也不属于 microtask。

浏览器只保证 requestAnimationFrame 的回调在重绘之前执行，没有确定的时间，何时重绘由浏览器决定。

验证代码：

```js
setTimeout(function() {
  console.log('timer1')
}, 0)

requestAnimationFrame(function() {
  console.log('requestAnimationFrame')
})

setTimeout(function() {
  console.log('timer2')
}, 0)

new Promise(function executor(resolve) {
  console.log('promise 1')
  resolve()
  console.log('promise 2')
}).then(function() {
  console.log('promise then')
})

console.log('end')
```


参考：
[从一道题浅说 JavaScript 的事件循环](https://github.com/dwqs/blog/issues/61)
