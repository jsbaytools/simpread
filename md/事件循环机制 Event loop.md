\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[juejin.im\](https://juejin.im/post/6884987711074091016)

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6927444c3cee4b07979deb57c2a974d7~tplv-k3u1fbpfcp-watermark.image)

导言
==

我们经常听到这样一句话，Javascript 是一门在`单线程`环境下运行的语言，什么是单线程呢？就是同一时间只能做一件事那为什么他不能设计成多线程呢？这样就能在同一时间做多件事。

想法是挺美好的，但是呢，Javascript 的诞生就是为了`与用户交互`，以及`操作DOM`，假设是多线程，其中一个线程的工作是删除某个 DOM 节点，另一个线程的作用又是在这个节点里面添加一些东西，这个时候我们该以哪个线程为主呢？？？

有的人可能会想到 html5 的`web worker`，他允许 Javascript 脚本同时创建多个线程，在 mdn 中，他是这样定义的： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e45c113f639a4e73a19ecfb73f67e441~tplv-k3u1fbpfcp-watermark.image) 也就是说 web workers 是子线程，由主线程控制，也就是说他相当于主线程的`辅助`，为了避免主线程被`阻塞`或者`减速`。

浏览器下的事件循环机制
===========

调用栈和任务队列
--------

栈是一种结构化的内存，遵循 LIFO（先进后出）的原则

队列也是一种结构化的内存内存，遵循 FIFO（先进先出）的原则

*   调用栈

上面说的是两种常见的数据结构，我们的`调用栈`（也叫执行栈）和上面的`栈`的定义又有点不同，它指的是一种代码的运行方式， 维基百科的定义是这样的：

```
 a call stack is a stack data structure that stores information about the active subroutines of a computer program

the caller pushes the return address onto the stack, and the called subroutine, when it finishes, pulls or pops the return address off the call stack and transfers control to that address. If a called subroutine calls on yet another subroutine, it will push another return address onto the call stack, and so on.
复制代码

```

被调用函数或子例程的`返回的地址`会被推入执行栈中，当这个过程完成后，返回的地址就会从调用栈中弹出，并且将这种控制调用栈的权利`转移`给该地址

如果调用函数（子例程）中又`调用`了其他函数（子例程），就又会`重复`上述操作。

以函数举例，也就是说，当函数被`调用`使，他就会`进入`执行栈，然后执行函数，遇到`返回`（return）的时候，这个函数就会被`弹出栈`，如果函数返回的又是一个`函数`，就会实现一种`层层调用`。

举个栗子🌰

```
function one() {
  return 1;
}

function two() {
  return one() + 1;
}

function three() {
  return two() + 1;
}

console.log(three());

复制代码

```

他的调用栈如下（动图来源于下面第四个参考链接）： ![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/30736e4e6d22484fb1f0df69b90e647b~tplv-k3u1fbpfcp-watermark.image)

*   任务队列（task queue）

javascript 是单线程运行的，也就是说，所有的任务都是`排队执行`的，只有一个任务被处理`完成后`，才会去处理另一个任务。

如果有的任务处理需要消耗很多时间，就会带来`阻塞`。

而 javascript 的一大特点就是`非阻塞`的，实现非阻塞主要是依靠`任务队列`这一机制。

Javascript 引擎执行代码是一段一段执行的，在执行一段代码时，他会先判断所执行的任务是同步任务（synchronous）还是异步任务（asynchronous），如果是同步任务，那么直接进入主线程，异步任务执行完后，得到的`回调函数`，进入任务队列。

```
任务进入执行栈中，判断是同步还是异步

若是同步，直接进入主线程按照调用栈的顺序被执行

若是异步，则进行一些处理，再将回调函数推入任务队列

当主线程中的同步任务执行完成后，执行栈为空，开始读取任务队列

任务队列中的任务依次进入执行栈被执行，直到任务队列为空

完成
复制代码

```

如图：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e48efcbb05964d31856e76ec2a719b06~tplv-k3u1fbpfcp-watermark.image)

再举个栗子🌰

```
1 console.log('a');

2 setTimeout(function () {
  console.log('b');
}, 4000);

3 setTimeout(function () {
  console.log('c');
}, 0);

4 console.log('d');
复制代码

```

执行的结果是什么呢？

`a d c b`

任务进行执行栈，遇到`1`, 是同步任务，立即执行 于是答案中就有了`a` ，

再执行到`2`和`3`, 是异步任务，进入异步处理模块进行处理, 由于`3`的 delay 比`2`短，`3`的结果先返回，`3`比`2`的回调函数先进入任务队列

再执行到`4`, 是同步任务，立即执行，于是答案变成了`a d`

现在主线程的执行栈为空，任务队列中的任务依次入执行栈执行

所以最终答案为`a d c b`

这样就结束了？？当然不是👻👻

*   宏任务（macrotask）和微任务（microtask）

上述的异步任务只是一个宏观的概念，若对异步任务进行细分的话，又可以分为宏任务和微任务，他们之间的执行顺序又是不同的。

在异步任务回调函数进入任务队列前会对这个异步任务进行判断看他是宏任务还是微任务

宏任务进入宏任务队列，微任务进入微任务队列

在同步任务执行完成后，会先执行微任务队列的任务，直到微任务队列为空，再执行宏任务队列中的任务

循环往复，完成！！

常见的宏任务

```
整体代码script
I/O
setTimeout
setInterval
requestAnimationFrame
复制代码

```

常见的微任务

```
MutationObserver
Promise的回调
复制代码

```

再举个栗子🌰

```
1 console.log('a');

2 setTimeout(function () {
  console.log('b');
}, 0);

3 Promise.resolve()
  .then(function () {
    console.log('c');
  })
  .then(function () {
    console.log('d');
  });

4 console.log('e');
复制代码

```

执行的结果是什么呢？

`a e c d b`

任务进行执行栈，遇到`1`, 是同步任务，立即执行 于是答案中就有了`a` ，

再执行到`2`, 是异步宏任务，进入异步处理模块进行处理, 他的回调函数进入宏任务队列

再执行到`3`, 是异步微任务，进入异步处理模块进行处理，他的回调函数进入微任务队列

再执行到`4`，是同步任务，立即执行，答案变成`a e`

现在主线程的执行栈为空，先去查看微任务队列，`3`的回调函数进入执行栈执行，答案变成`a e c`, 这个函数返回'undefined', 触发下一个回调函数，由于它又是异步微任务，进入异步处理模块进行处理，他的回调函数进入微任务队列

现在主线程中的执行栈又为空了，又去查看微任务队列，`3`的回调的回调进入执行栈执行后出栈，答案为`a e c d`

主线程中的执行栈又为空了, 重复上面的步骤，微任务队列也为空，查看宏任务队列，执行宏任务队列中的任务

最后答案为`a e c d b`

完成 ✅

*   思考一下🤔，下面的代码执行顺序又是怎样的呢

```
console.log(0);

setTimeout(function () {
    console.log(1);
});

new Promise(function(resolve,reject){
    console.log(2)
    resolve(3)
}).then(function(val){
    console.log(val);
})

console.log(4);
复制代码

```

答案是

```
                                                                                                  0 2 4 3 1
复制代码

```

Node 下的事件循环机制
=============

process.nextTick 和 setImmediate
-------------------------------

Node 也是单线程的运行环境，仅从`api`的角度来讲，相对于浏览器的 event loop， 多了一个微任务的`process.nextTick`以及宏任务的`setImmediate`,

*   process.nextTick

process.nextTick 的回调和 Promise 的回调都是微任务，但是 process.nextTick 的回调会比 Promise 的回调先执行 对于以下代码

```
process.nextTick(() => console.log(3));
Promise.resolve().then(() => console.log(4));
(() => console.log(5))();
复制代码

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6be3ce139d94440e8a388b5342319e5c~tplv-k3u1fbpfcp-watermark.image) 交换位置后，得出来同样的结果

```
process.nextTick(() => console.log(3));
Promise.resolve().then(() => console.log(4));
(() => console.log(5))();
复制代码

```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5310098b89c47ccb59f743f5602c9fc~tplv-k3u1fbpfcp-watermark.image)

所以，如果我们想要一个异步任务能够尽快的执行，就可以使用`process.nextTick`

*   setImmediate `setImmediate`是宏任务，从官方的定义来讲，`setImmediate`会在一次 Event loop 中立即执行，但从运行上来讲，得到的结果确是不一定的 (原因在后面)，比如以下代码：

```
setTimeout(() => console.log(1));
setImmediate(() => console.log(2));
复制代码

```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f75187a08a65469199f6d5ca6c6f96be~tplv-k3u1fbpfcp-watermark.image)

事件循环的阶段
-------

*   事件循环解析

```
   ┌───────────────────────────┐
┌─>│           timers          │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │
   └───────────────────────────┘

复制代码

```

*   阶段概述

`timers`: 执行`setTimeout()`和`setInterval()` 的调度 (看是否满足 delay 的要求) 的回调函数，不满足将直接离开这个阶段

`I/O callbacks`: 执行延迟到下一个循环迭代的 I/O 回调 (也就是`除了`以下操作以外的回调)

```
setTimeout()和setInterval()的回调函数
setImmediate()的回调函数
用于关闭请求的回调函数，比如socket.on('close', ...)
复制代码

```

`idle, prepare`: 仅 libuv 系统内部使用

`Poll`: 这个阶段是轮询阶段

```
在poll队列不为空的时候，会检索并执行新的I/O回调事件
如果为空：
  若调用了setImmediate()， 就结束poll阶段，直接进入check阶段,
  如果没有调用setImmediate()，就会等待，等待新的回调I/O事件的到来，然后立即执行
  如果脚本没有调用了setImmediate()，并且poll队列为空的时候，事件循环将检查哪些计时器 timer 已经到时间。 如果一个或多个计时器 timer 准备就绪，则事件循环将返回到计时器阶段，以执行这些计时器的回调，这也相当于开启了新一次的循环（tick）

复制代码

```

`check`: 在这个阶段执行 setImmediate() 的回调函数

`close callbacks`: 执行关闭请求的回调函数，比如 socket.on('close', ...)

*   setTimeout 和 setImmediate

回到之前的问题，为什么这两句代码的执行顺序是不一样的呢？

```
setTimeout(() => console.log(1)，0);
setImmediate(() => console.log(2));
复制代码

```

照理来说，setTimeout 在 timers 阶段，并且它回调执行的 delay 参数是 0，而 setImmediate 在 check 阶段，但是 nodejs 官网关于 setTimeout 的定义有这样一句话

`When delay is larger than 2147483647 or less than 1, the delay will be set to 1. Non-integer delays are truncated to an integer.`

也就是说，下面这两个表达式是`等价`的

`setTimeout(() => console.log(1)，0); === setTimeout(() => console.log(1)，1);`

而实际执行的时候，进入事件循环以后，有可能到了`1ms`，也有可能还没到，这取决于系统当时的状况。如果没到 1ms，就会跳过`timers`，向下执行，到了`check`，先执行`setImmediate`，然后再在下一次循环中执行`setTimeout`。

但是对于下面的代码，一定会先打印`2`，再打印`1`

```
const fs = require('fs');

fs.readFile('test.js', () => {
  setTimeout(() => console.log(1));
  setImmediate(() => console.log(2));
});
复制代码

```

他的执行过程是会先跳过 timers 阶段，回调直接进入`I/0 callback`，然后向下执行，到了`check`阶段执行`setImmediate`，然后才在下一次循环的 timers 执行`setTimeout`。

参考链接
====

[维基百科中的调用栈](https://en.wikipedia.org/wiki/Call_stack)

[Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)

[JavaScript 运行机制详解：再谈 Event Loop](https://www.ruanyifeng.com/blog/2014/10/event-loop.html)

[How JavaScript Works: An Overview of JavaScript Engine, Heap, and Call Stack](https://dev.to/bipinrajbhar/how-javascript-works-under-the-hood-an-overview-of-javascript-engine-heap-and-call-stack-1j5o)

[Node 定时器详解](http://www.ruanyifeng.com/blog/2018/02/node-event-loop.html)

[node 官网关于 event loop 的机制](https://nodejs.org/zh-cn/docs/guides/event-loop-timers-and-nexttick/)

[node 关于 setTimeout 的官方文档](https://nodejs.org/api/timers.html#timers_settimeout_callback_delay_args)