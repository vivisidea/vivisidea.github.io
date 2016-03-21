---
title: scala语言学习-数组Array
layout: post
tags:
  - scala
---

## Scala的数组
- 初始化固定数组`val arr = new Array[Int](10) // 长度为10，初始化为0` 注意这里的`new`
- 使用初始值初始化数组`val arr = Array(1,2,3,4) // 长度为4，初始值为1,2,3,4`
- 可变长度数组`val buffer = ArrayBuffer[Int]()// import scala.collection.mutable.ArrayBuffer` 注意这里又没有`new`
    - `buffer += 1 // ArrayBuffer(1)`
    - `buffer += (2,3,4) // ArrayBuffer(1,2,3,4)`
    - `buffer ++= Array(5,6,7) // append collection using ++=`
    - `buffer.toArray` 转换为固定数组
- 数组下标遍历`for(i <- 0 until arr.length)...`
- 反过来下标遍历`for(i <- (0 until arr.length).reverse)...`
- 直接遍历元素`for(elem <- arr)...`
- 数组变换`for(elem <- arr if elem ...) yield 2 * elem`
- 多维数组`val matrix = Array.ofDim[Double](3,4)` 定义多维数组，初始化为0，这里需要指定类型
    - 多维数组`val matrix = new Array[Array[Int]](10)` 定义数组的数组
