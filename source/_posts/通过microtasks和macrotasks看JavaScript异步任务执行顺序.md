---
title: 通过microtasks和macrotasks看JavaScript异步任务执行顺序
date: 2017-10-24 11:08:09
tags: JavaScript
---
### 1. 初探 --- setTimeout()的那些事儿

***

相信很多人在初学JavaScript的时候都遇到过类似的代码：


{% codeblock lang:javascript %}
    // part1
    console.log(1);
    setTimeout(function(){
        console.log(2);
    },100);
    console.log(3);
    console.log(4);
    // part2
    console.log(1);
    setTimeout(function(){
        console.log(2);
    },0);
    console.log(3);
    console.log(4);
{% endcodeblock %}

萌新们看到这个，肯定觉得很简单。part1输出顺序是1,3,4,2;part2输出顺序是1,2,3,4嘛。显而易见，part1中在打印2的时候延迟了100ms，所以被放到了队列的尾端执行，理所当然的最后输出；part2中虽然调用了setTimeout函数，但是延迟设置为0ms，实际上并未延迟，因此应该立即执行，所以输出顺序应该是1,2,3,4。

看到这里，很多人应该都知道了，上面的说法实际上是错误的。

> setTimeout(fn,0)的含义是，指定某个任务在主线程最早可得的空闲时间执行，也就是说，尽可能早得执行。它在"任务队列"的尾部添加一个事件，因此要等到同步任务和"任务队列"现有的事件都处理完，才会得到执行。

上面的这段文字摘自[阮一峰的博客](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)，即使你的延迟设定的是0，setTimeout仍然会将你的函数给放到队列最尾端，等你的当前任务执行完毕以后,才会去执行该函数。

### 2. 深入 --- setTimeout()和Promise同场竞技时是什么样呢？

***

众所周知，Promise是ES6发布的一种非常流行的异步实现方案，当Promise和setTimeout同时出现在一段代码中，他们的执行顺序是什么样子的呢？请看下面这段代码：

{% codeblock lang:javascript %}
    setTimeout(function(){
        console.log(1)
    },0);
    new Promise(function(resolve){
        console.log(2)
        for( var i=100000 ; i>0 ; i-- ){
            i==1 && resolve()
        }
        console.log(3)
    }).then(function(){
        console.log(4)
    });
    console.log(5);
{% endcodeblock %}

如果按照正常逻辑分析，应该是这样的：
1. 当运行到setTimeout时，会把setTimeout的回调函数console.log(1)放到任务队列里去，然后继续向下执行。
2. 接下来会遇到一个Promise。首先执行打印console.log(2)，然后执行for循环，即时for循环要累加到10万，也是在执行栈里面，等待for循环执行完毕以后，将Promise的状态从fulfilled切换到resolve，随后把要执行的回调函数，也就是then里面的console.log(4)推到任务队列里面去。接下来马上执行马上console.log(3)。
3. 然后出Promise，还剩一个同步的console.log(5)，直接打印。这样第一轮下来，已经依次打印了2，3，5。
4. 然后再读取任务队列，任务队列里还剩console.log(1)和console.log(4)，因为任务队列是队列嘛，肯定遵循的先进先出的策略，因此更早入列的setTimeout()的回调函数先执行，打印1，最后剩下Promise的回调，打印4。

因此一通分析下来，得到的打印结果是2,3,5,1,4。那我们实际试一下呢？

{% codeblock lang:javascript %}
    2
    3
    5
    4
    1
{% endcodeblock %}

啊嘞嘞？跟我们一开始想象的貌似有点不一样啊！是什么原因导致了原本应该在setTimeout回调后面的Promise的回调反而跑到前面去执行了呢？

为了搞清这个问题，我专门去翻阅了一下资料，首先找到了[Promises/A+标准](https://promisesaplus.com/#point-67)里面提到：

>   Here “platform code” means engine, environment, and promise implementation code. In practice, this requirement ensures that onFulfilled and onRejected execute asynchronously, after the event loop turn in which then is called, and with a fresh stack. <span style="color:red">This can be implemented with either a “macro-task” mechanism such as setTimeout or setImmediate, or with a “micro-task” mechanism such as MutationObserver or process.nextTick. </span>Since the promise implementation is considered platform code, it may itself contain a task-scheduling queue or “trampoline” in which the handlers are called.

这里提到了micro-task和macro-task这两个概念，并分别列举了两种情况：setTimeout和setImmediate属性macro-task，MutationObserver和process.nextTick属性micro-task。但并没有进一步的详述，于是我以此为线索进一步搜索资料，找到[stackoverflow上的一个问答](https://stackoverflow.com/questions/25915634/difference-between-microtask-and-macrotask-within-an-event-loop-context)，终于让我的疑惑得到解决。

macrotasks和microtasks的划分：

#### macrotasks:
- setTimeout
- setInterval
- setImmediate
- requestAnimationFrame
- I/O
- UI rendering

#### microtasks:
- process.nextTick
- Promises
- Object.observe
- MutationObserver

那我们上面提到的任务队列到底是什么呢？跟macrotasks和microtasks有什么联系呢？

>   - An event loop has one or more task queues.(task queue is macrotask queue)
  - Each event loop has a microtask queue.
  - task queue = macrotask queue != microtask queue
  - a task may be pushed into macrotask queue,or microtask queue
  - when a task is pushed into a queue(micro/macro),we mean preparing work is finished,so the task can be executed now.

翻译一下就是：
- 一个事件循环有一个或者多个任务队列；
- 每个事件循环都有一个microtask队列
- macrotask队列就是我们常说的任务队列，microtask队列不是任务队列
- 一个任务可以被放入到macrotask队列，也可以放入microtask队列
- 当一个任务被放入microtask或者macrotask队列后，准备工作就已经结束，这时候可以开始执行任务了。

可见，setTimeout和Promises不是同一类的任务，处理方式应该会有区别，具体的处理方式有什么不同呢？我从[这篇文章](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/?utm_source=html5weekly&utm_medium=email)里找到了下面这段话：

> Microtasks are usually scheduled for things that should happen straight after the currently executing script, such as reacting to a batch of actions, or to make something async without taking the penalty of a whole new task. The microtask queue is processed after callbacks as long as no other JavaScript is mid-execution, and at the end of each task. Any additional microtasks queued during microtasks are added to the end of the queue and also processed. Microtasks include mutation observer callbacks, and as in the above example, promise callbacks.

通俗的解释一下，<font color="red">microtasks的作用是用来调度应在当前执行的脚本执行结束后立即执行的任务。</font>  例如响应事件、或者异步操作，以避免付出额外的一个task的费用。

microtask会在两种情况下执行：
1. 任务队列(macrotask = task queue)回调后执行，前提条件是当前没有其他执行中的代码。
2. 每个task末尾执行。

另外在处理microtask期间，如果有新添加的microtasks，也会被添加到队列的末尾并执行。

也就是说执行顺序是：

开始 -> <font color="red">取task queue第一个task执行 -> 取microtask全部任务依次执行 -> 取task queue下一个任务执行 -> 再次取出microtask全部任务执行 -> ...  </font>  这样循环往复


> Once a promise settles, or if it has already settled, it queues a microtask for its reactionary callbacks. This ensures promise callbacks are async even if the promise has already settled. So calling .then(yey, nay) against a settled promise immediately queues a microtask. This is why promise1 and promise2 are logged after script end, as the currently running script must finish before microtasks are handled. promise1 and promise2 are logged before setTimeout, as microtasks always happen before the next task.

Promise一旦状态置为完成态，便为其回调(.then内的函数)安排一个microtask。

接下来我们看回我们上面的代码

{% codeblock lang:javascript %}
    setTimeout(function(){
        console.log(1)
    },0);
    new Promise(function(resolve){
        console.log(2)
        for( var i=100000 ; i>0 ; i-- ){
            i==1 && resolve()
        }
        console.log(3)
    }).then(function(){
        console.log(4)
    });
    console.log(5);
{% endcodeblock %}

按照上面的规则重新分析一遍：
1. 当运行到setTimeout时，会把setTimeout的回调函数console.log(1)放到任务队列里去，然后继续向下执行。
2. 接下来会遇到一个Promise。首先执行打印console.log(2)，然后执行for循环，即时for循环要累加到10万，也是在执行栈里面，等待for循环执行完毕以后，将Promise的状态从fulfilled切换到resolve，<font color="red">随后把要执行的回调函数，也就是then里面的console.log(4)推到microtask里面去。</font>接下来马上执行马上console.log(3)。
3. 然后出Promise，还剩一个同步的console.log(5)，直接打印。这样第一轮下来，已经依次打印了2，3，5。
4. 现在第一轮任务队列已经执行完毕，没有正在执行的代码。符合上面讲的microtask执行条件，因此会将microtask中的任务优先执行，因此执行console.log(4)
5. 最后还剩macrotask里的setTimeout放入的函数console.log(1)最后执行。

如此分析输出顺序是：

{% codeblock lang:javascript %}
    2
    3
    5
    4
    1
{% endcodeblock %}

看吧，这次分析对了呢ヾ(◍°∇°◍)ﾉﾞ

### 3. 总结和参考资料

***

microtask和macrotask看起来很容易混淆，实际上还剩容易区分的。macrotask就是我们常说的任务队列(task queue)。

JavaScript执行顺序可以简要总结如下：

开始 -> <font color="red">取task queue第一个task执行 -> 取microtask全部任务依次执行 -> 取task queue下一个任务执行 -> 再次取出microtask全部任务执行 -> ...  </font>

循环往复，直至两个队列全部任务执行完毕。

#### 参考资料

[Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/?utm_source=html5weekly&utm_medium=email)
[github-setImmediate.js](https://github.com/YuzuJS/setImmediate)
[Promises/A+](https://promisesaplus.com/#point-67)
[知乎-Promise的队列与setTimeout的队列有何关联？](https://www.zhihu.com/question/36972010)
[stack overflow-Difference between microtask and macrotask within an event loop context
](https://stackoverflow.com/questions/25915634/difference-between-microtask-and-macrotask-within-an-event-loop-context)