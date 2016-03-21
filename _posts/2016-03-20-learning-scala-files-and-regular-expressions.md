---
title: scala语言学习-文件和正则表达式
layout: post
tags:
  - scala
---

## Files文件操作
- `import scala.io.Source`
- `Source.fromFile("myfile.txt", "UTF-8").getLines.toArray` 获得文件所有的行
- `Source.fromFile("myfile.txt", "UTF-8").mkString` 获得文件内容
- `Source.fromURL("http://horstmann.com", "UTF-8")` 直接读取网址内容（这里不安全，没有地方设置timeout属性）
- 写文本文件还是用`java.io.PrintWriter`:`val out = new PrintWriter("numbers.txt")`

文件的其他操作等用到了再去翻官方文档了

## 正则表达式
- `scala.util.matching.Regex`
- 字符串的`r`方法可以将字符串转换成正则表达式对象：`val numPat = "[0-9]+".r`
- 可以用原生字符串的写法，避免混乱的转义字符`val r = """\s+[0-9]\s+""".r`
- 正则表达式常用方法`val r = """\s+""".r`
    - `r.split("hello world") // Array("hello", "world")`
    - `r.findAllIn("hello world") // Iterator`
    - `r.findFirstIn("hello world") // Optional[String]`
    - `numPattern.replaceFirstIn("99 bottles, 98 bottles", "XX")`
    - `numPattern.replaceAllIn("99 bottles, 98 bottles", "XX")`
- 分组匹配
{%highlight scala%}
// Groups are useful to get subexpressions of regular expressions. Add parentheses around the subexpressions that you want to extract, for example:
val numitemPattern = "([0-9]+) ([a-z]+)".r
// To match the groups, use the regular expression object as an “extractor” (see Chapter 14), like this:
val numitemPattern(num, item) = "99 bottles"
// Sets num to "99", item to "bottles"
// If you want to extract groups from multiple matches, use a for statement like this:
for (numitemPattern(num, item) <- numitemPattern.findAllIn("99 bottles, 98 bottles"))
    process num and item
{%endhighlight%}    

