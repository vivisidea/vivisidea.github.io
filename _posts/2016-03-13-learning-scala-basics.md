---
title: scala语言学习-语法和Java的区别
layout: post
tags:
  - scala
---

走出自己的comfort zone之学习一门新的语言，先从JVM语言开始，Scala因Apache Spark项目而流行起来，提供了一个实时数据处理和机器学习的平台。Scala语法相较于Java而言简洁许多，偏向脚本语言。

以下笔记来自《Scala for the Impatient》部分记录，记录一个Java程序员视角的主要区别。

在操作中学习是一个非常好的方式，REPL is your friend.

{%highlight bash%}
vivi@ssd:/tmp$ scala
Welcome to Scala version 2.11.6 (Java HotSpot(TM) 64-Bit Server VM, Java 1.7.0_79).
Type in expressions to have them evaluated.
Type :help for more information.

scala> println("hello Scala")
hello Scala
{%endhighlight%}

## 基本语法
- `val`关键字定义常量， `var`关键字定义变量
- 常量/变量定义时可以指定类型，类型在变量名后面： `val msg:String = "hello"`
- 没有基本类型，一切都是对象， `1.toString`调用是合法的
- 操作符其实也是方法 `a.+(b)`
- `a.method(b)`可以写成`a method b`，但是有多个参数的时候还是要写成`a.method(p1, p2 ..)`
- 没有参数的方法调用通常省略括号，例如`"Hello".length`等价于`"hello".length()`
	- 通常来说只有不修改对象的无参方法调用才省略括号
- 没有`++/--`操作符，用`+=1/-=1`代替
- `import scala.math._ // In Scala, the _ character is a “wildcard,” like * in Java`
- 没有静态方法，有singleton对象
- 使用apply函数（可省略）来构造对象是常见的方式： `BigInt("1234")`等价于`BigInt.apply("1234")`
- Array元素的访问方式是 `array($index)`, 如`array(1), "hello"(0)`（实际上也是方法调用，省略了apply方法）
- 数字类型需要关注的类是`RichInt, RichDouble`, 字符串类型需要关注的类型是`StringOps`，另外还关注 `Range, Seq`
- 函数(Function)可以作为参数`def count(p:(Char) => Boolean):Int`，例如`"Hello".count(_.isUpper)// 返回1`
- `if/else`语句是有返回值的`val s = if( x > 0 ) 1 else -1`
- `{}`也是有返回值的：`val distance = { val dx = x - x0; val dy = y - y0; sqrt(dx * dx + dy * dy) }`
- 循环语法：`while( expr ) ...`， `for( i <- expr )`，没有`break/continue`语句。。有别的办法可以实现，蛋疼
	- 高级循环1， `for( i <- 1 to 10 if i % 2 == 0 ) println(i)`
	- 高级循环2, `val col = for( i <- 1 to 10 if i % 2 == 0 ) yield i`， `yield`语句收集符合条件的元素生成`Vector`

## 函数
- 函数定义`def abs(x:Double) = if ( x > 0 ) x else -x`，非递归函数省略返回类型
- 函数不需要`return`语句，直接用表达式作为返回
- 参数可以有默认值`def hello(s:String = "default") = println(s)`，这里调用如果使用默认值，也是需要`()`的:`hello()`
- 可变参数`def sum(args:Int*) ={ var res = 0; for( i <- args ) res += i; res }` 参数实际是`Seq`类型
    - 这里参数虽然是`Seq`，`sum(x)`只有一个参数时，`x`必须是一个`Int`，如果要把`Seq`作为单个参数，需要写成`sum(1 to 10 : _*)`不要问我为什么。。
- lazy变量，直到被使用时才会初始化：`lazy val content = scala.io.Source.fromFile("/tmp/file.txt").mkString`
- 没有`checked exception`，异常处理语法如下
{%highlight scala%}
try {
    process(new URL("http://horstmann.com/fred-tiny.gif"))
} catch {
    case _: MalformedURLException => println("Bad URL: " + url)
    case ex: IOException => ex.printStackTrace()
}
{%endhighlight%}
- `try { ... } catch { ... } finally { ... }`句式依旧可用
- 函数定义可以嵌套
{%highlight scala%}
// 快速排序函数示例
def sort(xs: Array[Int]) {
    def swap(i: Int, j: Int) {
        val t = xs(i); xs(i) = xs(j); xs(j) = t
    }
    def sort1(l: Int, r: Int) {
        val pivot = xs((l + r) / 2)
        var i = l; var j = r
        while (i <= j) {
            while (xs(i) < pivot) i += 1
            while (xs(j) > pivot) j -= 1
            if (i <= j) {
                swap(i, j)
                i += 1
                j -= 1
            }
        }
        if (l < j) sort1(l, j)
        if (j < r) sort1(i, r)
    }
    sort1(0, xs.length - 1)
}
{%endhighlight%}
- 函数式编程
{%highlight scala%}
def sort(xs: Array[Int]): Array[Int] = {
    if (xs.length <= 1) xs
    else {
        val pivot = xs(xs.length / 2)
        Array.concat(
            sort(xs filter (pivot >)), // def >(x: Byte): Boolean 操作符号也是function
            xs filter (pivot ==),
            sort(xs filter (pivot <)))
    }
}
{%endhighlight%}
- 以函数作为参数的函数，或者返回函数的函数，被称为高阶函数(higher-order)
- 可以给一个`expression`命名，`expression`的值只有到被使用时才会计算，可以认为是别名，效果实际上是替换`{expression}`
    - `def h(i:Int):Int = { println("i = "+i); i }`
    - `def s = 3 * h(5) // 这里并不会打印出 i = 5`，注意和`val/var s = 3 * h(5)`的区别
- [by name vs by value](http://stackoverflow.com/questions/13337338/call-by-name-vs-call-by-value-in-scala-clarification-needed)

书上也是说成`call-by-value/name`但其实理解下来应该描述成`pass-by-value/name`比较合适，看demo代码就清楚了
{%highlight scala%}
scala> def f(x:Int, y:Int) = { println("f is called!"); x+y }
f: (x: Int, y: Int)Int

scala> def by_value(x:Int) = x + x
by_value: (x: Int)Int

scala> def by_name(x: =>Int) = x + x // 注意冒号后面的空格，不可省略写成x:=>
by_name: (x: => Int)Int

scala> by_value(f(3,2)) // by_value好理解，就是先把函数调用的值计算出来，传给by_value
f is called!
res9: Int = 10

scala> by_name(f(3,2)) // by_name我的理解就是把x全部替换成f(3,2)的效果，类似说x是f(3,2)的别名
f is called!
f is called!
res10: Int = 10

// 利用这个特性可以做一些神奇的事情，比如自己实现一个while循环
scala> :paste
// Entering paste mode (ctrl-D to finish)

def _while(cond: =>Boolean)(f: =>Unit){
  if(cond){
    f
    _while(cond)(f)
  }
}

// Exiting paste mode, now interpreting.

_while: (cond: => Boolean)(f: => Unit)Unit

scala> var c=5
c: Int = 5

scala> _while(c>0){ println(c); c-=1 }
5
4
3
2
1

{%endhighlight%}

## Pattern Matching
{%highlight scala%}
def fibonacci(in: Any): Int = in match {
    case 0 => 0
    case 1 => 1
    case n: Int if n > 1 => fibonacci(n - 1) + fibonacci(n - 2)
    case n: String => fibonacci(n.toInt)
    case _ => 0
}
{%endhighlight%}



