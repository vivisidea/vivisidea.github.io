---
title: scala语言学习-scala中的神奇的下划线_
layout: post
tags:
  - scala
---
这几天在学scala，发现语法定义还是有点多的，看到下划线(underscore)没错就是那个`_`的时候，混乱了，因为scala中的下划线用法太多了，以下尝试列举看到的几个

## Import
{%highlight scala%}
import scala.util._ // 这里是相当于Java里面的import java.lang.*;语法
import java.util.{HashMap => _, _} // 引入java.util._并且隐藏java.util.HashMap，避免和scala的HashMap冲突，还有这设定，好累
{%endhighlight%}

## Pattern Matching
{%highlight scala%}
def matchx(x:Int):String = x match {
    case 1 => "one"
    case 2 => "two"
    case _ => "anything else" // 匹配其他值
}
{%endhighlight%}

## 匿名函数的参数占位符
这也许是最容易混乱的部分了
{%highlight scala%}
scala> val arr = Array(1,2,3,4,5,6,7,8,9,10)
arr: Array[Int] = Array(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

scala> arr.filter(_%2==0)
res44: Array[Int] = Array(2, 4, 6, 8, 10)

// 说明
// filter这个方法定义是这样的
def filter(p: (T) => Boolean): Array[T]
// 也就是说，这个方法调用实际上补全了是这样的
arr.filter((x:Int) => x % 2 == 0)
// 由于强大的编译器可以自动推导出匿名函数的参数类型，所以这里的类型不是必要的，可以简化成这样
arr.filter((x) => x % 2 == 0)
// 语言作者觉得，匿名函数费事敲那么多代码搞毛阿，干脆直接留个函数体得了，至于参数么，那就用万能的下划线 _ 好了！
arr.filter(_%2==0)
// 值得注意的是，多个参数的函数一样可以用下划线表达，例如
arr.reduceLeft((a:Int, b:Int) => a+b)
// 直接写成
arr.reduceLeft(_+_)
// 就完事了
{%endhighlight%}

## 函数引用赋值
{%highlight scala%}
def fun(a:Int, b:Int) = a+b
def funref = fun // 这么写会报错，需要写成以下这两种其一
def funref = fun _
def funref = fun(_, _)
{%endhighlight%}

## 其他
{%highlight scala%}
for(_ <- 1 to 10) println("ha") // 忽略参数，意思是我只要循环次数，不需要这个参数
f(xs: _*) // 数组xs作为参数，调用可变参数函数f
{%endhighlight%}
关于Scala中的下划线，还有个很geek的[冷调查](http://www.scala-lang.org/old/node/5496)
{%highlight scala%}
Can you name all the uses of "_"?
-------------------------------------
Yes ========(7%)
 No =========================(40%)
  _ =====================================(52%)
{%endhighlight%}

以上列举的用法肯定不全，所以

`To Be Continued ...`

