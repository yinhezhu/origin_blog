---
title: Vue_vue源码分析_原型&全局API
date: 2018-05-10 16:32:04
tags: [Vue,源码]
category: "Vue"
---

# 前端架构之路，好胆你就来。

看过很多关于`Vue`源码的文章，觉得自己技术栈很重要的一环就是`Vue`，所以就想自己也写一篇关于`Vue`源码的文章。
这是我关于`Vue`源码分析的第一篇文章，主要是讲的是构造函数`Vue`原型上的方法，全局API。并不涉及编译过程、数据绑定、路由实现、store数据仓库等每个细节的具体实现，具体实现细节后边的文章中会陆续涉及。
本文章讲解的源码是基于`Vue 2.5.13`的。因为自己的业务线需要使用`Vue.compiler`，并且我做的项目是web客户端渲染，所以这里只讲解`with-compiler`的版本。

了解一项工程首先要从目录结构以及入口文件开始了解。就像你了解一个姑娘，应该以她为中心辐射她的社交圈一样。

## 入口文件
`Vue`的源码是一个标准的`npm`工程目录结构，目录结构如下
```bash
├── dist ---------------------------------- 构建后文件的输出目录
├── examples ------------------------------ 存放一些使用Vue开发的应用案例
├── flow ---------------------------------- 类型声明，使用开源项目 [Flow](https://flowtype.org/)
├── test ---------------------------------- 包含所有测试文件
├── scripts ------------------------------- 构建相关的文件，一般情况下我们不需要动
├── src ----------------------------------- 这个是我们最应该关注的目录，包含了源码
│   ├── compiler -------------------------- 编译器代码的存放目录，将 template 编译为 render 函数
│   │   ├── codegen ----------------------- 存放从抽象语法树(AST)生成render函数的代码
│   │   ├── directives -------------------- 存放处理指令的相关代码
│   │   ├── parser ------------------------ 存放将模板字符串转换成元素抽象语法树的代码
│   │   ├── optimizer.js ------------------ 分析静态树，优化vdom渲染
│   ├── core ------------------------------ 存放通用的，平台无关的代码
│   │   ├── components -------------------- 包含抽象出来的通用组件
│   │   ├── global-api -------------------- 包含给Vue构造函数挂载全局方法(静态方法)或属性的代码
│   │   ├── instance ---------------------- 包含Vue构造函数设计相关的代码
│   │   ├── observer ---------------------- 反应系统，包含数据观测的核心代码
│   │   ├── util -------------------------- 包含核心代码的一些常用工具和配置【策略打表】
│   │   ├── vdom -------------------------- 包含虚拟DOM创建(creation)和打补丁(patching)的代码
│   ├── platforms ------------------------- 包含平台特有的相关代码以及不同的构建的或包的入口文件
│   │   ├── entry-compiler.js ------------- vue-template-compiler 包的入口文件
│   │   ├── entry-runtime.js -------------- 运行时构建的入口，输出 dist/vue.common.js 文件，不包含模板(template)到render函数的编译器，所以不支持 `template` 选项，我们使用vue默认导出的就是这个运行时的版本。大家使用的时候要注意
│   │   ├── entry-runtime-with-compiler.js  独立构建版本的入口，输出 dist/vue.js，它包含模板(template)到render函数的编译器
│   │   ├── entry-server-renderer.js ------ vue-server-renderer 包的入口文件
│   ├── server ---------------------------- 包含服务端渲染(server-side rendering)的相关代码
│   ├── sfc ------------------------------- 包含单文件组件(.vue文件)的解析逻辑，用于vue-template-compiler包
│   ├── shared ---------------------------- 包含整个代码库通用的代码
├── package.json -------------------------- 不解释
```

### entry-runtime-with-compiler.js
我们看到独立构建版本的入口，是`entry-runtime-with-compiler.js`,所以我们从这个文件入手。
```javascript
// 引入Vue
import Vue from './runtime/index'
import { compileToFunctions } from './compiler/index'

// 缓存原有原型上的方法$mount
const mount = Vue.prototype.$mount

// 替换原型方法$mount
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {

}

// 挂在全局方法compile
Vue.compile = compileToFunctions

export default Vue
```
- 1、我们看到这个入口文件重载了原型上的`$mount`方法
- 2、在Vue上挂载了全局方法compile
- 3、`Vue`是从`/src/platforms/web/runtime/index.js`引入的，我们查看这个文件

### /src/platforms/web/runtime/index.js
```javascript
import Vue from 'core/index'

Vue.config.mustUseProp = mustUseProp
Vue.config.isReservedTag = isReservedTag
Vue.config.isReservedAttr = isReservedAttr
Vue.config.getTagNamespace = getTagNamespace
Vue.config.isUnknownElement = isUnknownElement

extend(Vue.options.directives, platformDirectives)
extend(Vue.options.components, platformComponents)

// 安装原型方法__patch__ [带下划线代表内部使用]
Vue.prototype.__patch__ = inBrowser ? patch : noop

// 安装原型上的$mount方法
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}

export default Vue
```
- 1、我们看到这个文件主要是安装了原型方法`$mount`
- 2、设置了全局属性`Vue.config`
- 3、`Vue`是从`/src/core/index.js`引入的，我们查看这个文件

### /src/core/index.js
```javascript
import Vue from './instance/index'
import { initGlobalAPI } from './global-api/index'
import { isServerRendering } from 'core/util/env'
import { FunctionalRenderContext } from 'core/vdom/create-functional-component'

// 挂在全局API
initGlobalAPI(Vue)

Object.defineProperty(Vue.prototype, '$isServer', {
  get: isServerRendering
})

Object.defineProperty(Vue.prototype, '$ssrContext', {
  get () {
    /* istanbul ignore next */
    return this.$vnode && this.$vnode.ssrContext
  }
})

// expose FunctionalRenderContext for ssr runtime helper installation
Object.defineProperty(Vue, 'FunctionalRenderContext', {
  value: FunctionalRenderContext
})

Vue.version = '__VERSION__'

export default Vue
```
- 1、执行了`initGlobalAPI(Vue)`，就字面意思而言，应该是初始化了一些全局的API，后边用图示讲解
- 2、添加原型属性`$isServer`,`$ssrContext`
- 3、添加全局属性`FunctionalRenderContext`
- 4、`Vue`是从`/src/core/instance/index.js`引入的，我们查看这个文件

### /src/core/instance/index.js
```javascript
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

// 安装_init
initMixin(Vue)

// 安装$set $delete $watch $data $props
stateMixin(Vue)

// 安装$on $once $off $emit
eventsMixin(Vue)

// 安装_update $forceUpdate $destroy
lifecycleMixin(Vue)

// 安装$nextTick _render 和一堆render相关方法
renderMixin(Vue)

export default Vue
```
- 我们终于看到构造函数`Vue`的庐山真面目了，众里寻他千百度，蓦然回首。
- 除了声明了`Vue`构造函数，这里还分别调用了
    - `initMixin(Vue);`
    - `stateMixin(Vue);`
    - `eventsMixin(Vue);`
    - `lifecycleMixin(Vue);`
    - `renderMixin(Vue)`
- 他们的作用是向`Vue`原型上安装方法。具体安装哪些后边用图示说明
- 值得注意的是，在`renderMixin(Vue)`中还安装了好几个的原型方法，用于渲染VNode相关操作。

> 至此，`Vue`的构造函数创建过程就完成了，用一张图来表示整个`Vue`的原型方法，全局API的安装过程

<img src="/static/img/Vue.svg" width="880" />

## 经过这一系列的骚操作，Vue就是这个样子了

<img src="/static/img/Vue123.svg" width="880" />

## 总结
如果你也想写一个大的框架的话，在最新的标准`es2015`下，你可以借鉴`Vue`的写法，分层次给构造函数添加原型方法以及全局API。利用策略模式分离可变和不变逻辑。