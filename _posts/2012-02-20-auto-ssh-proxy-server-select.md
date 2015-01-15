---
title: ssh代理自动登录/断线自动重连脚本
author: vivi
layout: post
permalink: /posts/182.html
categories:
  - 技术记录
tags:
  - shell
  - ssh
---
买了个ssh代理，试了下性价比还行，就是偶尔会出现登录会失败、Broken Pipe等状况导致`ssh`退出，于是自己动手写了个脚本

系统: ubuntu 10.10  
安装软件
{% highlight bash %}
sudo apt-get install sshpass
sudo apt-get install libnotify-bin
{% endhighlight %}

其中`sshpass`是用来自动发送`ssh`代理密码的，当然也可以用`expect`，但是个人认为`expect`太重量级，文档又长又难懂，还是`sshpass`方便；`libnotify-bin`这个工具是用来发送屏幕右上角浮动通知的，这样就可以及时掌握当前ssh代理的状况了

原本只是一行sshpass自动登录代码，改进后增加了以下功能

  * 自动从4台服务器中选择一台（因为商家给了4个地址供选择）
  * verbose log到文件(输出控。。。)
  * notify-send桌面通知
  * 断线后自动重连
  * 增加pill文件检测，否则ssh服务器真出问题了只能kill，现在只要手动touch pill就会自动退出循环

{% highlight bash %}
#!/bin/bash
# file ssh.proxy
# created by make-template :)
# vivi@2012-02-20_21:30:05

ROOT=$HOME/scripts/ssh.proxy
WAIT=5
PILL=$ROOT/pill
LOG=$ROOT/../ssh.log
SERVERS=(s{1,2}.alissh.com)
USER=jmepvcts
PORT=56
COUNT=${#SERVERS[@]}

function logger(){
  local MSG=$1
  local TIMESTAMP=$(date '+%F %H:%M:%S')
  echo "$TIMESTAMP: $MSG"  >> $LOG
}

function random_server(){
  local INDEX=$(( $RANDOM % $COUNT ))
  echo -n ${SERVERS[$INDEX]}
}

# 开机连无线需要一段时间
sleep 20

notify-send -t 5000 "starting ssh.proxy :)"
while [ 1 ] ; do
  if [ -f $PILL ]; then
      rm $PILL
      echo "pill file detected, exiting main loop"
      break
  fi
  SERVER=$(random_server);
  logger ".............connecting to server: $SERVER:$PORT.............."
  sshpass -f "$ROOT/ssh.proxy.pass" ssh -2 -gCNvD 7071 $USER@$SERVER -p $PORT \
        -o PasswordAuthentication=yes \
        -o ChallengeResponseAuthentication=no \
        -o GSSAPIAuthentication=no \
        -o HostbasedAuthentication=no \
        -o PubkeyAuthentication=no \
        -o RhostsRSAAuthentication=no \
        -o UserKnownHostsFile=/dev/null \
        -o StrictHostKeyChecking=no \
        -o RSAAuthentication=no >> $LOG 2>&1
  notify-send -t 1000 "ssh maybe broken, try restarting in $WAIT seconds"
  sleep $WAIT
done
{% endhighlight %}

然后写了个launch脚本，放到.profile文件里面就可以实现开机自动连ssh代理了～
{% highlight bash %}
#!/bin/bash
# file ssh-launcher
# created by make-template :)
# vivi@2012-02-14_22:27:57

ROOT=$HOME/scripts/ssh.proxy
if pidof -x $ROOT/ssh.proxy > /dev/null 2>&1; then
    echo "already running."
    exit 1
fi

$ROOT/ssh.proxy > /dev/null 2>&1 &
echo $! > $ROOT/ssh.proxy.pid
{% endhighlight %}

[https://gist.github.com/vivisidea/5079782](https://gist.github.com/vivisidea/5079782)
