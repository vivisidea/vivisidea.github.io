---
title: linux命令-sed(stream editor)
author: vivi
layout: post
permalink: /posts/334.html
categories:
  - 技术记录
tags:
  - linux shell sed
---
linux命令-sed(stream editor)  
sed的全称是stream editor，原来这货也算是个编辑器，这应该算是一个很geek的编辑器了吧 :)

基本参数说明

- -n 关闭默认打印pattern space，例如使用p命令时一般需要加这个参数关闭默认打印
- -r 使用扩展的正则表达式，\d\s等
- -i inplace模式，输入文件将直接被修改


基本用法
{% highlight bash %}
sed 's/my/your/g' input.txt 将my替换成your
sed 's/my/your/1' input.txt 只替换第一个匹配
sed -n '/echo/=' input.txt 打印出包含echo的行号
sed '1i #!/bin/bash' input.txt 在第一行插入一行内容
sed '/echo/i # insert before' 匹配echo的行之前插入内容
sed '/echo/a # insert after' 匹配echo的行之后插入内容
sed '/echo/c # replace the line' 将匹配echo的行替换成指定的行内容
sed -r '/\s*&lt;[^&gt;]*&gt;\s*/ /g' html.txt 将html标签替换成空格，测试似乎不能使用非贪婪模式&lt;.*?&gt;表达
{% endhighlight %}

用法总结（不总结一下每次都要google命令。。）
{% highlight bash %}
s 命令，替换匹配

sed '[line-address1][,line-address2]s/pattern/replacement/[index|g]' input.txt
替换line1-line2($表示最后一行)的第index个pattern为replacement（g表示全部替换，[index]g表示替换第index个到最后的匹配）

其中line-address可以是一个具体的行，也可以是一个匹配，例如
sed -n '/{/,/}/p' input.txt 打印包含{的行到包含}的行

i/a/c 命令，插入，追加，修改匹配行

sed '[line-address1][,line-address2][a|i|c] new-content' input.txt
sed '/pattern/[a|i|c] new-content' 以匹配的行为基准append/insert/change/delete内容。
sed '1,4c #line replaced' input.txt c模式下是把1-4行替换成一行新内容，而不是每行分别替换。

p/d 命令，打印/删除匹配行
与i/a/c命令类似，只是不需要提供[new-content]

一次处理多个匹配

sed '1,3s/my/your/g; 3,$/this/that/g' input.txt 等价于
sed -e '1,3s/my/your/g' -e '3,$/this/that/g' input.txt

命令嵌套打包
sed '1,${/this/d; s/^ *//g}' input.txt 如果匹配到this那么删除，否则删除行开头空格
{% endhighlight %}

参考资料

- [简明sed教程](http://coolshell.cn/articles/9104.html) 后面的hold space之类的概念瞄了一眼觉得应该用不着，先放着
