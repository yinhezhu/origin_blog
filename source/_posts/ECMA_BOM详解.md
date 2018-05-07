---
title: BOM详解
date: 2018-04-17 15:38:04
tags: [ECMA,BOM]
category: "ECMA"
---
### 概念
``BOM`` 主要处理浏览器窗口和框架，不过通常浏览器特定的JavaScript 扩展都被看做 ``BOM`` 的一部分。这些扩展包括：
- 弹出新的浏览器窗口
- 移动、关闭浏览器窗口以及调整窗口大小
- 提供 Web 浏览器详细信息的定位对象
- 提供用户屏幕分辨率详细信息的屏幕对象
- 对 cookie 的支持
- IE 扩展了 BOM，加入了 ActiveXObject 类，可以通过 JavaScript 实例化 ActiveX 对象

### 认识``BOM``
``BOM``的核心是``window``，而``window``对象又具有双重角色，它既是通过``js``访问浏览器窗口的一个接口，又是一个``Global``（全局）对象。这意味着在网页中定义的任何对象，变量和函数，都以``window``作为其``Global``对象。
### 一张图说明window对象
<img src="/static/img/window对象.svg" width="800" />