---
layout: post
title: 你想 pick 哪款嵌入式 Markdown 编辑器?
date: 2020-09-01
categories: article
---

首先先声明下，我们今天要讲的并非客户端软件，而是“嵌入式”的 Markdown。

什么是嵌入式？嵌入式是计算机工程中的一个专业术语，即嵌入式系统。

> 嵌入式系统用于控制、监视或者辅助操作机器和设备的装置，是一种专用的计算机系统。

本文中提到的嵌入式，其实更偏向于这种解释：一个可插拔的组件。

插上它，某个功能就会实现。拔掉，又不会影响系统运行。

Markdown是一种轻量级标记语言，易读易写，对图片、图标、数学式都有支持。可用于十几种编程语言，目前许多平台和框架都支持Markdown，主要应用在各个笔记和博客平台。


> **Markdown的时间轴**
>
> 它诞生于2004年
>
> 2014年9月，被更名为 [CommonMark](https://commonmark.org/)
>
> 2012年，实施标准化
>
> 2017年，Github发布了基于CommonMark的[Github Flavored Markdown（GFM）](https://github.github.com/gfm/)的正式规范。
>
> 2018年，发布 CommonMark 规范和测试包
>
> ...


此时，你可能会cue到富文本编辑器，它拥有着类 Word 软件般的用户体验————所见即所得，但其致命的缺点是容易造成 XSS 漏洞。

跑偏了，今天的主角不是它『富文本』，而是它『Markdown』👈


接下来推荐几款小编接触到的可嵌入的Markdown编辑器。


## tui.editor

第一款是由韩国 nhn 官方维护 `tui.editor`。

[TOAST UI Editor](https://github.com/nhn/tui.editor) 是一款的基于 GFM(GitHub Flavored Markdown)的编辑器，拥有两个模式 Markdown 和 WYSIWYG（所见即所得），可在写作过程中任意切换。 用户体验也是相当的舒适，对于偏好富文本编辑器的用户来说，可谓是顺滑切换。


如果你读过 tui.editor 的文档，是否也和我一样觉得真香呢。

👉它的特性：
- 提供了3种框架组件
  - [@toast-ui/jquery-editor](https://github.com/nhn/tui.editor/tree/master/apps/jquery-editor)
  - [@toast-ui/react-editor](https://github.com/nhn/tui.editor/tree/master/apps/react-editor)
  - [@toast-ui/vue-editor](https://github.com/nhn/tui.editor/tree/master/apps/vue-editor)
- 提供了强大的 Markdown 语法的扩展插件
  - [chart](https://github.com/nhn/tui.editor/tree/master/plugins/chart) 图表
  - [code-syntax-highlight](https://github.com/nhn/tui.editor/tree/master/plugins/code-syntax-highlight) 代码块区域高亮
  - [color-syntax](https://github.com/nhn/tui.editor/tree/master/plugins/color-syntax) 文本颜色选择器
  - [table-merged-cell](https://github.com/nhn/tui.editor/tree/master/plugins/table-merged-cell) 表格合并单元格
  - [uml](https://github.com/nhn/tui.editor/tree/master/plugins/uml) UML（ 统一建模语言）图
- 提供了丰富的 [API](https://nhn.github.io/tui.editor/latest/)，方便使用者开发自己的扩展
- 在浏览器支持方面也是很喜人，支持IE 10+
- 支持Viewer模式（只读模式），用于编辑器内容的最终呈现（多用于详情页中）
- 支持[国际化](https://github.com/nhn/tui.editor/blob/master/apps/editor/docs/i18n.md#supported-languages)

介绍到这，你是否也心动了。喜欢它，就Pick它吧~

[在线demo](https://nhn.github.io/tui.editor/latest/tutorial-example01-editor-basic)

## SimpleMDE

[SimpleMDE](https://github.com/sparksuite/simplemde-markdown-editor) 是一个可嵌入的JavaScript Markdown编辑器，除了基础的字体加粗、倾斜，插入图片、链接等功能外，还支持自动保存和拼写检查。

小编研究了下，这个自动保存是基于 localStorage 存储，如果你用到这个功能了，切记尽量不存储base64格式的图片，不然会造成内存溢出。你懂得『毕竟localstorage只支持5M大小』。

在预防XSS攻击方面，作者并未对此做处理。从github的代码提交时间来看，目前已不再维护，深表遗憾。如果你有想法，可以fork 它，自己维护，小编表示很赞同呢。

这款编辑器虽未像 tui.editor一样，提供了 “所见即所得”模式，但它有贴心地提供了 [Markdown使用指南](https://simplemde.com/markdown-guide)，不管你的项目是基于 Vue 还是 React，你都可以Pick它，还是很nice的~

[在线demo](https://simplemde.com/)

------
以上两款，不管在哪个框架中都能使用。接下来介绍三款分别基jQuery、Vue和React的Markdown Editor组件

------
## Editor.md

[Editor.md](https://github.com/pandao/editor.md) 是一款国产可嵌入的开源 Markdown 在线编辑器组件。基于 CodeMirror、jQuery 和 Marked 构建。同时支持通用 Markdown / CommonMark 和 GFM (GitHub Flavored Markdown) 风格的语法，特性相当丰富。在这5款编辑器中，目前 Star 数也是仅次于 tui.editor。

它的特性也是很丰富，总有一个满足你的要求😍。

👉它的特性：
- 支持实时预览、图片（跨域）上传、预格式文本/代码/表格插入、代码折叠、搜索替换、只读模式、自定义样式主题和多语言语法高亮等功能；
- 支持 ToC 目录（Table of Contents）、Emoji 表情、Task lists、@链接等 Markdown 扩展语法；
- 支持 TeX 科学公式（基于 KaTeX）、流程图 Flowchart 和 时序图 Sequence Diagram;
- 支持识别和解析 HTML 标签，并且支持自定义过滤标签解析，具有可靠的安全性和几乎无限的扩展性；
- 支持 AMD / CMD 模块化加载（支持 Require.js & Sea.js），并且支持自定义扩展插件；
- 兼容主流的浏览器（IE8+）和 Zepto.js，且支持 iPad 等平板设备；
- 支持自定义主题样式；

虽说这款编辑器已经对基础的XSS攻击做了处理，但还是存在安全隐患，由于它目前处于维护态，可能维护者已经在修BUG的路上了。如果你有解决办法，可以给它提 [PR](https://github.com/pandao/editor.md/compare) 👈。

如果你对我所说的安全隐患有兴趣，可以试运行以下代码。

```html
<img src="https://pandao.github.io/editor.md/images/logos/editormd-logo-180x180.png" onload="javascript:alert(1)"/>
```

[在线demo](http://editor.md.ipandao.com/)

## @uiw/react-md-editor

[@uiw/react-md-editor](https://github.com/uiwjs/react-md-editor) 是一个由 React 和 TypeScript 实现的带有预览的 Markdown 编辑器。目前由 uiw 团队维护。

> uiw 是基于 React 16+ 的 UI 组件库。

👉它的特性：
- 三种模式任意切换：编辑、预览、编辑 + 实时预览
- 支持Viewer模式（只读模式），用于编辑器内容的最终呈现。最常见于文章详情页的展示
- 支持 KaTeX 科学公式预览
- 包体积很小，才 439 kB

因为它是一个 React 组件，所以用法也很简单。直接引入，加上配置就哦了。

```js
import React from "react";
import ReactDOM from "react-dom";
import MDEditor from '@uiw/react-md-editor';

export default function App() {
  const [value, setValue] = React.useState("**Hello world!!!**");
  return (
    <div className="container">
      <MDEditor
        value={value}
        onChange={setValue}
      />
      <MDEditor.Markdown source={value} />
    </div>
  );
}
```
[在线demo](https://uiwjs.github.io/react-md-editor/)

## mavon-editor
[mavonEditor](https://github.com/hinesboy/mavonEditor)是一个基于 Vue 的 Markdown 编辑器。目前它是由个人维护，实现了部分国际化，支持8种语言。看过之前几款后，再来看它，有种眼前一亮的感觉，因为它的特性很新颖。可以在vue中使用，也可以在 nuxt.js 中使用。

👉它的特性：
- 支持上下角标
- 支持文字高亮（标记笔）
- 支持下划线
- 支持布局：居左、居中、居右
- 支持 KATEX公式 预览
- 支持标题导航
- 支持国际化
- 支持快捷键设置
- 支持撤回、恢复、清空等等

[在线demo](http://106.15.232.22/)


## 一览

| editor | 关注数 | 更新情况 | 存在xss风险级别 | 框架支持情况 | npm包 |
| --- | --- | ---- | --------- | ------ | ---- |
| tui.editor | 11k+ | 一个月前 | 低 | 支持jQuery, Vue, React | [@toast-ui/editor](https://www.npmjs.com/package/@toast-ui/editor) |
| SimpleMDE | 8k | 4年前 | 高 | 支持jQuery, Vue, React | [simplemde](https://www.npmjs.com/package/simplemde) |
| editor.md | 10k+ | 16个月前 | 较低 | 支持jQuery | [editor.md](https://www.npmjs.com/package/editor.md) |
| @uiw/react-md-editor | 134 | 几天前 | 较低 | 支持React | [@uiw/react-md-editor](https://www.npmjs.com/package/@uiw/react-md-editor) |
| mavonEditor | 4.2k | 4个月前 | 较低 | 支持Vue | [mavon-editor](https://www.npmjs.com/package/mavon-editor) |

以上是这几个嵌入式 Markdown 编辑器的一览。除了小编介绍的这几款，肯定还有很多其他优秀的 Markdown 编辑器，如果你知道也可推荐给我。

推荐了这么多款，总有一款适合你，快点来Pick吧！！！

