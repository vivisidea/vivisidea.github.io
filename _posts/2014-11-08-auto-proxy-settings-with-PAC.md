---
title: PAC自动代理配置文件
author: vivi
layout: post
---
PAC = Proxy Automatic Configuration 用来自动配置代理的，本来一直在用chrome的switchy插件，最近看了眼PAC，感觉比插件还简单一点，对我个人来说，平时需要翻的网站相对比较固定，使用的代理软件也很固定，相当于是“一次配置，到处运行”了

PAC文件内置的函数（<a href="http://findproxyforurl.com/pac-functions/" target="_blank">http://findproxyforurl.com/pac-functions/</a> 抄一份自己用好查）

{% highlight javascript %}
// If the hostname matches or contains google.com (e.g. maps.google.com, www.google.com),
// send direct to the Internet.
if (dnsDomainIs(host, ".google.com"))
    return "DIRECT";

// Any requests with a hostname ending with the extension .local
// will be sent direct to the Internet.
if (shExpMatch(url, "*.local"))
    return "DIRECT";

// If IP of requested website website falls within IP range, send direct to the Internet.
if (isInNet(dnsResolve(host), "172.16.0.0", "255.240.0.0"))
    return "DIRECT";

// If the machine requesting a website falls within IP range,
// send traffic via proxy 10.10.5.1 running on port 8080.
if (isInNet(myIpAddress(), "10.10.1.0", "255.255.255.0"))
    return "PROXY 10.10.5.1:8080";

// If IP of the requested host falls within any of the ranges specified, send direct.
if (isInNet(dnsResolve(host), "10.0.0.0", "255.0.0.0") ||
    isInNet(dnsResolve(host), "172.16.0.0",  "255.240.0.0") ||
    isInNet(dnsResolve(host), "192.168.0.0", "255.255.0.0") ||
    isInNet(dnsResolve(host), "127.0.0.0", "255.255.255.0"))
    return "DIRECT";

// If user requests plain hostnames, e.g. http://intranet/, 
// http://webserver-name01/, send direct.
if (isPlainHostName(host))
    return "DIRECT";

// If the Host requested is "www" or "www.google.com", send direct.
if (localHostOrDomainIs(host, "www.google.com"))
    return "DIRECT";

// If hostname contains any dots, send via proxy1.example.com, otherwise send direct.
if (dnsDomainLevels(host) &gt; 0)
    return "PROXY proxy1.example.com:8080";
    else return "DIRECT";

// If during the period of Monday to Friday, proxy1.example.com will be returned, otherwise
// users will go direct for any day outside this period.
if (weekdayRange("MON", "FRI")) return "PROXY proxy1.example.com:8080";
    else return "DIRECT";

// If during the period of January to March, proxy1.example.com will be returned, otherwise
// users will go direct for any month outside this period.
if (dateRange("JAN", "MAR")) return "PROXY proxy1.example.com:8080";
    else return "DIRECT";

// If during the period 8am to 6pm, proxy1.example.com will be returned, otherwise
// users will go direct for any time outside this period.
if (timeRange(8, 18)) return "PROXY proxy1.example.com:8080";
    else return "DIRECT";

// Outputs the resolved IP address of the host in the browser
// to end-user or error console. 
resolved_host = dnsResolve(host);
alert(resolved_host);
{% endhighlight %}

从内置的函数来看，PAC挺强大的，比如可以自动根据解析的地址选择是否使用代理，还可以根据不同的时间段来选择不同的代理

## PAC文件调试

用浏览器调试

- IE浏览器：直接可以看到alert输出（我测试了下有些诡异，具体表现是刷几次之后不弹出alert了，IE用的不多就不折腾了）
- chrome浏览器：chrome://net-internals/#events
- firefox浏览器：使用Error Console插件，在“消息”里可以看到pac的调试输出


用工具调试

- https://code.google.com/p/pactester/ 用于测试PAC文件的perl脚本，似乎已经不再维护了
- https://code.google.com/p/pacparser/ PAC解析工具（其实可以基于这个工具做一个在线测试PAC的页面）
- windows也有工具可以用，下载地址https://code.google.com/p/pacparser/downloads/list
    - pactester.exe -p E:\developer\www\gfw-proxy.pac -u http://facebook.com
    - SOCKS5 127.0.0.1:1080; SOCKS 127.0.0.1:1080; DIRECT;


## 参考资料

<a href="http://findproxyforurl.com/" target="_blank">http://findproxyforurl.com/</a> 专门介绍PAC文件的网站  
<a href="http://findproxyforurl.com/pac-functions/" target="_blank">http://findproxyforurl.com/pac-functions/</a> PAC文件支持的函数  
<a href="https://github.com/clowwindy/gfwlist2pac/blob/master/test/proxy.pac" target="_blank">https://github.com/clowwindy/gfwlist2pac/blob/master/test/proxy.pac</a> 

国内fq可以参考这份列表，从gfwlist翻译过来的（ps。项目owner是shadowsocks的作者。。）
