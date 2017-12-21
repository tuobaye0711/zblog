---
title: web超时配置总结
date: 2017-12-19 19:54:22
tags: [JavaScript,Nginx,Node.js]
picture: timeout.jpg
description: 前段时间做项目时，由于某个请求在后台要进行长时间的脚本处理，频频超出系统默认的超时时长，于是我就对超时时长进行了检查。由于超时的时候请求报错的提示信息并不明确，为了修改这玩意儿也查了一堆资料，忙活了半天，终于把超时问题解决了，分享在这里供大家参考。
---

## 引子

***

前段时间做项目时，由于某个请求在后台要进行长时间的脚本处理，频频超出系统默认的超时时长，于是我就对超时时长进行了检查。由于超时的时候请求报错的提示信息并不明确，为了修改这玩意儿也查了一堆资料，忙活了半天，终于把超时问题解决了，在这里做个记录。

我的项目用的是web前台（这个无所谓用的什么框架，反正归根结底是js）+nginx代理+Node.js后台搭建的，我发现每一节都设了一道timeout超时的关卡，也就是说有**请求超时**，**代理超时**，**响应超时**三种情况。下面针对每种超时，我将进行分别讲解。

![timeout!](timeout.jpg)

## 请求超时

***

### XMLHttpRequest的超时设置

讲请求超时，就要从XMLHttpRequest来谈起。

XMLHttpRequest是一个API，它为客户端提供了在客户端和服务器之间传输数据的功能。它提供了一个通过URL来获取数据的简单方式，并且不会使整个页面刷新。这使得网页只更新一部分页面而不会打扰到用户。XMLHttpRequest在AJAX中被大量使用。

XMLHttpRequest一开始只是微软浏览器提供的一个接口，后来各大浏览器纷纷效仿也提供了这个接口，再后来W3C对它进行了标准化，提出了XMLHttpRequest标准。XMLHttpRequest标准又分为Level 1和Level 2。
XMLHttpRequest Level 1主要存在以下缺点：

- 受同源策略的限制，不能发送跨域请求；
- 不能发送二进制文件（如图片、视频、音频等），只能发送纯文本数据；
- 在发送和获取数据的过程中，无法实时获取进度信息，只能判断是否完成；

那么Level 2对Level 1 进行了改进，XMLHttpRequest Level 2中新增了以下功能：

- 可以发送跨域请求，在服务端允许的情况下；
- 支持发送和接收二进制数据；
- 新增formData对象，支持发送表单数据；
- 发送和获取数据时，可以获取进度信息；
- 可以设置请求的超时时间；

在Level 2版本的XMLHttpRequest对象，增加了timeout属性，可以设置HTTP请求的时限。

{% codeblock lang:javascript %}
    xhr.timeout = 60*1000;
    xhr.ontimeout = function(event){
　　　　alert('timeout！');
　　}
{% endcodeblock %}

在上述代码中，请求超时时间被设为1分钟（xhr.timeout的单位是毫秒），如果超过1分钟的时限，则会自动停止http请求；同时，触发ontimeout事件，弹出“timeout！”的提示框。

同时值得注意的一点是，超时时间的计算，是从调用xhr.send()开始，至xhr.loadend触发为止的这段时间。即时xhr.timeout的设置是在xhr.send()之后，timeout的计时起点仍为调用xhr.send()的时刻。

### 其他超时设置

其实会了XMLHttpRequest的超时设置，其他前端的框架啊、工具啊的超时设置都不再是问题，这就有点万法归宗的意思。因为我们常用的jQuery.ajax()方法实际上就是对浏览器提供的XMLHttpRequest对象的封装。而又有很多其他框架或者工具的请求模块是对jQuery.ajax()的封装。说到底，都是依赖的XMLHttpRequest对象。因此掌握了XMLHttpRequest，其他都很好学会。

依jQuery.ajax()的超时设置为例：

{% codeblock lang:javascript %}
    $.ajax({
        url: "test.html",
        error: function(){
            // will fire when timeout is reached
        },
        success: function(){
            // do something
        },
        timeout: 60*1000 // sets timeout to 1 minute
    });
{% endcodeblock %}

实在是简单，对不对？

## 代理超时

***

nginx的超时设置主要分3种：

- proxy_connect_timeout
- proxy_read_timeout
- proxy_send_timeout

### proxy_connect_timeout

> 语法: proxy_connect_timeout timeout_in_seconds
> 上下文: http, server, location
> 默认值: 60s

proxy_connect_timeout是和后端建立连接的超时时间。需要记住的是，这个时间不能超过75秒。

这个不是等待后端返回页面的时间，那是由proxy_read_timeout声明的。如果你的upstream服务器起来了，但是挂起了（例如，没有足够的线程处理请求，所以把你的请求放到请求池里稍后处理），那么这个声明是没有用的，由于与upstream服务器的连接已经建立了。


### proxy_read_timeout

> 语法: proxy_read_timeout the_time
> 上下文: http, server, location
> 默认值: 60s

proxy_read_timeout是从后端读取数据的超时时间，两次读取操作的时间间隔如果大于这个值，和后端的连接会被关闭。如果一个请求时间时间非常大，要把这个值设大点。

### proxy_send_timeout

> 语法: proxy_send_timeout time
> 上下文: http, server, location
> 默认值: 60s

proxy_send_timeout是向后端写数据的超时时间，两次写操作的时间间隔大于这个值，也就是过了这么长时间后端还是没有收到数据，连接会被关闭。

## 响应超时

***



## 小结