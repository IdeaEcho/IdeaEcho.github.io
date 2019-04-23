---
layout:     post
title:      "object、param、embed等视频音频标签"
subtitle:   ""
author:     "csy"
header-img: "img/home-bg2.jpg"
header-mask:  0.5
catalog: true
tags:
    - Web
---

## 1、&lt; object &gt;

定义一个**嵌入的对象**，比如图像、音频、视频、Java applets、ActiveX、PDF 以及 Flash。

#### 1.1 示例

```html
<object data="move.swf" type="application/x-shockwave-flash">
  <param name="foo" value="bar">
</object>
```

#### 1.2 object的属性
-   data 一个合法的 URL 作为资源的地址，需要为 data 和 type 中至少一个设置值。
-   classid 对象实现的 URI，可以同时与 data 属性使用，或者使用 data 属性替代
-   codebase 解析 classid，data 或者 archive 中定义的相对路径的根路径，如果没有定义，默认为当前文档的 base URI。

## 2、&lt; param &gt;

为object标签提供**嵌入内容的运行时参数的name与value对**

#### 2.1 param的参数

- **name为wmode**  
**window:**最顶层，遮住父div的背景和兄弟div  
**oqaque:**不透明的，遮住父div的背景，不遮兄弟div  
**transparent:**透明的 背景色alpha值将为0并且只会绘制stage上真实可见的对象，同样也可以使用z-index来控制flash影片的深度值  

- **name为AllowScriptAccess**  
当 AllowScriptAccess 设置为 "never" 时，运行对外脚本会失败；  
当 AllowScriptAccess 设置为 "always" 时，可以成功运行对外脚本；  

## 3、&lt; embed &gt;

HTML 5 中的新标签，和object一样是表示引入一个外部资源，不同的是它定义**嵌入的内容**，表示引入一个外部资源，比如资源可能是一张图片，一个嵌入的浏览上下文，亦或是一个插件所使用的资源。慎用!不同浏览器之间显示有差异。Blink 内核浏览器（Chrome，Opera）会显示 HTML 资源的内容，但 Firefox 会显示一条通知消息，指出内容需要一个插件（见 bug 1237963）。建议使用 <object> 或 <iframe> 元素。

## 4、视频流Demo
```html
<object width="100%" height="100%"
:data="curUrl"
classid="clsid:D27CDB6E-AE6D-11cf-96B8-444553540000"
codebase="http://download.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#version=9,0,28,0">
  <param name="quality" value="high">
  <param name="bgcolor" value="#000000">
  <param name="allowscriptaccess" value="always">
  <param name="allowfullscreen" value="true">
  <param name="wmode" value="opaque">
  <param name="allowFullScreenInteractive" value="true">
  <param name="flashvars" value="uuid=edfed58b-9e06-4cef-a9d0-c19c4a83e2a2&amp;quality=normal&amp;file=87ac2ba8bc11b7a322b27a9f12a07b7d">
  <param name="movie" value="curUrl">
  <embed width="100%" height="100%"
  name="loadingframe"
  type="application/x-shockwave-flash"
  src="curUrl"
  wmode="opaque"
  bgcolor="#000000"
  quality="high"
  pluginspage="http://www.adobe.com/shockwave/download/download.cgi?P1_Prod_Version=ShockwaveFlash"
  allowscriptaccess="always">
</object>
```

## 3、&lt; audio &gt;

标签是 HTML 5 的新标签。audio标签定义声音，比如音乐等其它音频流。

1、部分手机浏览器无法实现自动播放

> 这个现象产生的原因是：部分浏览器考虑了安全问题（偷跑流量），所以必须用户交互后才能播放。
> 用户触发 可以同时播放两个音频（部分安卓设备会忽略同时播放时的某个声音），但是如果第二个音频在async/await之后就无法播放。

2、Audio sprite，它的原理类似于CSS sprite。

3、多个音频文件预加载，跳转客户端登录sdk后不登录就返回，会自动播放音频
