---
title: linux commands cheat sheet V0.1
author: vivi
layout: post
tags:
  - command
  - linux
---
收集在用的linux命令/工具，备忘，不定期更新

## commands/tools
```bash
enca -L zh_CN filename                           # 检查文件的编码，比file要精确一些
iconv -f gbk -t utf8 ingbk_file -o oututf8_file  # 转换文件内容的编码
convmv -f gbk -t utf8  filename                  # 转换文件名的编码，确认之后加 --notest
wget -mk http://www.example.com/                 # 镜像一个站点
sudo blkid                                       # 查看分区的uuid
csplit -f part_ -n 2 /--/ file.txt {*}           # 按照内容分割文件
rename 's/\.bak$//' *.bak                        # 去掉所有.bak文件名后缀，跟sed差不多用法
```
	
## .ssh/config
```bash
Host gateway
    Hostname gateway-server-ip
    User user
    Port 22
    
Host *.xxx
    User user
    Port 22
    ProxyCommand ssh gateway exec nc -w 10 %h %p 2>/dev/null
```

## find
```bash
#通常我会这么用
for f in $(find . -type f -name *.txt); do
    echo $f;
done
#但其实可以这么用
find . -type f -name *.txt -exec echo '{}' \;
find . -type f -name *.txt -exec grep --color=auto "abcd" '{}' +
find . -type f -name *.txt | xargs echo
# '{}' 会被替换成符合条件的文件名
# \;一条条结果分别传给command处理; +是将结果拼成一个字符串后一起传给command处理
```
