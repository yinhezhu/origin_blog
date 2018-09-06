---
title: Vue_vue数据监听原理
date: 2018-05-30 18:31:54
tags: [Vue,源码]
category: "Vue"
---
# vue整体框架和主要流程分析

之前对看过比较多关于vue源码的文章，但是对于整体框架和流程还是有些模糊，最后用chrome debug对vue的源码进行查看整理出这篇文章。。。。

本文对vue的整体框架和整体流程进行简要的分析，不对某些具体的细节进行分析，所有需要对vue有初步的认识，包括对Object.defineProperty、虚拟DOM有一定了解，本文不会对Object.defineProperty、虚拟DOM的原理和细节进行分析。
vue大体可以分两个部分：

1.采用Object.defineProperty进行数据的双向绑定；

2.采用虚拟DOM技术进行视图渲染；

## vue入口

<img src="/static/img/vue1.png" width="800" />

vue构造函数调用了this._init(options)方法，这个方法在initMixin中，如上图所示，进入initMixin
<img src="/static/img/vue2.png" width="800" />

initMixin主要完成数据的初始化和视图的初始化：

1.数据初始化主要是数据的observe，在上图的initState中进行；

2.视图的初始化在vm.$mount(vm.$options.el),其中vm为Vue的实例，watcher的设置也是在vm.$mount(vm.$options.el）中完成的；

我们可以看到这里定义了beforeCreated和created这两个钩子函数。

## 数据初始化

接着上面我们看看数据初始化都做了什么，进入initState
<img src="/static/img/vue3.png" width="800" />

这里我们主要对数据进行操作的是initData，传入的是vm，我们来具体看看initData：
<img src="/static/img/vue4.png" width="800" />

我们先忽略前面的一些逻辑判断，主要看两个地方：

1.数据代理，主要是将_data的数据代理到vm上，这样的话可以直接对vm上的数据进行修改;

2.数据observe，传入data；

我们先看看vue怎么对数据进行observe的，进入observe
<img src="/static/img/vue5.png" width="800" />

在observe里返回的是ob，也就是Observer类的实例，我们看看Observer类是怎么定义的，进入Observer类
<img src="/static/img/vue6.png" width="800" />

如上图在对data进行observe时对数组进行了特殊的处理，这块我们先不看，先看一般情况下的处理，即调用this.walk(value)
<img src="/static/img/vue7.png" width="800" />

walk主要对data的属性进行遍历，进入defineReactive
<img src="/static/img/vue8.png" width="800" />

可以看到Object.defineProperty是在这里对属性设置get和set的，其中get主要进行依赖收集，其实就是在收集视图渲染的watcher，后面会提到，set主要是数据更新时进行视图的更新

至此，数据的初始化就完成了，从上面的分析来看，数据的初始化主要的工作就是对数据进行observe。

## 视图挂载
接着上面，在vue入口那里，我们知道视图的挂载主要是调用了vm.$mount(vm.$options.el)

<img src="/static/img/vue9.png" width="800" />
如图，所以我们进入vm.$mount，看看里面都干了啥，在源码里面有两处地方涉及到$mount

<img src="/static/img/vue10.png" width="800" />

这是第一处，就是return mountComponent
<img src="/static/img/vue11.png" width="800" />

这是第二处

咱们看看第二处，里面做了一个处理，就是将template编译成render函数，在vue的教程里有render函数的使用，这里我们可以看出我们在组件里定义render函数会比定义template快，因为在定义template的组件挂载时多了一步将template编译成render函数；

第二处的return 还是调用了第一处，所以我们看看第一处调用的mountComponent方法，进入mountComponent

<img src="/static/img/vue13.png" width="800" />


这里我们可以看到定义了两个钩子beforeMount和mount，中间调用了watcher，我们看一下这里watcher的定义，这里标注的不太好，挡住了。。。我们看看watcher的这行代码：
```javascript
vm._watcher=new Watcher(vm,updateComponent,noop)
```

我们可以看到Watcher类主要传入了vm,updateComponent,noop三个参数，其中updateComponent的主要作用是将虚拟DOM转化为真实的DOM并进行挂载，具体的细节下面在讨论，我们下面看看Watcher类是怎么定义的，进入Watcher

<img src="/static/img/vue15.png" width="800" />

这里我们注意两个地方，一个是this.getter的定义，这里就是上面传进来的updateComponent，还有就是执行this.get()，我们进入这个get方法
<img src="/static/img/vue16.png" width="800" />

这里我们看到首先收集的依赖是当前watcher实例，然后调用getter方法也就是updateComponent方法，之前我们对updateComponent方法的作用进行了简单的说明，这里我们具体看看updateComponent都干了啥，进入updateComponent:
<img src="/static/img/vue17.png" width="800" />

这里调用了vm._update方法，其中传入的参数有vm._render()，_render函数主要的作用是产生虚拟DOM，进入_update
<img src="/static/img/vue18.png" width="800" />

这里主要是将虚拟DOM转化为真实DOM并进行挂载，分两种情况，分别是有旧的虚拟DOM和无旧的虚拟DOM，对应初始化时调用还是数据更新时调用,这里定义了一个钩子beforeUpdate

到这里，视图的初始化和挂载也结束了，下面看看数据变化时视图是如何更新的

## 数据变化时视图更新过程
接着上面我们看看数据变化时视图是怎么变化的，在数据初始化的时候，我们知道数据变化时将触发set方法，如下图：

<img src="/static/img/vue19.png" width="800" />

上图可以看出，set最后调用了dep.notify，进入notify
<img src="/static/img/vue20.png" width="800" />

如上图，notify主要将收集的依赖，也就是收集的所有watcher，调用所有watcher的update方法，我们看看watcher的updata方法干了啥

<img src="/static/img/vue21.png" width="800" />

这里就是调用了queueWatcher,进入queueWatcher

<img src="/static/img/vue22.png" width="800" />
这里采用队列异步更新，就是讲=将watcher push进队列queue中，然后执行nextTick方法，进入nextTick

<img src="/static/img/vue23.png" width="800" />


这个部分有点难看，cb为传入的flushSchedulerQueue函数，执行timerFunc，将nextTickHander加入异步队列，执行nextTickHander，执行cb，既执行flushSchedulerQueue，进入flushSchedulerQueue
<img src="/static/img/vue25.png" width="800" />

主要看watcher.run(),进入watcher.run
<img src="/static/img/vue27.png" width="800" />

执行了this.get()，即进入前面数据渲染和挂载的地方

到这里，vue整个的执行流程基本就结束了。

## vue流程图

盗用一下vue官网关于vue生命周期的图，对照之前的内容梳理一下：

<img src="/static/img/vue28.png" width="400" />











