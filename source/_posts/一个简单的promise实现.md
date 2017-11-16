---
title: 一个简单的promise实现
date: 2017-10-17 15:54:45
tags: JavaScript
description: Promise 是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。它由社区最早提出和实现，ES6 将其写进了语言标准，统一了用法，原生提供了Promise对象。本文采用最简单的方法，实现了一个基本的Promise雏形
picture: promise.png
---
![promise/A+](promise.png)
> Promise 是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。它由社区最早提出和实现，ES6 将其写进了语言标准，统一了用法，原生提供了Promise对象。

通常我们是这么定义和使用promise的：
{% codeblock lang:javascript %}
    function getSomethingAsync() {
        return new Promise(function(resolve) {
            //异步请求,这里简单的用setTimeout模拟异步
            setTimeout(function(){
                resolve('got it!');
            },5000)
        })
    }
    getSomethingAsync().then(function(id) {
        //一些处理
        let information = id;
        console.log(information)
    })
{% endcodeblock %}

这例中主要涉及了Promise的两个核心，一个是then，一个是resolve。据此，我们可以写出一个最基础的Promise的架子：

{% codeblock lang:javascript %}
    function Promise(fn) {
        //value用来存储向resolve传递的值，callbacks用来存储已注册的回调函数
        var value = null,
            callbacks = [];  //callbacks为数组，因为可能同时有很多个回调
        this.then = function (onFulfilled) {
            //将注册的回调函数依次压入栈中
            callbacks.push(onFulfilled);
        };
        function resolve(value) {
            //当调用resolve函数时，根据传入的value，依次执行已注册的回调函数
            callbacks.forEach(function (callback) {
                callback(value);
            });
        }
        fn(resolve);
    }
{% endcodeblock %}

在这里，我们分别分析一下then和resolve的作用,首先是then：

<img src='Promise.prototype.then.png' width="50%" height="50%">

注册回调函数的作用很明显，就是在Promise的异步操作执行成功时，将需要执行的回调函数压入callbacks队列中。

同时，then函数要实现类似于

{% codeblock lang:javascript %}
    getSomethingAsync().then(function() {
        //一些处理
    }).then(function(){
        //二些处理
    }).then(function(){
        //三些处理
    })
{% endcodeblock %}

的功能，即链式调用，仅需一行代码即可实现：

{% codeblock lang:javascript %}
    this.then = function (onFulfilled) {
        //将注册的回调函数依次压入栈中
        callbacks.push(onFulfilled);
        //一行代码实现链式调用
        return this
    };
{% endcodeblock %}

是不是很巧妙呢？
***
实现了then的基本功能，我们再回过头来看resolve。resolve函数的作用是，将Promise对象的状态从“未完成”变为“成功”（即从 pending 变为 resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去。这样，当Promise实例生成以后，可以用then方法指定resolved状态。

由此可见，resolve状态下，还有一件必须要完成的事情是状态切换:
- 对象的状态不受外界影响。Promise对象代表一个异步操作，有三种状态：pending（进行中）、fulfilled（已成功）和rejected（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是Promise这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。
- 一旦状态改变，就不会再变，任何时候都可以得到这个结果。Promise对象的状态改变，只有两种可能：从pending变为fulfilled和从pending变为rejected。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果。

pending -> fulfilled/pending -> rejected

我们接下来就在resolve里面加上状态切换：

{% codeblock lang:javascript %}
    function Promise(fn) {
        var value = null,
            callbacks = [],
            //初始状态置为pending
            state = 'pending';
        this.then = function(onFulfilled) {
            //假如调用then函数时，状态仍为pending，这时候需要将回调函数压入到callbacks栈中
        	if (state === 'pending') {
        		callbacks.push(onFulfilled);
                return this
        	}
        	//假如调用then函数时，状态不是pending，则直接调用
            onFulfilled(value);
            return this
        }
        function resolve(value) {
        	value = newValue;
        	//将状态切换为fulfilled
        	state = 'fulfilled';
            callbacks.forEach(function(callback) {
                callback(value)
            })
        }
        fn(resolve)
    }
{% endcodeblock %}

这样，我们就实现了一个最基础的Promise。等等，是不是还缺了什么？如果在then方法注册回调之前，resolve函数就执行了，怎么办？比如promise内部的函数是同步函数：

{% codeblock lang:javascript %}
    function getSomething() {
        return new Promise(function(resolve) {
            resolve(8384);
        })
    }
    getSomething().then(function (id) {
        // 一些处理
    });
{% endcodeblock %}

这么做显然不对，我们必须将resolve放到队列尾端，以保证在resolve执行前，所有then函数的回调都已经注册完毕，因此我们再加一个延时：

{% codeblock lang:javascript %}
    function resolve(value) {
        value = newValue;
        state = 'fulfilled';
        // 放到队列尾端执行回调
        setTimeout(function() {
            callbacks.forEach(function(callback) {
                callback(value)
            })
        }, 0)
    }
{% endcodeblock %}

这样，我们一个最基本的Promise就写完了，为了便于理解逻辑，暂时还没加rejected的状态切换，后面有机会咱再继续丰富我们的Promise源码~下面贴一下完整的代码：

{% codeblock lang:javascript %}
    function Promise(fn) {
      var value = null,
        callbacks = [],
        state = "pending";
      this.then = function(onFulfilled) {
        if (state === "pending") {
          callbacks.push(onFulfilled);
          return this; //一行即可实现链式调用
        }
        onFulfilled(value);
        return this;
      };
      function resolve(newValue) {
        value = newValue;
        state = "fulfilled";
        setTimeout(function() {
          callbacks.forEach(function(callback) {
            callback(value);
          });
        }, 0);
      }
      fn(resolve);
    }
{% endcodeblock %}

点击[这里](https://codepen.io/tuobaye0711/pen/BwMQyL?editors=0010)查看codepen完整代码示例~