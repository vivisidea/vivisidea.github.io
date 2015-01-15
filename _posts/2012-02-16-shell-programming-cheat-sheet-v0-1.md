---
title: shell programming cheat sheet V0.1
author: vivi
layout: post
permalink: /posts/128.html
categories:
  - 技术记录
tags:
  - linux
  - shell
---
shell programming cheat sheet V0.1

1. 数值计算
{% highlight bash %}
a=0; (( a = $a +1 )); echo $a
{% endhighlight %}


2. 循环
{% highlight bash %}
for f in `ls`; do echo $f; done
for (( i=1; $i&lt;10; i++)); do echo $i ; sleep 1; done # 好亲切的for循环。。
a=0; while [ $a -lt 10 ]; do echo -n "$a "; (( a = $a+1 )); sleep 1; done ; echo;
a=0; while (( $a &lt; 10 )); do echo -n "$a "; (( a = $a+1 )); sleep 1; done ; echo;
{% endhighlight %}

3. 判断
{% highlight bash %}
if [ -d /usr/local/bin ] ; then echo "exists."; else echo "not exists."; fi
[ -d /usr/local/bin ] && echo "exists"       # if .. then
[ -d /user/local/bin ] || echo "not exists"  # if not .. then
# integers
-eq, -ne, -lt, -gt, -le, -ge
# string
str1 = str2, str1 != str2, -n (str not null, length&gt;0), -z (str is null, length=0)
# file conditions
-f(ile), -d(irectory), -s(ize not zero), -r(eadable), -w(ritable), -x(excutable)
# logical
cond1 -a cond2, cond1 -o cond2, ! cond1
{% endhighlight %}

4. sed
{% highlight bash %}
sed 's/one/two/g' &lt; in.file.txt &gt; out.file.txt
sed -r 's/[0-9]{3,}//g' &lt; in.file.txt  &lt;-- 注1
sed 's:old:new:g' &lt; in.file.text &gt; out.file.txt &lt;-- 注2
sed -r 's/[0-9]{3,}/(&)/g' &lt; sed.txt
sed -r 's/([0-9]+)\s+([a-z]+)/\2 \1/g' &lt; sed.txt &lt;-- 注3
sed -n -r '/\S+/p' &lt; sed.txt &lt;-- 注4
sed -r '/^$/d' &lt; sed.txt
sed -n -r 's/([0-9]+)\s+([a-z]+)/\2 \1/gpw test.txt' &lt; sed.txt &lt;-- 注5
sed -r -e 's/([0-9]+)\s+([a-z]+)/\2 \1/g' -e '/^$/d' &lt; sed.txt  &lt;--注6
sed -i '1i\line to insert' input_file # insert a line at the beginning of the file
sed -i '1 d' input_file # delete first line
# 注1: -r参数启用extended regular expression
# 注2: 分割符号可以是 / , :  ,  _ , |
# 注3: 如果不启用 -r 那么'('还得写成'\('，很恶心
# 注4: sed默认会打印每一行（不论有没匹配），-n表示默认不打印，后面的p表示如果匹配了那执行打印该行
# 注5: /p 打印 /g 全局 /d 删除 /w file 结果写到文件，可以组合多个command
# 注6: 多条sed指令用-e分开
{% endhighlight %}

5. function
{% highlight bash %}
#!/bin/bash
function logger(){
  local MSG=$1
  local TIMESTAMP=$(date '+%F %H:%M:%S')
  echo "$TIMESTAMP: $MSG"
}
logger "this is output for test"
{% endhighlight %}

6. the GAME ...
{% highlight bash %}
#!/bin/bash
# generate a random integer and guess it
MAGIC=$(( $RANDOM % 100 ))
LOWER=0
UPPER=100
while true; do
    echo -n "$LOWER to $UPPER: "
    read INPUT
    if [ $INPUT -lt $MAGIC ] ; then
        LOWER=$INPUT
    elif [ $INPUT -gt $MAGIC ] ; then
        UPPER=$INPUT
    else
        echo "yep, the magic is $MAGIC"
        break
    fi
done
{% endhighlight %}

7. 处理输入参数
{% highlight bash %}
#!/bin/bash
SEC=0
MIN=0
HOUR=0
while getopts :s:m:h: INPUT 2&gt;/dev/null; do
    case $INPUT in
      s) SEC=$OPTARG
        ;;
      m) MIN=$OPTARG
        ;;
      h) HOUR=$OPTARG
        ;;
      *) echo "usage $(basename $0) -s SEC -m MIN -h HOUR"
         exit 1
        ;;
    esac
done
echo "$HOUR:$MIN:$SEC"
{% endhighlight %}

8. process-file-line-by-line
{% highlight bash %}
while read LINE; do
  echo "$LINE" &gt;&gt; $OUTFILE # do something to the line
done &lt; $INFILE
{% endhighlight %}

9. 访问数组
{% highlight bash %}
#!/bin/bash
# vivi@2012-02-17_00:03:04
# chars=({a..z} {A..Z} {0..9})       # 大括号展开，例如 cp file{,.bak}
# count=${#chars[@]}                 # 获得数组长度
chars=
count=0
line=""
for char in {a..z} {A..Z} {0..9}; do # {}展开，shell好像没用全大写做变量的习惯
  chars[$count]=$char                # 数组赋值
  line="$line${chars[$count]}"       # 访问数组元素
  (( count = $count + 1 ))
done
echo $line
{% endhighlight %}

10. find的用法
{% highlight bash %}
通常我会这么用
for f in $(find . -type f -name *.txt); do
    echo $f;
done
但其实可以这么用
find . -type f -name *.txt -exec echo '{}' \;
find . -type f -name *.txt -exec grep --color=auto "abcd" '{}' +
find . -type f -name *.txt | xargs echo
# '{}' 会被替换成符合条件的文件名
# \;一条条结果分别传给command处理; +是将结果拼成一个字符串后一起传给command处理
{% endhighlight %}
