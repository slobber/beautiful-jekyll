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

这种功能还是很常见的，比如我们在其他模块中有投票模块、幻灯片模块，我们需要在 UEditor 里插入对应的内容，HTML 里写个 `[poll id]` 或者 `[slide id]`，然后渲染的时候通过正则把相应的 id 取出来再添加对应 render 出来的代码。如何写插件实现
