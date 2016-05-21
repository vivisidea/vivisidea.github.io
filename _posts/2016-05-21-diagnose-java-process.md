---
title: Java线上问题排查
layout: post
tags:
  - java
---
## 场景
* 前端机 apache/nginx
* 后端 N 台 tomcat﻿
* 域名 http://your-site.com

用户反馈，your-site.com 访问特别慢，如何一步步排查定位问题？

### 大体思路
* 确认问题
* 定位问题
* 修复问题

### 确认问题/重现问题
有时候并不是应用有问题，可能是用户自己的网络或者机器有问题，所以首先第一步要做的是确认问题是否存在

* 确认用户反馈对应的页面/功能，参数等细节
* 自己打开 your-site.com ，还原用户的请求
* F12 查看网络面板，定位到慢的 URL

### 确认与前端机的网络连接状况
* `ping -i 0.2 -c 20 your-site.com` 发送 20 个 ping 包之后停止， 查看 packet loss / rtt
* `curl` 请求查看各个请求阶段的耗时细节
* `curl -w @curl-format.txt -s -o /dev/null http://your-site.com`

{%highlight bash%}
# 输出 curl 执行细节
            time_namelookup:  %{time_namelookup}\n
               time_connect:  %{time_connect}\n
            time_appconnect:  %{time_appconnect}\n
           time_pretransfer:  %{time_pretransfer}\n
              time_redirect:  %{time_redirect}\n
         time_starttransfer:  %{time_starttransfer}\n
                            ----------\n
                 time_total:  %{time_total}\n
{%endhighlight%}

### 确认 nginx 前端机与后端机的连接情况
* 查看前端机负载
* 查看 nginx 日志，确认请求后端返回情况
* 直连后端 tomcat 节点看问题是否存在

### 查看 tomcat 日志
* 排查异常
* 查看请求时间差
* 查看对应的代码

### 查看服务器状态
* `top` 主要查看 load / cpu / mem
* `free - m` 查看空闲内存， 如果进程莫名被 kill，可以查看 dmesg 日志
* `iostat -d 1` 关注是否有某块磁盘写入/读取异常
* `vmstat -n 1` 关注 si/so(swap in/ swap out)
* `netstat -ant | awk '{print $6}' | sort | uniq -c | sort -rn` 各 tcp 连接状态统计
* `netstat -ant | grep 'x.x.x.x:1024' | awk '{print $5}' | awk -F: '{print $1}' | sort | uniq -c | sort -rn` 统计某个服务端口的 ip 连接数量

### 查看 jvm 状态
* 高 CPU 占用
    * `jstack <pid> stack.txt` 保存案发现场
        * 也可以使用 `kill -3 <pid>` 的方式生成 thread dump
    * `top -H` 查看具体占用 cpu 高的线程
    * 记录线程 id，`printf "%x" id` 转成 16 进制，通过 `jstack` 定位到异常的线程堆栈
    * `jstack 30979 | grep -C 20 'printf "%x" 31000'`
* 高内存占用
    * `jmap -dump:live,format=b,file=dump.bin <pid>` 保存现场事后再用 MAT 分析
    * `jmap -heap <pid>` 查看 jvm 堆内存的使用情况
    * `jmap -histo:live <pid> | more` 查看存活状态的对象数量统计
    * `jstat -gcutil -h50 <pid> 1000` 具体字段意义可以查看 `man jstat`
    * `jstat -gccause -h50 <pid> 1000` 显示最后一次 gc 和当前一次 gc 的原因
    
### 数据库
* `show processlist` 查看当前正在执行的查询列表
* 查看慢查询日志
* `explain select ... ` 分析执行计划，针对执行计划优化语句

### 参考资料
* [http://docs.oracle.com/javase/7/docs/technotes/tools/share/jstat.html](http://docs.oracle.com/javase/7/docs/technotes/tools/share/jstat.html)







