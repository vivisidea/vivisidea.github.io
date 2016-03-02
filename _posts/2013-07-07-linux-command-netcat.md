---
title: linux命令-nc(netcat)
author: vivi
layout: post
permalink: /posts/321.html
categories:
  - 技术记录
tags:
  - command
  - linux
---
linux命令-nc(netcat)

今天才在《高性能MySQL》书里看到这个好用的linux命令行工具，有了这个工具之后服务器之间临时要传个文件什么的，会变得很简单

先看下没有这个命令之前我是怎么在两台服务器之间传输文件的

1. 不需要ssh的情况下，使用python的SimpleHTTPServer和wget &#8230;

{% highlight bash %}
# 先到源服务器，cd到要传输文件所在的目录，利用python自带的HTTPServer模块起一个简单的web服务器
vivi@ubuntu-ssd:~$ python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
# 然后再到目标服务器，使用wget把文件下载下来。。。如果有文件夹的话就tar打包一下
vivi@ubuntu-ssd:~$ wget http://source-server-ip:8000/file.txt
{% endhighlight %}

2. 需要ssh的情况下，例如从本地传一个文件到服务器上（或者从服务器上下载一个文件）
{% highlight bash %}
scp -i /path/to/private.key -P 22 /path/to/local/file.txt username@remote-server:/path/to/dest/folder/
-r 参数传输目录
tar -czvf - /path/to/file.txt | ssh username@server "tar -xzvf - -C /path/to/file.txt"
{% endhighlight %}

3. 使用filezilla，上传/下载/折腾/不解释

nc(netcat)

> The nc (or netcat) utility is used for just about anything under the sun involving TCP or UDP. It can open TCP connections, send UDP packets,  
> listen on arbitrary TCP and UDP ports, do port scanning, and deal with both IPv4 and IPv6. Unlike telnet(1), nc scripts nicely, and separates  
> error messages onto standard error instead of sending them to standard output, as telnet(1) does with some. 

常用参数说明

* -l 监听端口，而不是连到远程端口
* -q interval 接收到EOF之后等待interval秒后退出

传输单个文件

* 目标服务器： nc -l 12345 -q 1 > /path/to/file.txt
* 发送服务器： nc remote-server 12345 < /path/to/file.txt

传输大文件或者文件夹，可以配合tar打包压缩传输

* 目标服务器：nc -l 12345 -q 1 \| tar -xzvf -
* 发送服务器：tar -czvf - /path/to/source/folder \| nc remote-server 12345

另外说下如果tar要解压到非当前目录，那么参数是 `-C /path/to/dest/folder/` 而不是想当然的 `tar -xzvpf /path` 这个形式其实是提取压缩包指定的一个文件。
