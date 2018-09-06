---
title: 综合-文档模式《!DOCTYPE》
date: 2018-04-17 14:48:59
tags: [基础知识,DOCTYPE]
category: "综合"
---
## 文档模式

### 严格模式与混杂模式

#### 不同文档模式的由来

相信了解过浏览器或浏览器厂商发展历史的同学都知道，在浏览器的发展初期，是没有什么标准可言的，各大浏览器厂商各自实现一套解析文档的方式，这对于开发者来说就如同灾难一样，开发者需要针对不同厂商的浏览器做兼容处理。慢慢的人们开始注意到标准的重要性，并成立了规范组织制定相应的标准，各个浏览器厂商也开始向标准靠拢，但随之而来的问题就是，如果一味的向标准靠拢，那必然会导致一个问题：老旧的网站将无法正常显示。为了做到向后兼容，浏览器厂商就保留了原有的文档解析方式，也就是现在所说的 `混杂模式`，同时将向标准靠拢的解析方式称为 `标准模式`，又称 `严格模式`。

也就是说，两种模式所代表的是浏览器解析文档的方式。而至于如果开启这两种模式，可以使用 `<!DOCTYPE>` 标签。

### 文档类型

#### <!DOCTYPE>

`<!DOCTYPE>` 用来声明文档类型，目的告诉浏览器使用哪种模式去解析文档。说白了就是告诉浏览器在解析文档的时候是采用 `混杂模式` 还是 `标准模式`。

###### 开启 混杂模式(quirks mode)

如果浏览器发现在文档开始处没有 `文档类型声明`，即没有 `<!DOCTYPE>` 标签，那么浏览器默认会使用混杂模式解析文档，当然不写 `<!DOCTYPE>` 标签是极其不推荐的方式。

###### 开启 标准模式(standards mode)

```html
<!-- HTML 4.01 严格型 -->
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"  "http://www.w3.org/TR/html4/strict.dtd">

<!-- XHTML 1.0 严格型 -->
<!DOCTYPE html PUBLIC  "-//W3C//DTD XHTML 1.0 Strict//EN"  "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">

<!-- HTML 5 -->
<!doctype html>
```

以上三种的任意一种都可以触发标准模式

### doctype详解
1. `<!DOCTYPE>`声明必须处于HTML文档的头部，在`<html>`标签之前，`HTML5`中不区分大小写
2. `<!DOCTYPE>`声明不是一个HTML标签，是一个用于告诉浏览器当前HTMl版本的指令
3. 现代浏览器的html布局引擎通过检查doctype决定使用兼容模式还是标准模式对文档进行渲染，一些浏览器有一个接近标准模型。
4. 在HTML4.01中`<!DOCTYPE>`声明指向一个DTD，由于HTML4.01基于SGML，所以DTD指定了标记规则以保证浏览器正确渲染内容
5. HTML5不基于SGML，所以不用指定DTD

### 常见dotype
1. HTML4.01 strict：不允许使用表现性、废弃元素（如font）以及frameset。声明：
```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
```
2. HTML4.01 Transitional:允许使用表现性、废弃元素（如font），不允许使用frameset。声明：
```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
```
3. HTML4.01 Frameset:允许表现性元素，废气元素以及frameset。声明：
```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Frameset//EN" "http://www.w3.org/TR/html4/frameset.dtd">
```
4. XHTML1.0 Strict:不使用允许表现性、废弃元素以及frameset。文档必须是结构良好的XML文档。声明：
```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
```
5. XHTML1.0 Transitional:允许使用表现性、废弃元素，不允许frameset，文档必须是结构良好的XMl文档。声明：
```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
```
6. XHTML 1.0 Frameset:允许使用表现性、废弃元素以及frameset，文档必须是结构良好的XML文档。声明：
```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Frameset//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-frameset.dtd">
```
7. HTML 5:
```html
<!doctype html>
```