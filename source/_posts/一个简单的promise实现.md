---
title: 一个简单的promise实现
date: 2017-10-17 15:54:45
tags: javascript
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