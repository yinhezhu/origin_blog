---
title: ECMA-运行机制之Event_Loop
date: 2018-05-30 15:37:39
tags: [ECMA,Event-Loop]
category: "ECMA"
---
## 前言
距离这篇文章完笔虽然才两个月，但是我已经对各种细节忘记得差不多（不常用的东西马上就忘记了，大脑内存不足会经常自动腾出空间记忆别的事情），各位如果有任何疑问我大概率是回答不上来，非常抱歉。另外我觉得深入折腾这种东西意义其实不是太大，还不如学习一下更加通用价值更加高的知识（例如算法、数据库原理、操作系统原理、tcp/ip协议、架构设计、高数线代概率统计等）
## 不同的event loop
event loop是一个执行模型，在不同的地方有不同的实现。浏览器和nodejs基于不同的技术实现了各自的event loop。网上关于它的介绍多如牛毛，但大多数是基于浏览器的，真正讲nodejs的event loop的并没有多少，甚至很多将浏览器和nodejs的event loop等同起来的。  我觉得讨论event loop要做到以下两点：

- 首先要确定好上下文，nodejs和浏览器的event loop是两个有明确区分的事物，不能混为一谈。
- 其次，讨论一些js异步代码的执行顺序时候，要基于node的源码而不是自己的臆想。

简单来讲，

- nodejs的event是基于libuv，而浏览器的event loop则在[html5的规范](https://www.w3.org/TR/html5/webappapis.html#event-loops)中明确定义。
- libuv已经对event loop作出了实现，而html5规范中只是定义了浏览器中event loop的模型，具体实现留给了浏览器厂商。

## nodejs中的event loop
关于nodejs中的event loop有两个地方可以参考，一个是nodejs[官方的文档](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)；另一个是libuv的[官方的文档](http://docs.libuv.org/en/v1.x/design.html)，前者已经对nodejs有一个比较完整的描述，而后者则有更多细节的描述。nodejs正在快速发展，源码变化很大，以下的讨论都是基于nodejs9.5.0。

（然而nodejs的event loop似乎比预料更加复杂，在查看nodejs源码的过程中我惊奇发现原来nodejs的event loop的某些阶段，还会将v8的micro task queue中的任务取出来运行，看来nodejs的浏览器的event loop还是存在一些关联，这些细节我们往后再讨论，目前先关注重点内容。）
### event loop的6个阶段（phase）
nodejs的event loop分为6个阶段，每个阶段的作用如下（process.nextTick()在6个阶段结束的时候都会执行，文章后半部分会详细分析process.nextTick()的回调是怎么引进event loop，仅仅从uv_run()是找不到process.nextTick()是如何牵涉进来）：

- timers：执行setTimeout() 和 setInterval()中到期的callback。
- I/O callbacks：上一轮循环中有少数的I/Ocallback会被延迟到这一轮的这一阶段执行
- idle, prepare：仅内部使用
- poll：最为重要的阶段，执行I/O callback，在适当的条件下会阻塞在这个阶段
- check：执行setImmediate的callback
- close callbacks：执行close事件的callback，例如socket.on("close",func)

```bash
   ┌───────────────────────┐
┌─>│        timers         │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     I/O callbacks     │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     idle, prepare     │
│  └──────────┬────────────┘      ┌───────────────┐
│  ┌──────────┴────────────┐      │   incoming:   │
│  │         poll          │<─────┤  connections, │
│  └──────────┬────────────┘      │   data, etc.  │
│  ┌──────────┴────────────┐      └───────────────┘
│  │        check          │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
└──┤    close callbacks    │
   └───────────────────────┘
```

event loop的每一次循环都需要依次经过上述的阶段。  每个阶段都有自己的callback队列，每当进入某个阶段，都会从所属的队列中取出callback来执行，当队列为空或者被执行callback的数量达到系统的最大数量时，进入下一阶段。这六个阶段都执行完毕称为一轮循环