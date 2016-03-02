---
title: linux commands cheat sheet V0.1
author: vivi
layout: post
permalink: /posts/167.html
categories:
  - 技术记录
tags:
  - command
  - linux
---
收集在用的linux命令/工具，备忘，不定期更新

## commands/tools

	enca -L zh_CN filename                           # 检查文件的编码，比file要精确一些
	iconv -f gbk -t utf8 ingbk_file -o oututf8_file  # 转换文件内容的编码
	convmv -f gbk -t utf8  filename                  # 转换文件名的编码，确认之后加 --notest
	wget -mk http://www.example.com/                 # 镜像一个站点
	sudo blkid                                       # 查看分区的uuid
	csplit -f part_ -n 2 /--/ file.txt {*}           # 按照内容分割文件
	rename 's/\.bak$//' *.bak                        # 去掉所有.bak文件名后缀，跟sed差不多用法
 
## useful package installation

	sudo apt-get install ubuntu-restricted-extras  # 安装ubuntu受限资源
	sudo apt-get install nautilus-open-terminal    # nautilus文件管理器，右键open terminal
