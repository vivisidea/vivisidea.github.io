---
title: 删除known_hosts文件里面的记录
author: vivi
layout: post
permalink: /posts/246.html
categories:
  - 技术记录
---
今天连<a href="http://www.usssh.com" target="_blank">代理服务器</a>的时候，发现服务器的RSA key似乎修改了，解决的方法显然是删除原有的known_hosts文件里的记录然后重新添加，这里有好几个方法

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@       WARNING: POSSIBLE DNS SPOOFING DETECTED!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
The RSA host key for 1.usssh.com has changed,
and the key for the corresponding IP address 50.31.254.246
is unknown. This could either mean that
DNS SPOOFING is happening or the IP address for the host
and its host key have changed at the same time.
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that the RSA host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
80:bf:52:8a:f2:78:b2:9d:3a:5c:01:77:a5:66:a9:df.
Please contact your system administrator.
Add correct host key in /home/vivi/.ssh/known_hosts to get rid of this message.
Offending key in /home/vivi/.ssh/known_hosts:71
RSA host key for 1.usssh.com has changed and you have requested strict checking.
Host key verification failed.
```

1. 网上搜到的方法一般都是说直接打开`~/.ssh/known\_hosts`文件，然后找到相应的`host`，删掉那一行，但是实际发现`known\_hosts`里保存的host不是明文的(ubuntu10.04)，因此这个方法不行  
2. 注意到错误提示文字里面已经说了冲突的key在哪一行，因此用`vim`打开，:71直接定位到那一行`dd`删除，`:wq`即可  
3. 使用`ssh-keygen -R host_name`命令删除（推荐），还带自动备份的

## update
后来还是觉得太麻烦，服务器老是换来换去的，每次要重新设置一次也很麻烦，而且连接个代理没必要检查，反正你都会输入yes，不是么？

{% highlight bash %}
while [ 1 ] ; do
  SERVER=$(random_server);
  logger ".............connecting to server: $SERVER.............."
  sshpass -f "$ROOT/cdkey.in.pass" ssh -2 -gCNvD 7071 $USERNAME@$SERVER -p 22 \
        -o PasswordAuthentication=yes \
        -o ChallengeResponseAuthentication=no \
        -o GSSAPIAuthentication=no \
        -o HostbasedAuthentication=no \
        -o PubkeyAuthentication=no \
        -o RhostsRSAAuthentication=no \
        -o UserKnownHostsFile=/dev/null \
        -o StrictHostKeyChecking=no \
        -o RSAAuthentication=no &gt;&gt; $LOG 2&gt;&1
  notify-send -t 1000 "ssh maybe broken, try restarting in $WAIT seconds"
  sleep $WAIT
done
{% endhighlight %}

参数说明

1. `UserKnownHostsFile=/dev/null` 不使用known_hosts文件
2. `StrictHostKeyChecking=no` 不做严格的服务器key检查
3. 此设置只对当次`ssh`连接有效，如果需要对所有的ssh连接生效，可以将配置写到 `~/.ssh/config` 文件里面（不存在创建即可）

