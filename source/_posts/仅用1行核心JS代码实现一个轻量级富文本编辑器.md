---
title: 仅用1行核心JS代码实现一个轻量级富文本编辑器
date: 2017-11-03 17:01:21
tags: JavaScript
description: 富文本编辑器是我们在生活中非常常用到的编辑工具，现在有很多功能完备且强大的编辑器，比如Quill Rich Text Editor、ueditor等，都是很优秀的富文本编辑器。甚至说我们每个人都会用到的word，才是最优秀、国民度最高的富文本编辑器。这篇文章使用极少的代码，实现了一个简洁、无任何依赖的轻量级富文本编辑器。
picture: zEditor.png
---

## 引子

把[demo](http://tuobaye.com/demo/zEditor/index)放在显眼的位置

***

富文本编辑器是我们在生活中非常常用到的编辑工具，现在有很多功能完备且强大的编辑器，比如[Quill Rich Text Editor](https://github.com/quilljs/quill)、[ueditor](http://ueditor.baidu.com/website/)等，都是很优秀的富文本编辑器。甚至说我们每个人都会用到的word，才是最优秀、国民度最高的富文本编辑器。

今天我们要实现一个轻量级的编辑器，主要利用的是[document.execCommand](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/execCommand)。

> 当一个HTML文档切换到设计模式(designMode)时，文档对象暴露 execCommand方法，该方法允许运行命令来操纵可编辑区域的内容。大多数命令影响文档的选择（粗体，斜体等），而其他命令插入新元素（添加链接）或影响整行（缩进）。当使用 contentEditable时，调用 execCommand() 将影响当前活动的可编辑元素。

用document.execCommand和contentEditable相互配合，就可以实现我们想要实现的功能。

在查阅了execCommand的文档后，我们决定实现以下功能

1. 选中文字样式调整
    - 斜体
    - 粗体
    - 下划线
    - 删除线
2. 对齐方式调整
    - 左对齐
    - 右对齐
    - 居中
    - 两端对齐
3. 缩进调整
    - 右缩进
    - 左缩进
4. 列表操作
    - 有序列表
    - 无序列表
5. 上下标
6. 文字操作
    - 全选
    - 复制
    - 粘贴
7. 基本的字号调整
8. 基本的颜色调整
9. 基本的字体调整
10. undo&redo

## 实现

***

说到编辑器，最基本的会分为上下两部分
> - 上部分为控制区域，用于对文本进行各种控制修改
> - 下部分为文本区域，用于输入和展示文本的样式

因此我们可以先在HTML里把两部分简单画出来

{% codeblock lang:html %}
<div id="wrapper">
    <div id="control-area"></div>
    <div id="text-area" contenteditable></div>
</div>
{% endcodeblock %}

注意，既然文本区域要可编辑，要加上contenteditable属性，这样就可以自如的在文本区域输入文字了。OK,编辑器雏形已经出来了~

接下来要对两部分分别进行操作，在控制区域，应该就是各种按钮。参考document.execCommand的语法：
![document.execCommand](execCommand.png)

主要控制到样式的有两个api：aCommandName和aValueArgument。同时，命令名aCommandName每条execCommand都会有，aCommandName和aValueArgument。因此我们需要在每个按钮上传递两个变量。第一个是必填项aCommandName，第二个是选填项aValueArgument。我们考虑采用HTML5的新属性data-*传递这两个参数:
{% codeblock lang:html %}
    <!--调整为斜体-->
    <a href="#" data-command='italic' onclick="changeStyle(this.dataset)">斜体</a>
    <!--调整字号为1号-->
    <a href="#" data-command='fontSize' data-value="1" onclick="changeStyle(this.dataset)">1号</a>
{% endcodeblock %}

上面是两个基本按钮示例，一个是不传参的，一个是传参的，然后在js端做一个判断即可实现功能：

{% codeblock lang:javascript %}
    const changeStyle = (data) => {
        //一行核心代码即可实现基本编辑器功能
        data.value? document.execCommand(data.command, false, data.value):document.execCommand(data.command, false, null)
    }
{% endcodeblock %}

是不是很简单呢？

最后，只有两个按钮肯定是不够的，我们要把按钮数和编辑器的功能扩充丰富起来。我们就按照第一节整理的决定要实现的功能列表列出来的功能，按照分类，一一来实现。最后，再把按钮和文本区域的样式美化一下，即可实现我们这个轻量级富文本编辑器啦~

这是我们的预览图：
![zEditor](zEditor.png)

大家可以访问[我的博客](http://tuobaye.com/demo/zEditor/index)尝试一下这个小小富文本编辑器，也希望能去我的[github项目](https://github.com/tuobaye0711/zEditor)上点颗star,谢谢啦~

顺道贴一个codepen的预览，有些网络可能加载不出来...
<p data-height="744" data-theme-id="dark" data-slug-hash="LOZRaL" data-default-tab="html,result" data-user="tuobaye0711" data-embed-version="2" data-pen-title="zEditor" class="codepen">See the Pen <a href="https://codepen.io/tuobaye0711/pen/LOZRaL/">zEditor</a> by zhleven (<a href="https://codepen.io/tuobaye0711">@tuobaye0711</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

## 结语

***

实际上，我们这个富文本编辑器是一种最最简单的实现，仅仅是对contentEditable和document.execCommand进行了一层封装，所有的样式实现都是调用的这同一条api。

在当前流行的真正功能强大的富文本编辑器中，基本都是实现了自己的contentEditable，抛弃了对浏览器原生的contentEditable特性的依赖。大家都说富文本编辑器是个天坑，事实也正是如此，原生的document.execCommand实现的功能太少，BUG太多，如果使用自己写的功能来编辑文字，又会破坏contentEditable的undo/redo栈。总之，contentEditable最大的特点就是样式、html语意环境跟调用页面混合在一起，非常容易出现覆盖现象，而牛逼的富文本编辑器基本上都会使用一套自己的事件监听机制来实现类似contentEditable的功能，这些东西太繁琐太复杂，不在这里的讨论范围之内了~