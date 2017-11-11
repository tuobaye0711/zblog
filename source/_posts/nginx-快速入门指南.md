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

而应用服务器，则是一个应用执行的容器。它首先需要支持开发语言的Runtime（对于Tomcat来说，就是 ava），保证应用能够在应用服务器上正常运行。其次，需要支持应用相关的规范，例如类库、安全方面的特性。对于 Tomcat 来说，就是需要提供 JSP/Sevlet 运行需要的标准类库、Interface 等。为了方便，应用服务器往往也会集成HTTP Server的功能，但是不如专业的 TTP Server 那么强大，所以应用服务器往往是运行在HTTP Server的背后，执行应用，将动态的内容转化为静态的内容之后，通过HTTP Server 分发到客户端。
