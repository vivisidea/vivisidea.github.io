---
title: 一台服务器运行多个tomcat实例
author: vivi
layout: post
permalink: /posts/340.html
categories:
  - 技术记录
tags:
  - linux
  - tomcat
---
之前整理过的，贴出来备忘一下

一台服务器运行多个tomcat实例
how to run multiple tomcat instance on a single server.

理解几个概念

- `$CATALINA_HOME` tomcat的安装目录，lib目录是tomcat用到的库，对tomcat所有instance可见，bin是tomcat启动停止脚本，用这个路径来查找bin和lib目录
- `$JAVA_HOME` java的安装目录，一般是/usr/lib/jvm/java-xx-xx，通常会使用符号链接
- `$CATALINA_TMPDIR` tomcat运行的临时目录
- `$CLASSPATH`
- `$CATALINA_BASE` 是用来配置多个instances的利器，如果这个值定义了，tomcat会使用这个值来计算以下目录的位置，而不使用CATALINA_HOME的值
- `conf` contains configuration files and related DTDs. The most important file located here is server.xml.
- `logs` holds log and output files.
- `webapps` is where you put the applications.
- `work` – Tomcat translates and converts any JavaServer Pages (JSP) into servlets and stores them here.
- `temp` is used for temporary files.

默认$CATALINA\_BASE是没有配置的，所以以上这些目录都会通过$CATALINA\_HOME计算，也就是tomcat安装目录。

- /usr/share/$NAME tomcat安装目录，启动脚本bin/，jar包lib/，也就是CATALINA_HOME需要设置的值
- /var/lib/$NAME
- /etc/$NAME
- /etc/init.d/$NAME tomcat的启动/停止脚本，可以作为一个模板实现multiple instance的脚本

打开文件数量限制修改

```
修改 /etc/security/limits.conf（貌似要重启还是重新login）
vivi            soft    nofile          200000
vivi            hard    nofile          200000
ulimit -n "$MAX_FILEDESCRIPTORS" 注意这个值只能改成&lt;=limits.conf里面设置的值
```

tomcat端口说明
Here is a list of the ports to change:

- `Shutdown` port is used for shutting down Tomcat. When we call the shutdown.sh script, it sends a signal to shutdown port. If the Tomcat Java process receives the signal, it cleans up and exits.
- `HTTP` port is the actual port that exposes the application to an outside client via HTTP.
- `ajp` (Apache JServ Protocol) port may be used by a web server (such as the Apache httpd server) to communicate with Tomcat. This port is also used if you set up a load-balanced server.
- `Redirect` port – If the Connector supports non-SSL requests and a request is received for which a security constraint requires SSL, Catalina will automatically redirect the request to this port.

Tomcat至少需要两个端口，Shutdown和HTTP，shutdown端口总是监听在本机localhost，所以是无法从远程发送shutdown命令的。

如果使用rotatelogs来做一天一个log文件

{% highlight bash %}
# Run the catalina.sh script as a daemon
set +e
touch "$CATALINA_PID"
rm -f "$CATALINA_BASE/logs/catalina.out"
mkfifo -m700 "$CATALINA_BASE/logs/catalina.out"
# Look for rotatelogs/rotatelogs2
if [ -x /usr/sbin/rotatelogs ]; then
          ROTATELOGS=/usr/sbin/rotatelogs
else
          ROTATELOGS=/usr/sbin/rotatelogs2
fi

# -p preserves the environment (for $JAVA_HOME etc.)
# -s is required because tomcat5.5's login shell is /bin/false
$ROTATELOGS -f $CATALINA_BASE/logs/catalina_%F.log 86400 \
                        &lt; "$CATALINA_BASE/logs/catalina.out" &
chown $TOMCAT6_USER "$CATALINA_PID" "$CATALINA_BASE"/logs/catalina.out
start-stop-daemon --start -b -u "$TOMCAT6_USER" -g "$TOMCAT6_GROUP" \
        -c "$TOMCAT6_USER" -d "$CATALINA_TMPDIR" -p "$CATALINA_PID" \
        -x /bin/bash -- -c "$AUTHBIND_COMMAND $TOMCAT_SH"
status="$?"
# 需要注释掉/usr/share/tomcat.x/bin/catalina.sh里的一行，否则会碰到启动卡住catalina.out无法输出的问题
#touch "$CATALINA_OUT"
{% endhighlight %}

reference

- <http://www.openlogic.com/wazi/bid/188102/How-to-Run-Multiple-Instances-of-Tomcat-on-a-Single-Server>
- <http://www.javacodegeeks.com/2011/08/multiple-tomcat-instances-on-single.html>
- <http://stackoverflow.com/questions/10300527/tomcat-multiple-localhost-instances>
- <http://www.shaunabram.com/multiple-tomcat-instances/>

