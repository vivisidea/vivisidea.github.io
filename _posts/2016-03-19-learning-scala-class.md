---
title: scala语言学习-Class
layout: post
tags:
  - scala
---

## Class
- 类的`var`属性自动获得`getter`和`setter`方法, `val`属性自动获得`getter`方法
- 使用`@BeanProperty val/var name`来生成`JavaBean`的`getter`和`setter`(var)方法
- 主构造函数，这个构造函数的参数自动变成这个class的对应字段， `class Person(val/var name: String, val/var age: Int){...}`
- 主构造函数的参数可以有默认值`class Person(val name: String = "", val age: Int = 0)`
- 辅助构造函数？`def this(name: String)`，需要调用主构造函数`this()`，或者另一个辅助构造函数
- class不需要定义为`public`，一个Scala文件中可以有多个class定义，并且都是`public`的
- 只读属性
    - 如果构造之后不再修改，那么使用`val name`定义
    - 如果构造之后需要修改，那么使用`private var name`定义，另外还需要手动增加一个`getter`方法
- 可以重新定义`getter`和`setter`
{%highlight scala%}
class Person {
    private var privateAge = 0 // Make private and rename，需要重新定义的时候，自定名就不能定为 age 了
    def age = privateAge // 重新定义getter
    def age_=(newValue: Int) { // 这里重新定义setter，注意这里的 _ 是和 age 一起的，并不是和 = 一起的
        if (newValue > privateAge) privateAge = newValue; // Can’t get younger
    }
}
{%endhighlight%}

