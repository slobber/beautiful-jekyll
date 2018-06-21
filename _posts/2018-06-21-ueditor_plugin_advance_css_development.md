---
layout: post
title: UEditor 插件开发中的 CSS 高级应用
image: /assets/python/pythonnet.logo.png
category: [ frontend ]
---

## 背景

UEditor 是百度出品的一个即见即所得的 HTML 可视化编辑器，在国内得到了广泛应用。但是，由于出现的较早，架构已经比较陈旧，没有使用完整的 HTML5、CSS3 特性，有很多实现从现在看来可能直接一两句代码就可以完成，但在 UEditor 中却都是通过 Javascript 从头到尾自己写出来的。当然，牵一发而动全身，很难重构 UEditor，但是在对 UEditor 的二次开发过程中，我们可以通过新的技术简化自己的插件实现，利用 CSS 技术实现原来需要复杂 Javascript 才能解决的问题。

在 UEditor 当中有一些功能是插入特定的 HTML 标签，在源代码视图下看到的是 HTML 代码或特定的标识文本，而设计视图下是真实的 HTML 元素，如图片，表格、分割符等等。其内部为了创建结点，都是在每个插件中写了相应的转换代码。比如：有一个分割页面（PageBreak）的[工具栏按钮](https://github.com/fex-team/ueditor/blob/master/_src/plugins/pagebreak.js)，具体实现就不分析。总之，我们的目标是利用 CSS 一些高级 feature 快速实现一个带标记的 HTML 代码的可视化，保证用户体验。

## 需求

需要制作一个 Dialog 插件，用户选择一个列表中的条目，在 UEditor 中插入一个带着 id 信息的 HTML 标签，设计视图中需要和其他内容有明显不同并显示出 id 信息，同时删除该元素时要完整删除，不可以只删一部分。

这种功能还是很常见的，比如我们在其他模块中有投票模块、幻灯片模块，我们需要在 UEditor 里插入对应的内容，HTML 里写个 `[poll id]` 或者 `[slide id]`，然后渲染的时候通过正则把相应的 id 取出来再添加对应 render 出来的代码。

如何写插件实现插入 HTML 不是本文的关注点，具体 js 参看 demo 就好了。本文重点是研究如何通过 CSS 实现插入的 HTML 在设计视图中独立显示，并能够正确的响应删除操作。

## 解决

假设我们插入的代码是这样的：

```
<p>[poll 12]</p>
```

这块内容在设计视图中是可以**一个字符一个字符的删除的**，同时和其他的段落内容样式上无差别。我们首先要解决的就是第一个问题。

对于`contenteditable` 之后的标签文字内容都是一个字符一个字符的删除啊，这种内在逻辑我们改不了。但是注意到图片的话，是可以整体删除的，按一下退格键，整个图片就没了（好一句废话）。所以，我们要让这个文本能作为一个整体，在设计视图中让光标无法定位到 `[poll 12]` 中间。最简单的方法是什么？就是让 p 标签中没有文字，让你没的可选，如果只是`<p></p>` 不就没内容可删就该删标签了嘛，好有道理！但是一个空标签怎么携带相关的条目信息给最终文章页渲染呢？同时又怎么在设计视图区分不同的条目呢？比如有十个投票。所以我们要用这样的一套样式：

```css
p.poll {
    display: block;
    position: relative;
    visibility: hidden;   /* 此方法可保证占位，但内容不显示 */
    padding: 5px;
    height: 21px;
}
p.poll:after {
    position: absolute;
    left:0px;
    right: 0px;
    top: 0px;
    bottom: 0px;
    content:"======= 请勿修改此投票域 [[" attr(style) "]] ======";
    text-align: center;
    color:#cacaca;
    z-index: 0;           /* 最风骚的设置，
    border:1px solid #ccc;
    background: #efefef;
    padding: 5px;
    visibility: visible;
    height: 21px;
}
```
