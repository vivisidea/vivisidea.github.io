---
title: Java 日志框架
author: vivi
layout: post
categories:
  - 技术记录
tags:
  - Java
---

Java 日志框架整理
Java Logging

日志框架对于 Java 程序员来说肯定不会陌生，而且第一印象估计都是 log4j，由于其配置&用法都相对简单，所以很容易忽略掉背后的日志框架，大部分的日志打印代码类似这样
{% highlight java %}
import org.apache.log4j.Logger;

public class LoggingTest {
    private static final Logger logger = Logger.getLogger(LoggingTest.class);

    @Test
    public void test() {
        logger.info("this is logging message");
    }
}

{% endhighlight %}
这么写有个不好的地方是，日志框架从此就绑定了 log4j，如果要使用别的日志框架，必须大面积的修改代码。更优雅的方案是结合 commons-logging 和 log4j，面向 commons-logging API 编程

{% highlight java %}
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

public class SkeletonTest {
    private static Log log = LogFactory.getLog(SkeletonTest.class);
    @Test
    public void test(){
        log.info("this is log from commons-loggin API");
    }
}
{% endhighlight %}
我们测试发现这段代码的效果和上面是一样的，这是因为 commons-logging 框架设计了一种动态查找日志框架实现的机制，我们想要使用 log4j 的实现，只需要把 log4j 的 jar 包放在 classpath 下即可

首先，动态查找 `LogFactory` 实现类

* 查找系统属性中的 `org.apache.commons.logging.LogFactory` 定义
* 扫描 `META-INF/services/org.apache.commons.logging.LogFactory` 文件，加载里面的配置（JDK1.3中的服务发现机制）
* 查找 `commons-logging.properties` 文件中的 `org.apache.commons.logging.LogFactory` 定义
* 使用默认的 `org.apache.commons.logging.impl.LogFactoryImpl` 实现类

其次，动态查找 `Log` 实现类

* 查找配置属性中的 `org.apache.commons.logging.Log` 定义
* 查找系统属性中的 `org.apache.commons.logging.Log` 定义
* 使用标准实现类的尝试加载顺序

{% highlight java %}
/**
 * The names of classes that will be tried (in order) as logging
 * adapters. Each class is expected to implement the Log interface,
 * and to throw NoClassDefFound or ExceptionInInitializerError when
 * loaded if the underlying logging library is not available. Any 
 * other error indicates that the underlying logging library is available
 * but broken/unusable for some reason.
 */
private static final String[] classesToDiscover = {
        LOGGING_IMPL_LOG4J_LOGGER,
        "org.apache.commons.logging.impl.Jdk14Logger",
        "org.apache.commons.logging.impl.Jdk13LumberjackLogger",
        "org.apache.commons.logging.impl.SimpleLog"
};
{% endhighlight %}

    Apache Commons Logging
    log4j
    slf4j
    logback
    http://en.wikipedia.org/wiki/Java_logging_framework
