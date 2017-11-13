---
title: Nginx 配置指南
date: 2017-11-11 10:50:50
tags: Nginx
---

## 什么是Nginx？

***

没有什么文档比直接从[Nginx官网]()来的更准确清晰了。

> Nginx [engine x] is an HTTP and reverse proxy server, a mail proxy server, and a generic TCP/UDP proxy server, originally written by Igor Sysoev. For a long time, it has been running on many heavily loaded Russian sites including Yandex, Mail.Ru, VK, and Rambler. According to Netcraft, Nginx served or proxied 29.43% busiest sites in October 2017. Here are some of the success stories: Dropbox, Netflix, Wordpress.com, FastMail.FM.

Nginx [engine x]是最初由Igor Sysoev编写的HTTP和反向代理服务器，邮件代理服务器和通用TCP/UDP代理服务器。很长时间以来，它一直在许多重负荷的俄罗斯网站上运行，包括Yandex，Mail.Ru，VK和Rambler。根据Netcraft，2017年10月，Nginx服务或代理了 29.43％最繁忙的站点。下面是一些成功案例： Dropbox， Netflix， Wordpress.com， FastMail.FM。

Nginx的主要特性：
- Basic HTTP server features(基本的HTTP服务器功能)
- Other HTTP server features(其他HTTP服务器功能)
- Mail proxy server features(邮件代理服务器功能)
- TCP/UDP proxy server features(TCP/UDP代理服务器功能)
- Architecture and scalability(架构和可扩展性)

## 为什么要使用Nginx，跟以前用的tomcat有什么区别？

***

虽然大家都叫web server，但是Nginx和tomcat有本质的不同。

Nginx常用来做静态内容服务器和代理服务器，用来放置静态资源或者转发请求给后面的应用服务；而tomcat常用来做应用容器，让java app在其中运行。

因此，严格来说，Nginx应该叫<span style="color:red">HTTP Server</span>，而tomcat则是一个<span style="color:red">Application Server</span>

一个 HTTP Server关心的是HTTP协议层面的传输和访问控制，所以在Nginx上你可以看到代理、负载均衡等功能。客户端通过HTTP Server 访问服务器上存储的资源（HTML 文件、图片文件等等）。通过CGI技术，也可以将处理过的内容通过HTTP Server分发，但是一个HTTP Server始终只是把服务器上的文件如实的通过HTTP协议传输给客户端。

而应用服务器，则是一个应用执行的容器。它首先需要支持开发语言的 Runtime（对于 Tomcat 来说，就是 Java），保证应用能够在应用服务器上正常运行。其次，需要支持应用相关的规范，例如类库、安全方面的特性。对于 Tomcat 来说，就是需要提供 JSP/Sevlet 运行需要的标准类库、Interface 等。为了方便，应用服务器往往也会集成 HTTP Server 的功能，但是不如专业的 HTTP Server 那么强大，所以应用服务器往往是运行在 HTTP Server 的背后，执行应用，将动态的内容转化为静态的内容之后，通过 HTTP Server 分发到客户端。

于我个人来说，选择Nginx主要是因为其占用资源低的优点，比起tomcat，选择Nginx能给我释放大量的系统内存，供我其他IDE和chrome使用。

## nginx.conf配置文件指南

***

Nginx的安装，启动，关闭等步骤的教程在这里我们就不做赘述了，我们来主要谈一谈nginx.conf文件应该如何配置，这是nginx的灵魂所在。

### Nginx配置文件结构

nginx包含由配置文件中指定的指令控制的模块。指令分为**简单指令**和**块指令**。

一个简单的指令由**名称**和**参数**组成，以空格分隔，并以分号（;）结束:

{% codeblock lang:javascript %}
    root /data/www;
{% endcodeblock %}

一个block指令和一个简单的指令有相同的结构，但是**不是以分号结尾，而是用一系列由大括号（{和}）包围的附加指令来结束**。如果一个block指令在大括号内可以有其他的指令，它就被称为一个context（上下文，例如：events，http，server和location）:

{% codeblock lang:javascript %}
    http {
        server {
            #location / {
                 proxy_pass   http://127.0.0.1:8090/;
                 proxy_redirect  http://127.0.0.1:8090/ /;
                 proxy_connect_timeout 600s;
                 proxy_read_timeout 600s;
                 proxy_send_timeout 600s;
            }
        }
    }
{% endcodeblock %}

置于任何context之外的配置文件中的指令被认为是在main context中。在*events*和*http*指令驻留在main content中，*server*是在*http*中，*location*则在*server*中。

#后面的部分是注释：

{% codeblock lang:javascript %}
    # 这是一段注释
{% endcodeblock %}

### Nginx作为静态服务器使用

作为一个Web服务器，其最主要的任务是作为静态服务器使用。

你需要将静态网页和文件放到一个目录（例如/data/www），将图片等文件放到另一个目录（例如/data/images），然后在nginx.conf中进行配置。这需要在*http*模块下的*server*模块内新建两个*location*模块：

{% codeblock lang:javascript %}
    http {
        server {
            location / {
                root /data/www;
            }
            location /images/ {
                root /data;
            }
        }
    }
{% endcodeblock %}

看起来很好理解吧~也可以直接把文件放到一块，直接配置绝对路径：

{% codeblock lang:javascript %}
    location / {
        root   F:\webapp\portal;
    }
{% endcodeblock %}

