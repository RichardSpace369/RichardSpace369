---
title: 理解html
author: richard
toc: true
excerpt: 理解html...
categories:
- html
---

## html

>HTML（HyperText Markup Language）即超文本标记语言，是一种用于创建网页的标记语言。HTML经历过多个版本，包括HTML 2.0、HTML 3.x、HTML 4.x以及最新的HTML 5。HTML源于SGML（Standard Generalized Markup Language，标准通用标记语言），遵循SGML指定的语法和规则，但从HTML 5开始，将不再基于S

### HTML与Xhtml

HTML的格式比较松散，这会导致一些问题，下面只列出了两个比较有代表性的问题。
1. 兼容性差，非常依赖浏览器的容错性，市面上的浏览器众多，各种未知的错误随时会发生。
2. 移植性差，将来如果将网页移植到对格式要求严格的设备中（例如盲文设备），就必须重构

为了解决上述所列的问题，W3C（World Wide Web Consortium，万维网联盟）在2000年发布了XHTML 1.0，XHTML是XML的一种应用，作为HTML的一个子集，完全兼容HTML，但格式更严谨。

### HTML文档的基本结构

HTML文档的基本结构包括四个HTML元素（如以下代码所示），分别是DOCTYPE、html、head和body，任何HTML文档都需要这四个元素

| 元素 | 说明 |
| :-----| :---- |
| DOCTYPE | 文档类型声明，有助于确定浏览器的渲染模式 | 
| html | 根元素 |
|head|包含文档的元数据|
|body|包含文档的内容|

## html5

HTML5不仅是HTML的最新版本，还是一系列Web技术的集合，包括CSS3、JavaScript、多媒体、缓存和无障碍访问等。HTML5的规范是由两个组织制定的，分别是WHATWG（网页超文本技术工作小组）和W3C

### html5新特性

HTML5包含很多新特性，下面重点列举其中的4个。
1. 语义化的Web，让人和计算机更容易理解。
2. 削弱对第三方插件的依赖，以往播放音频或视频需要借助 Flash，现在HTML5 已原生支持多媒体。
3. 更丰富的应用，涵盖各方面。包括以下几种：
    - 新增绘图元素canvas，可直接操纵图片、制作游戏和动态广告特效等。
    - 扩展JavaScript API，例如Web存储、地理定位、拖放、操纵历史浏览记录和读取文件等。
    - 创建离线Web程序，解决无网络时无法使用Web应用的情况。
    - 引入Web Workers规范，解决Web应用越来越复杂的计算。


## DOCTYPE
DOCTYPE（Document Type Declaration）用于声明文档类型和DTD（Document Type Definition）规范，确保不同浏览器以相同的方式解析文档，以及执行相同的渲染模式。DTD就是文档类型定义，一种标记符的语法规则，保证SGML和XML文档格式的合法性。

### 语法
DOCTYPE元素必须声明在HTML文档的第一行，位于html元素之前，不需要结束标签

### 常用声明
#### html5
HTML5的声明方式略有不同，因为不再基于SGML，所以不需要引用DTD，只需一个根元素即可
#### HTML 4.01
HTML 4.01中的DTD可分为3种，分别是严格（Strict），过渡（Transitional）和框架集（Frameset）。接下来会逐个介绍。注意，在每一类的DTD旁都给出了相应的示例。
1. 严格的DTD能包含所有的HTML元素和属性，但不包括已被弃用的元素（例如font、center等），也不包括框架相关的元素（例如frameset、frame等）
2. 过渡的DTD仅不包含框架相关的元素
3. 框架集的DTD包含所有HTML元素和属性。

#### XHTML 1.0

XHTML的DTD同样也分为3种，严格（Strict）、过渡（Transitional）和框架集（Frameset）。3种DTD包含的元素和属性与HTML4.01中的相同，但会多一点XML的验证规范

### 浏览器渲染模式

浏览器的渲染模式有3种，分别是怪异模式（Quirks mode）、接近标准模式（Almost standards mode）和标准模式（Standards mode）


#### 怪异模式
这个模式主要是为了兼容早期的浏览器。早期的网站并不会遵循完整的规范，随着浏览器支持越来越多的规范，在那些旧的浏览器中开发的页面，在显示时会被破坏。为了向后兼容，浏览器就发明了怪异模式，模拟旧浏览器的行为。一行错误或无效的DOCTYPE都会触发怪异模式

#### 接近标准模式

这个模式是由某些DOCTYPE触发，基本上就是标准模式，但有一些调整，例如计算表格单元格的尺寸遵循CSS2规范，可以消除单元格中图像底部的空隙。图像之所以有空隙是因为它要与文本的基线对齐，像g、j要正常显示尾巴，就需要有一些空隙。


## 语义化