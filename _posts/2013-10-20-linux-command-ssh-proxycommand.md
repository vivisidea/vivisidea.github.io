---
title: 通过跳板机ssh登录服务器
author: vivi
layout: post
permalink: /posts/346.html
categories:
  - 技术记录
tags:
  - linux
  - ssh
---
通过跳板机ssh登录服务器

背景
厂里面登录服务器使用到VPN，但是由于使用人数众多，上班高峰期（例如上午）VPN连接非常卡，经常敲一个命令需要停顿10-15秒，有时还会因为VPN连接太卡而导致一些错误的命令执行，咨询了SA之后，得到的解决方案是：以部分可以在办公区直接登录的服务器为跳板，登录目标服务器，绕开VPN连接。
废话少说，直接上配置

{% highlight bash %}
.ssh/config配置

Host gateway
    Hostname gateway-server-ip
    User user
    Port 22
    
Host *.xxx
    User user
    Port 22
    ProxyCommand ssh gateway exec nc -w 10 %h %p 2>/dev/null
{% endhighlight %}

所有ssh *.xxx 的请求都会通过gateway发送，这里的

```
-w 10
```
用来防止退出登录后跳板机上的nc进程一直存在，

```
2>/dev/null
```
用来防止出现`Killed By Signal 1`现象。

公钥认证
由于纯密码认证比较脆弱，容易遭到暴力破解，所以ssh登录服务器一般使用publickey（公钥认证）的形式，生成private/public密码对，并且将publickey加到目标服务器的$HOME/.ssh/authorized\_keys文件中（需权限的管理员授权），连接时指定当前连接需要的private-key（默认是.ssh/id\_rsa），可通过-i参数指定(IdentityFile)
ssh -i /path/to/private-key user@host -p 22

该过程简要步骤如下

1. ssh-client根据指定端口与远程服务器建立tcp连接
2. ssh-client提示输入private-key以解密私钥
3. ssh-client发送私钥签名的包含username和公钥信息的message
4. 远程服务器ssh-daemon检查消息中的公钥是否在$HOME/.ssh/authorized_keys文件中存在，如果ok，那么通过认证

如果在每次连接的时候都指定private-key，那么每次连接的时候都需要输入一次private-key的密码以解锁private-key，比较麻烦，因此有了ssh-agent

ssh-agent

ubuntu系统默认会启动ssh-agent服务，如果没有启动，那么可以通过ssh-agent $SHELL 启动，启动之后通过ssh-add private.key将私钥添加到agent中，ssh-client发起登录请求时会先通过$SSH\_AUTH\_SOCK尝试获取私钥签名的message，一个agent可以添加多个private-key，可以理解成私钥管理器

```
vivi@ssd:~$ echo $SSH_AUTH_SOCK
/tmp/ssh-NAgrC11785/agent.11785
```

ForwardAgent yes
这个配置应该是用来配置是否将本地agent配置信息转发到远程服务器的，但是测试发现这个配置似乎没有用，即使设置成no，也能通过跳板机登录服务器，而且登录到目标服务器之后$SSH\_AUTH\_SOCK也是处于未设置状态的（目前还没弄清楚这个是什么情况）

注:
其实private-key里面是包含publickey信息的，可以通过ssh-keygen -y(见man ssh-keygen参数说明)读出publickey

```
ssh-keygen -y -f /path/to/private-key
```

搜资料的过程中找到一篇关于ssh-agent安全性的说明文章

<a href="http://blog.hellosa.org/2010/03/07/ssh-agent-secure.html" target="_blank">ssh-agent的安全隐患 似乎碰上了我厂前员工</a>

好乱，其实没搞明白底层的原理，暂且先记录一下吧
