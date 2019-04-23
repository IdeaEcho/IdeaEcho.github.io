---
layout:     post
title:      "js检测浏览器切换到后台的方案"
subtitle:   ""
author:     "csy"
header-img: "img/common/banner.jpg"
header-mask:  0.5
catalog: true
tags:
    - Web
    - JavaScript
---

## 需求

游戏中播放背景音效，如果手机切换到其他程序，暂停背景音乐。手机回到游戏页面，播放背景音乐。

### H5引入的Page Visibility API
这个 API 本身非常简单，由以下三部分组成。
- document.hidden：表示页面是否隐藏的布尔值。页面隐藏包括页面在后台标签页中 或者 浏览器最小化（注意，页面被其他软件遮盖并不算隐藏，比如打开的 sublime 遮住了浏览器）

- document.visibilityState：表示下面 4 个可能状态的值

- visibilitychange 事件：当文档从可见变为不可见或者从不可见变为可见时，会触发该事件。
1. hidden：页面在后台标签页中或者浏览器最小化
2. visible：页面在前台标签页中
3. prerender：页面在屏幕外执行预渲染处理 document.hidden 的值为 true
4. unloaded：页面正在从内存中卸载

这样，我们可以监听 visibilitychange 事件，当该事件触发时，获取 document.hidden 的值，根据该值进行页面一些事件的处理。
### js实现方法

```js
document.addEventListener("visibilitychange", e => {
  if (document.hidden) {
    if (this.openMusic) {
      window.closeMusicBeforeLeave = true;
      // TODO:关闭背景音乐
    }
  } else {
    if (window.closeMusicBeforeLeave) {
      // TODO:开启背景音乐
      window.closeMusicBeforeLeave = false;
    }
  }
});
```
