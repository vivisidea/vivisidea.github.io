---
layout: post
title: "测试中文字体在主题中的显示效果"
date: 2015-01-08 12:00:00
categories: jekyll
featured_image: /images/cover.jpg
---

## linux命令-sed(stream editor)
sed的全称是stream editor，原来这货也算是个编辑器，这应该算是一个很geek的编辑器了吧 :)

基本参数说明

`-n` 关闭默认打印pattern space，例如使用p命令时一般需要加这个参数关闭默认打印
-r 使用扩展的正则表达式，\d\s等
-i inplace模式，输入文件将直接被修改
基本用法


{% highlight bash %}
sed 's/my/your/g' input.txt 将my替换成your
sed 's/my/your/1' input.txt 只替换第一个匹配
sed -n '/echo/=' input.txt 打印出包含echo的行号
sed '1i #!/bin/bash' input.txt 在第一行插入一行内容
sed '/echo/i # insert before' 匹配echo的行之前插入内容
sed '/echo/a # insert after' 匹配echo的行之后插入内容
sed '/echo/c # replace the line' 将匹配echo的行替换成指定的行内容
sed -r '/\s*<[^>]*>\s*/ /g' html.txt 将html标签替换成空格，测试似乎不能使用非贪婪模式<.*?>表达
{% endhighlight %}

用法总结（不总结一下每次都要google命令。。）
s 命令，替换匹配

```
sed '[line-address1][,line-address2]s/pattern/replacement/[index|g]' input.txt
```

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

> 参考资料
> 简明sed教程 后面的hold space之类的概念瞄了一眼觉得应该用不着，先放着
> 如果后面还是中文
> 
> 如果后面还是中文
> 如果后面还是中文
> 如果后面还是中文
> 如果后面还是中文
> 
> 如果后面还是中文

{% highlight ruby %}
def show
  @widget = Widget(params[:id])
  respond_to do |format|
    format.html # show.html.erb
    format.json { render json: @widget }
  end
end
{% endhighlight %}

中间空格一行

Strange Case of Dr Jekyll and Mr Hyde is the original title of a novella written by the Scottish author [Robert Louis Stevenson]() that was first published in 1886. The work is commonly known today as The Strange Case of Dr. Jekyll and Mr. Hyde, Dr. Jekyll and Mr. Hyde, or simply Jekyll & Hyde. It is about a London lawyer named Gabriel John Utterson who investigates strange occurrences between his old friend, Dr. Henry Jekyll, and the evil Edward Hyde.

## Inspiration & Writing

Stevenson had long been intrigued by the idea of how personalities can affect a human and how to incorporate the interplay of good and evil into a story. While still a teenager, he developed a script for a play about Deacon Brodie, which he later reworked with the help of W. E. Henley and saw produced for the first time in 1882. In early 1884 he wrote the short story "Markheim", which he revised in 1884 for publication in a Christmas annual. One night in late September or early October 1885, possibly while he was still revising "Markheim," Stevenson had a dream, and upon wakening had the intuition for two or three scenes that would appear in the story. Biographer Graham Balfour quoted Stevenson's wife Fanny Stevenson:

> "In the small hours of one morning, I was awakened by cries of horror from Louis. Thinking he had a nightmare, I awakened him. He said angrily: 'Why did you wake me? I was dreaming a fine bogey tale.' I had awakened him at the first transformation scene."

## Main Characters

The story revolves around **8 main characters**:

- Dr. Henry Jekyll/Edward Hyde
- Mr. Gabriel John Utterson
- Richard Enfield
- Dr. Hastie Lanyon
- Mr. Poole
- Inspector Newcomen
- Sir Danvers Carew
- Maid

## Reception

Strange Case of Dr Jekyll and Mr Hyde was an immediate success and is one of Stevenson's best-selling works. Stage adaptations began in Boston and London and soon moved all across England and then towards his home Scotland within a year of its publication and it has gone on to inspire scores of major film and stage performances.

The Strange Case of Dr Jekyll and Mr Hyde was initially sold as a paperback for one shilling in the UK and one dollar in the U.S. The American publisher issued the book on 5 January 1886, four days before the first appearance of the UK edition issued by Longmans; Scribner's published 3000 copies, only 1250 of them bound in cloth. Initially stores would not stock it until a review appeared in The Times, on 25 January 1886, giving it a favourable reception. Within the next six months, close to forty thousand copies were sold. The book's success was probably due more to the "moral instincts of the public" than any perception of its artistic merits; it was widely read by those who never otherwise read fiction, quoted in pulpit sermons and in religious papers. By 1901 it was estimated to have sold over 250,000 copies.
