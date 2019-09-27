---
layout:     post
title:      "前端存储之IndexedDB"
subtitle:   ""
author:     "csy"
header-img: "img/home-bg2.jpg"
header-mask:  0.5
catalog: true
tags:
- IndexedDB
- 笔记
---

# 前言 - 前端存储

##### Cookie

存储空间只有4k，存储时间有限制，每次请求都会带上Cookie，而且最重要的是，Cookie设计之初就不是就是让我们前端存数据用，而是为了让网站验证用户身份用的。

##### Web Storage

存储空间有5M大小，有永久存储的localStorage和会话期间存储的sessionStorage。对比Cookie优势很明显，但现在web不断发展，5M对于一些大型SPA单页应用来说不够，而且web Storage只能存储string类型的数据。对于Object类型的数据只能先用JSON.stringify()转换一下在存储。

##### IndexedDB

**存储更大量**的结构化数据以供离线使用，实现高性能搜索，也遵守**同源策略**，IndexedDB执行的操作是**异步执行**的。之前的世界观里只有关系型数据库，而IndexedDB是一个基于**事务**操作的key-value型前端数据库。不仅可以储存字符串，还可以储存二进制数据（ArrayBuffer 对象和 Blob 对象）。

# 基础概念

我们知道通常的数据库，可以有很多表，表的结构有唯一标识数据的主键，还可以建立索引加快对表中记录的查找和排序等。那IndexedDB呢？

IndexedDB是一个比较复杂的API，它把不同的实体，抽象成一个个对象接口。它的“表”是一个对象，称为对象仓库：IDBObjectStore 对象。每个对象仓库都有一个索引(index)集合以方便查询和迭代遍历。它的主键也是一个对象，称为主键集合：IDBKeyRange 对象。

通常，新建数据库以后，第一件事是新建对象仓库（即新建表）。

```javascript
request.onupgradeneeded = function(e){
    db = e.target.result;
    // 如果不存在Users对象仓库则创建
    if(!db.objectStoreNames.contains('Users')){
        var store = db.createObjectStore('Users',{keyPath: 'id', autoIncrement: true});
        //创建索引
        var idx = store.createIndex('index','username',{unique: false})
    }
}
```
createObjectStore新建对象仓库时设置id为主键，并设置autoIncrement为true，指定主键为一个递增的整数。  
createIndex的三个参数分别为索引名称、索引所在的属性、配置对象（说明该属性是否包含重复的值）。

# 实践

:sparkles: [github地址](https://github.com/IdeaEcho/demo/tree/master/src/packages/IndexedDB)

:sparkles: [demo戳这里](https://htmlpreview.github.io/?https://github.com/IdeaEcho/demo/blob/master/release/view/index.html#/IndexedDB)

# 初衷
从学生到工作写了几年的代码，觉得虽然学习新语言，新框架的主要目的是为了解决实际问题，其中更重要的是要每次入门了一门新技术后要及时留下点学习的痕迹，方便以后忘记可以从学习轨迹中迅速上手。