---
title: 【译】Chrome浏览器开发者工具的13个有趣技巧——希望你已经掌握
date: 2018-03-26 11:27:06
tags: 外文翻译
description: Chrome浏览器提供了一套非常出色的开发者工具来帮助您在Web平台上开发。下面我将介绍13个有趣的技巧，掌握他们对你只有好处。
picture: 6.gif
---

Chrome浏览器提供了一套非常出色的开发者工具来帮助您在Web平台上开发。下面我将介绍13个有趣的技巧，掌握他们对你只有好处。

###  1. 在Elements面板中拖放元素

***

在Elements面板中，你可以拖放任意的HTML元素，并在整个页面中更改其位置：

![Drag-and-drop in the Elements panel](1.gif)

###  2. 在控制台中引用当前选定的元素

***

在Elements面板中，选择一个节点，然后输入$0控制台以引用它：

![Reference the currently selected element in the Console](2.gif)

###  3. 使用控制台中最后一个操作的值

***

使用$_引用在控制台执行的前一操作的返回值：

![Use the value of the last operation in the Console](3.gif)

###  4. 添加CSS并编辑元素状态

***

在Elements面板中有2个超级有用的按钮。

第一个可以为你的选择器(selector)增加新的CSS属性：

![Add CSS and edit the element state](4.gif)

第二个可以为你触发选定元素的状态，这样你就可以看到当它处于活动状态(active)，悬停状态(hovered)，焦点状态(focus)等等时应用的样式：

![Add CSS and edit the element state](4.2.png)

###  5. 将修改后的CSS保存到文件

***

点击你编辑的CSS文件的名称，进入到Sources面板，你会发现你的修改已经在里面了。然后你可以对你实时的编辑进行保存。

这个修改不适用于添加的新选择器，也不适用于element.style属性，仅仅适用于原有选择器。

![Save to file the modified CSS](5.gif)

###  6. 截图单个元素

***

选择一个元素，MAC下按cmd+shift+p、windows下按ctrl+shift+p来打开命令菜单，然后输入**Capture node screenshot**：

![Screenshot a single element](6.gif)

###  7. 使用CSS选择器查找元素

***

MAC下按cmd+f、windows下按ctrl+f来打开搜索框

您可以在其中键入任何字符串以匹配源代码，或者也可以使用CSS选择器让Chrome为您生成一个图像：

![Find an element using CSS selectors](7.gif)

###  8. 控制台中的shift和enter

***

要编写跨越控制台多行的命令，请按shift+enter

准备就绪后，在脚本末尾按Enter键即可执行该操作：

![Shift-enter in the Console](8.gif)

###  9. 跳转

***

在Sources面板中：

- cmd+o(windows下ctrl+o) 显示你页面加载的所有文件

- cmd+shift+o(windows下ctrl+shift+o) 显示当前文件中的symbols（属性、函数、类等）

- cmd+o(windows下ctrl+o) 跳转到指定行

准备就绪后，在脚本末尾按Enter键即可执行该操作：

![Go to…](9.png)

###  10. 监听表达式

***

不需要一次又一次地输入一个变量名或者表达式，你只需将他们添加到监视列表中就可以时时观察它们的变化：

![Watch Expression](10.gif)

###  11. XHR/Fetch调试

***

从调试器打开**XHR/Fetch Breakpoints**面板。

你可以针对某一个请求或者请求的关键字设置断点：

![XHR/Fetch debugging](11.png)

###  12. 调试DOM修改

***

右键单击某个DOM元素，并选择Break on下的subtree modifications。这样调试器就可以在脚本遍历到该元素并且要修改它的时候自动停止，以让用户进行调试检查。

![XHR/Fetch debugging](12.png)

###  13. 找到CSS属性定义的位置

***

MAC使用cmd+鼠标左键(windows下使用ctrl+鼠标左键)点击Elements面板中的CSS属性，可以直接帮您定位到Source面板中相应CSS定义的位置：

![Use the value of the last operation in the Console](13.gif)

### 原文链接

***

[https://flaviocopes.com/chrome-devtools-tips/](https://flaviocopes.com/chrome-devtools-tips/)