---
title: ubuntu下手动连接pptp
author: vivi
layout: post
permalink: /posts/362.html
categories:
  - 技术记录
tags:
  - linux
  - pptp
  - ubuntu
  - vpn
---
ubuntu下手动连接pptp-vpn

最近 Google 基本被墙了，Dropbox 彻底被墙了，无奈入手了一个 vpn，vpn 的原理是创建一个虚拟网卡，默认情况下会将默认网关设置为 vpn 的网关，也就是说所有流量都会走 vpn，这对于我们访问国内的网站是不方便的，于是有了 chnroute 项目， 看了下项目的 wiki，原理其实不复杂，简单来说就是在 vpn 启动之前，通过配置路由表的方式将国内的 ip 范围全部设置为直连，系统选择路由的时候会优先选择匹配度高的路由规则，如果没有才会选择默认路由规则。

本文记录的是ubuntu 12.04下手动连接pptp以及配置chnroute

## clone chnroute项目

clone chnroute项目（也可以直接找生成好的路由表，通常几个月才有点变化），生成国内路由信息，根据提示将生成的文件cp到对应的目录。

## 安装pptp-linux

```
sudo apt-get install pptp-linux
sudo pptpsetup --create ppp0 --client --server $server --username $username --password $password --encrypt --start
sudo poff ppp0 断开vpn连接
sudo pon ppp0 启用vpn连接
```

## 配置路由规则

pptp手动启动之后并不会添加默认路由，需要手动添加一下（可以自行加到ip-up.d/ip-down.d文件里去实现自动化）

```
sudo ip route del default
sudo ip route add default dev ppp0

vivi@ubuntu-ssd:~$ route -n | more
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         0.0.0.0         0.0.0.0         U     0      0        0 ppp0
1.0.1.0         192.168.1.1     255.255.255.0   UG    0      0        0 eth1
1.0.2.0         192.168.1.1     255.255.254.0   UG    0      0        0 eth1
1.0.8.0         192.168.1.1     255.255.248.0   UG    0      0        0 eth1
1.0.32.0        192.168.1.1     255.255.224.0   UG    0      0        0 eth1
1.1.0.0         192.168.1.1     255.255.255.0   UG    0      0        0 eth1
1.1.2.0         192.168.1.1     255.255.254.0   UG    0      0        0 eth1
...
```

## 修改dns /etc/resolv.conf

dns也要修改成8.8.8.8或者其他opendns配置，否则dropbox还是被dns污染的

## 验证效果

验证，浏览器访问

- [http://ip138.com/](http://ip138.com/) 看到是中国xx电信
- [http://showip.net/](http://showip.net/) 看到是United States xxx

就成了

之前看到过一个吐嘈，说国外人民看到我朝普通网民都会各种操作route/iptables应该会很惊讶吧。。尼玛都是被逼的好吧

[chnroute项目@github](https://github.com/fivesheep/chnroutes)
[国内ip数据来自APNIC](http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest)

