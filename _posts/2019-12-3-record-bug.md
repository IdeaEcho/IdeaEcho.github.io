---
layout:     post
title:      "记录一次兼容安卓4.2.x无报错信息的bug"
subtitle:   ""
author:     "csy"
header-img: "img/home-bg2.jpg"
header-mask:  0.5
catalog: true
tags:
    - 
    - 
---

## 先讲重点


```javascript
{
      path: "/",
      name: "home",
      component: () =>
        import( /* webpackChunkName: "home" */ "../home/home.vue")
    },
rel="prefetch"
```
prefetch
vue路由器的延迟加载，通过将import函数包装到箭头函数中，Vue将仅在请求时执行它，并在该时刻加载模块。

## 总结
同事请假了，修改他的刚上线的项目bug。

由于其他版本的安卓设备正常，而安卓4.2.x白屏，有所以定性为兼容性bug。
单页面应用白屏通常情况是dom树为空，一般是js报错导致的，但在设备上一看没有任何的报错信息。

后来排查发现在这台router-view
本地dev-server 正常，打包后才白屏

由于是线上问题，需要最快的解决方式，没有时间去深究原理。所以当时直接把路由懒加载去掉了。但是并没有结束，这个bug被我记到了TODOLIST，后来清理清单的时候又被捞起来了。

路由懒加载
```javascript
export default new Router({
  mode: "hash",
  base: process.env.BASE_URL,
  routes: [{
      path: "/",
      name: "home",
      component: () => import( /* webpackChunkName: "home" */ "../home/home.vue")
    }
  ]
})
```

<style scoped lang="scss">
p {
  color: red;
}
</style>

只要去掉所有的样式，

vue-loader 做了什么

