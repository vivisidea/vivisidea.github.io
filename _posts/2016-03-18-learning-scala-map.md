---
title: scala语言学习-Map
layout: post
tags:
  - scala
---

## Map
- 构造 `val m = Map("Alice", 1, "Bob"->2)`
- 默认的Map是imutable的，如果需要mutable的Map，需要引入`scala.collection.mutable.Map`
- 如果需要空的Map，可以指定一种具体的实现`val scores = new scala.collection.mutable.HashMap[String, Int]`
- `->`符号就是创建`pair`的意思，`"Alice"->1`和`("Alice", 1)`是等价的
- 访问`Map`对应key的值还是用`()`符号，如`m("Alice")`
- `()`访问，如果`key`不存在，会直接报错，可以用`m.getOrElse("not-exist-key", defaultValue)`
- `m.get(key)`返回的是一个`Option`对象，值要么是`Some`或者是`None`
- 删除一个key的操作是`-=`，例如`m -= "Alice"`
- 添加一些元素可以用`m += ("Tom"->3, "Lisa"->4)`
- 遍历元素`for((k,v) <- m) ...`
- `yield`语句也适用于`pair`，`for((k,v)<-m) yield (v,k)`
- `keys.zip(values).toMap` 建立两个数组的映射，返回一个`Map`对象

## Tuple
- `val _4tuple = ("Alice", 1, 2, 3)` 定义多元组
- `_4tuple._1` 元组的第一个元素`"Alice"`


