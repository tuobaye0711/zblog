---
title: 仅用1行核心JS代码实现一个轻量级富文本编辑器
date: 2017-11-03 17:01:21
tags: JavaScript
---

## 引子

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

注意，既然文本区域要可编辑，要加上contenteditable属性，这样就可以自如的在文本区域输入文字了。
