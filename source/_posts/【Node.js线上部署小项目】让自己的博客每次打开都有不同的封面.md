---
title: 【Node.js线上部署小项目】让自己的博客每次打开都有不同的封面
date: 2017-12-26 17:16:00
tags: Node.js
picture: wallpaper.jpg
description: 是不是看腻了自己博客的背景图片？赶紧来自己动手做一个吧！看完这篇文章，你将会从零开始学会写一个实用的小项目，然后在线上部署，在相当低成本的情况下将自己的接口暴露出去，造福自己，造福别人~
---

欢迎光临我的博客[拓跋的前端客栈](http://tuobaye.com)，这个是[原文地址](http://http://tuobaye.com/2017/12/26/%E3%80%90Node.js%E7%BA%BF%E4%B8%8A%E9%83%A8%E7%BD%B2%E5%B0%8F%E9%A1%B9%E7%9B%AE%E3%80%91%E8%AE%A9%E8%87%AA%E5%B7%B1%E7%9A%84%E5%8D%9A%E5%AE%A2%E6%AF%8F%E6%AC%A1%E6%89%93%E5%BC%80%E9%83%BD%E6%9C%89%E4%B8%8D%E5%90%8C%E7%9A%84%E5%B0%81%E9%9D%A2/)，这个是[项目地址](https://github.com/tuobaye0711/new-wallpaper-everyday)，这个是[线上部署地址](http://tuobaye.duapp.com/)。如果您发现我文章中存在错误，请尽情向我吐槽，大家一起学习一起进步φ(>ω<*)

## 1、引子

***

为什么想做这个功能呢？起因很简单————我看腻了自己博客的封面(特地要提一下，封面是来自[huno](https://github.com/letiantian/huno))。

![old cover](background-cover.jpg)

虽然我的封面很好看的说，但是谁还没有个审美疲劳不是？

毕竟再好看的图片都有看腻的一天，为了克服审美疲劳，最好能经常换换封面，每天都不一样就更好了！

想到了就去做，动手~

## 2、选图源

***

说起每天一张美图，那大伙第一个想到的肯定是[必应](https://www.bing.com)啦，必应的搜索引擎虽然大多数人用的不多——翻墙的用谷歌的多，不翻墙的用百度的多。但是必应搜索的首页真的很亮眼~就是因为每天都有不同的美图做背景，而且根据地理位置还有不同的图片：

![bingcn](bingcn.png)
![bingen](bingen.png)

既美又高清，很适合做壁纸。就是他了！

既然要做接，首先要抓取bing的api，emmm，打开bing主页，F12，点击Network-XHR，挨个找一下，很容易就能发现在哪里，看图~

![xhr](xhr.png)

很容易找到是哪条请求。其实这个bing的壁纸接口在网上一搜也能搜到了，[看这里](https://stackoverflow.com/questions/10639914/is-there-a-way-to-get-bings-photo-of-the-day)。
 
 接口格式是这样的：
 
> XML: http://www.bing.com/HPImageArchive.aspx?format=xml&idx=0&n=1&mkt=en-US
> JSON: http://www.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1&mkt=en-US
> RSS: http://www.bing.com/HPImageArchive.aspx?format=rss&idx=0&n=1&mkt=en-US

- *format*表示返回格式，可选字段有xml、js（json）、rss
- *idx*=x表示获取从第x天前开始的图片，最大值为7，n>7时返回的都是n=7的值
- *n*=y表示获取从idx开始连续y天的图片，最大值为9，同时限制最多能获取15天以内的图片
- *mkt*表示地区信息，已知的只有zh-CN和en-US，是否还有其他的我也不清楚

返回值如下：

{% codeblock lang:javascript %}
    {
        "images": [
            {
                "startdate": "20171226", 
                "fullstartdate": "201712261600", 
                "enddate": "20171227", 
                "url": "/az/hprichbg/rb/CPNYSnow_ZH-CN13335620157_1920x1080.jpg", 
                "urlbase": "/az/hprichbg/rb/CPNYSnow_ZH-CN13335620157", 
                "copyright": "中央公园，美国纽约市 (© Nisian Hughes/Getty Images)", 
                "copyrightlink": "http://www.bing.com/search?q=%E4%B8%AD%E5%A4%AE%E5%85%AC%E5%9B%AD&form=hpcapt&mkt=zh-cn", 
                "quiz": "/search?q=Bing+homepage+quiz&filters=WQOskey:%22HPQuiz_20171226_CPNYSnow%22&FORM=HPQUIZ",
                "wp": true, 
                "hsh": "381850a3d2d57acd0ded240e42ffec8e", 
                "drk": 1, 
                "top": 1, 
                "bot": 1, 
                "hs": [ ]
            }
        ], 
        "tooltips": {
            "loading": "正在加载...", 
            "previous": "上一个图像", 
            "next": "下一个图像", 
            "walle": "此图片不能下载用作壁纸。", 
            "walls": "下载今日美图。仅限用作桌面壁纸。"
        }
    }
{% endcodeblock %}

我们只要在 www.bing.com 后面拼上images[0].url就是我们想要的图片的链接了~还是1920x1080的高清图片呢，拿来做壁纸正合适~

有了这些已知信息，我们终于可以愉快的撸代码了φ(>ω<*)

## 3、撸代码

***

我选择采用Node.js+express来搭建我们的后台代码。并做一个规划，我们的接口要能返回当日的图片，能根据入参返回几天前的哪个地区的图片，哦对了，能随机返回图片，保证咱每次看到的都不一样。

很简单，说干就干。

关于怎么搭建一个Node.js+express服务我就不废话了，这几个接口都要用到请求uri，简单写个公共函数用来获取uri：

{% codeblock lang:javascript %}
    function getUri(start, number, mkt) {
        return 'https://www.bing.com/HPImageArchive.aspx?format=js&idx=' + start + '&n=' + number + '&mkt=' + mkt
    }
{% endcodeblock %}

另外由于获取随机图片和获取按参数指定的图片肯定是两个不同的接口，但是他们同时要用到一个请求并处理返回值的过程，为了代码好看，我们把他抽离出来：

{% codeblock lang:javascript %}
    function getWallpaper(res, days_ago, mkt) {
        let uri;
        if (days_ago <= 7){
            uri = getUri(days_ago, 1, mkt)
        }else {
            uri = getUri(7, days_ago-6, mkt)
        }
        request(uri, function (error, response, body) {
            if (!error && response.statusCode === 200) {
                let data = JSON.parse(body);
                let images = data.images;
                res.redirect('https://www.bing.com'+images[images.length-1].url)
            }else{
                res.send('request error!')
            }
        })
    }
{% endcodeblock %}

函数很简单，看一眼就明白了，至于为什么返回的时候使用res.redirect()，这跟我的目的有关系。

我一开始就提到了，我要实现自己博客每天更换封面的效果，博客的封面使用的是CSS的background样式，这个样式虽然可以用本地图片也可以用线上图片，但是必须是图片！由于我们的服务自己不做存储，因此直接把要获取到的图片的根地址返回过去才是最优解，这就是我们选择使用res.redirect()的原因。

还有随机接口，我们只要写个自定义范围的随机函数打乱就可以了：

{% codeblock lang:javascript %}
    function getRandomInteger(min, max) {
        return Math.floor(Math.random() * (max - min + 1)) + min;
    }
{% endcodeblock %}

函数getRandomInteger(min, max)返回min和max之间的任意整数。使用这个函数不仅可以随机获取0-15天内的某个天数，也可以使用ta获取随机的地区：

{% codeblock lang:javascript %}
    let mkt = getRandomInteger(0,1) ? 'zh-CN' : 'en-US';
{% endcodeblock %}

其他都没什么难度了，就是传参。可以自己去看[源码](https://github.com/tuobaye0711/new-wallpaper-everyday)~

## 4、BAE线上部署

***

既然我们自己写的接口希望能用在自己的个人网站上，接口肯定不能只能在本地调用吧？我们接下来要做的就是把博客线上部署啦，这样我们就能把接口暴露在互联网上，造福自己，造福大家~

因为我们的项目相当之轻量，不占存储，不占内存，基本上就是做一个转发的功能，因此不需要什么很高的服务器性能。为了这个项目还要自己租用一个云服务器那就得不偿失了~在你自己本身没有服务器的前提下，我推荐你使用百度的BAE进行线上部署(Baidu Application Engine)，一天只要两毛钱，用到天长地久（我都有点替百度心疼资源o(TωT)o ）。 

### 部署步骤

1. 首先登陆[百度云](https://console.bce.baidu.com)的首页，没有注册的注册一下，注册以后实名认证一下。这里不细说了~
2. 点击左侧菜单栏的应用引擎BAE，界面如图所示，点击“添加部署”。![step2](step2.png)
3. 界面如图所示，按你的需要选择配置，我选择的是这样的（请无视右侧的重叠框，滚动截图工具不支持position: fixed定位方式的锅）。其中，由于我要跑的是Node.js代码，因此类型选择Node能支持的最高版本。代码版本工具依靠个人喜好，我喜欢git。内存选最低，单元个数选最少，OK，一天只需要两毛钱~点击下一步。![step3](step3.png)
4. 点击“去支付”即可，后付费方式，账号里没有余额也能支付成功。![step4](step4.png)
5. OK,开通成功了，去控制台看看吧~![step5](step5.png)
6. 在部署列表里，直接点名称进到里面，选择发布设置。这里可以看到git地址，我们这就可以通过git clone的方式把代码库同步到本地，然后把本地的代码放到里面，push上来，就大功告成啦~对了，最好把自动发布设置打开，这样每次更新代码都能自动在线上部署最新的版本咯。![step6-1](step6-1.png) ![step6-2](step6-2.png)
7. 最后，调用一下我的接口试试吧：tuobaye.duapp.com

### BAE部署过程中的几个坑

1. BAE代码只能监听18080，要把原来的端口设置改为18080；
2. BAE默认不支持ES6语法，所以原来你代码里的let/const什么的老老实实改成var吧，
3. 我在上传的过程中，把node_modules省略了，然后在package.json里面写上了dependencies，但是报错了。我的解决办法是直接把node_modules上传，把package.json的dependencies删掉，只让他自动执行启动脚本，不管别的，就启动成功了。
4. [BAE文档](https://cloud.baidu.com/doc/BAE/Nodejs.html)，有其他问题可以看看这个，有一定参考价值。

OK，搞定啦

## 5、修改背景，目的达成！

我的博客是用hexo生成的，博客的背景图片在主题文件的样式里面。每个人都有自己的不同情况，但是只要找到这一行样式就行了。

{% codeblock lang:css %}
    .panel-cover {
      display: block;
      position: fixed;
      z-index: 900;
      width: 100%;
      max-width: none;
      height: 100%;
      background: url(../images/background-cover.jpg) top left no-repeat #666666;
      background-size: cover;
    }
{% endcodeblock %}

把background改成我们自己的接口，这个接口可以获取当天的壁纸图片：

{% codeblock lang:css %}
    .panel-cover {
      display: block;
      position: fixed;
      z-index: 900;
      width: 100%;
      max-width: none;
      height: 100%;
      background: url(http://tuobaye.duapp.com/wallpaper) top left no-repeat #666666;
      background-size: cover;
    }
{% endcodeblock %}

对比一下，是不是很有成就感~？

![before](before.png)
![after](after.png)

如果把url换成 http://tuobaye.duapp.com/wallpaper/random ，就可以每刷新一次都是新的封面了~帅气！

## 6、小结

***

看完这篇文章，你将会从零开始学会写一个实用的小项目，然后在线上部署，在相当低成本的情况下将自己的接口暴露出去，造福自己，造福别人~

从想到换封面的点子，到选图源，抓接口，撸代码，BAE部署到最后自己用上自己最新鲜的接口，实际的工作量一共也不超过一天时间，其中还有一大半时间都是在踩BAE的坑。

很多时候，限制我们的并不是我们的技术，而是创意和实干的心。

虽然项目简单，但是能对读到这里的读者一点启发作用，那我觉得就很知足了~

想到就去做，Just do it!