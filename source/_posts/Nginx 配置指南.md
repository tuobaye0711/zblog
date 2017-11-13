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

{% codeblock lang:nginx %}
    root /data/www;
{% endcodeblock %}

一个block指令和一个简单的指令有相同的结构，但是**不是以分号结尾，而是用一系列由大括号（{和}）包围的附加指令来结束**。如果一个block指令在大括号内可以有其他的指令，它就被称为一个context（上下文，例如：events，http，server和location）:

{% codeblock lang:nginx %}
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

{% codeblock lang:nginx %}
    # 这是一段注释
{% endcodeblock %}

### Nginx作为静态服务器使用

作为一个Web服务器，其最主要的任务是作为静态服务器使用。

你需要将静态网页和文件放到一个目录（例如/data/www），将图片等文件放到另一个目录（例如/data/images），然后在nginx.conf中进行配置。这需要在*http*模块下的*server*模块内新建两个*location*模块：

{% codeblock lang:nginx %}
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

看起来很好理解吧~也可以直接把文件放到一块，直接location配置绝对路径：

{% codeblock lang:nginx %}
    location / {
        root   F:\webapp\portal;
    }
{% endcodeblock %}

Nginx在未配置监听端口的情况下默认监听80端口，因此，你可以通过在本地访问 http://localhost/ 或者 http://127.0.0.1/ 来访问你的网站。

怎么样？赶紧启动一下Nginx吧，你的静态网站已经可以在本地运行了！修改配置后重启Nginx的命令是：

{% codeblock lang:nginx %}
    nginx -s reload
{% endcodeblock %}

如果您的Nginx无法启动或者出现其他错误，您可以尝试在 */usr/local/nginx/logs* 或者 */var/log/nginx* 中的 *access.log* 以及 *error.log* 中查找原因

### 搭建简单的代理服务器

Nginx经常作为反向代理服务器来使用，这意味着Nginx服务器接收请求，将其传递给被代理服务器，从中检索响应并将其发送给客户端。

> 反向代理（Reverse Proxy）方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。通过在网络各处放置反向代理节点服务器所构成的在现有的互联网基础之上的一层智能虚拟网络，CDN系统能够实时地根据网络流量和各节点的连接、负载状况以及到用户的距离和响应时间等综合信息将用户的请求重新导向离用户最近的服务节点上。

下面，我们将配置一个基本的代理服务器，它会处理本地图片文件的请求并返回其他的请求给被代理的服务器。在这个例子中，两个服务器将在一个Nginx实例上定义。

首先，在nginx的配置文件中增加一个server块来定义代理服务器，其中包含以下内容：

{% codeblock lang:nginx %}
    server {
        listen 8080;
        root /data/up1;
        location / {
        }
    }
{% endcodeblock %}

这将是一个简单的服务器，它监听8080端口。（如果不定义listen值的话，默认监听80端口）并将所有请求映射到本地文件系统上的目录/data/up1。创建这个目录并把index.html文件放进去。请注意，该root指令放置在server context中。当响应请求的 location 区块中，没有自己的 root 指令，上述的 root 指令才会被使用。

接下来，使用上一节中的服务器配置并对其进行修改，使其成为代理服务器配置。在第一个location块中，设置proxy_pass 指令，并在参数中配置指定的代理服务器的协议、名称和端口号（在我们的例子中是这样 http://localhost:8080）：

{% codeblock lang:nginx %}
    server {
        location / {
            proxy_pass http://localhost:8080;
        }
        location /images/ {
            root /data;
        }
    }
{% endcodeblock %}

我们将修改第二个location块，它将当前带有/images/前缀的请求映射到/data/images目录下，使其与具有典型文件扩展名的图像请求匹配。修改的location块如下所示：

{% codeblock lang:nginx %}
    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
{% endcodeblock %}

该参数是一个正则表达式，它会匹配所有以.gif、.jpg 或者.png结尾的URI。一个正则表达式需要以~开头。匹配到的请求会被映射到/data/images目录下。

当Nginx通过location去响应一个请求时，它会先检测带有前缀的location指令，兽先是检测带有**最长**前缀的 location，其次检测正则表达式。如果被正则的匹配的规则匹配成功，Nginx会选择使用该location的规则，否则，会选择之前缓存的规则。

最终，一个代理服务器的配置结果如下：

{% codeblock lang:nginx %}
    server {
        location / {
            proxy_pass http://localhost:8080/;
        }
        location ~ \.(gif|jpg|png)$ {
            root /data/images;
        }
    }
{% endcodeblock %}

该服务器将过滤以.gif、.jpg或者.png结尾的请求，并将它们映射到/data/images目录（通过添加URI到root指令的参数），并将所有其他请求传递给上述代理服务器。

要应用新配置，请保存修改后的配置文件，并nginx -s reload一下。

更多Nginx代理配置指令，尽在[ngx_http_proxy_module](https://nginx.org/en/docs/http/ngx_http_proxy_module.html)

## 参考文献

***

[Nginx官方网站](https://nginx.org/)
[Nginx官方入门文档](https://nginx.org/en/docs/beginners_guide.html)
