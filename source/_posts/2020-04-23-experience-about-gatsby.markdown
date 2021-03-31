---
layout: post
title: Gatsby 踩过的那些坑
date: 2020-04-23
categories: 前端 
tag: Gatsby
coauthor: LemonGirl
---

开始之前我们先来了解一下 Gatsby.js 是什么。

官网的介绍是：

> Gatsby is a free and open source framework based on React that helps developers build blazing fast websites and apps.

即 Gatsby 是一个基于 React 的开源框架，基于 Gatsby 开发者能够轻易搭建超快站点。

最近团队的小伙伴基于 Gatsby 开发了几个项目，其优势在于它结合了最新的技术，使开发变得简易快速。但在其便捷的背后不乏踩过的各种坑，今天我们就来分享一些经验，说说那些踩过的坑。

### 1、CSS 作用域

Gatsby 的 CSS 默认是全局样式，在编写不同页面或者组件的时候需要保证 className 不重复，否则会导致样式覆盖或错乱。

解决办法：
<!--more-->
#### CSS Module 方案

使用 CSS Module 生成只作用于当前模块的 CSS，避免造成样式的全局污染。需要注意的是模块化的 CSS 文件命名需以`.module.css` 为后缀，内容只需要按照普通 CSS 文件来编写即可：

```css moduleA.module.css
.container {
  background: red;
  margin: 0 auto;
}
```

```jsx moduleA.js
import React from "react";
import moduleAStyle from "../styles/moduleA.module.css";

export default ({ children }) => (
  <div className={moduleAStyle.container}>{children}</div>
);
```

其中需要注意 className 的命名，`-`符号（无论是连续多少个`-`符号）编译后会自动转为驼峰式命名，以下 moduleB 页面的 h1 标题只会渲染`title--default-1`定义的样式:

```css moduleB.module.css
.title-default--1 {
  color: blueviolet;
  font-size: 12px;
}
.title--default-1 {
  color: brown;
}
```

```jsx moduleB.js
import React from "react";
import moduleBStyle from "../styles/moduleB.module.css";

const classNameToCamel () => (
  <h1 className={moduleBStyle.titleDefault1}>ClassNameToCamel</h1>
);
export default classNameToCamel
```

当然，你也可以使用 Sass，对应的文件后缀为`.module.scss`，记得引入对应的插件：

```js gatsby-config.js
plugins: [
  //...
  `gatsby-plugin-sass`,
];
```

#### TIPS

- 当一个组件在不同场景下使用需要加载不同样式时，样式模块可以作为变量从父组件传入

#### CSS in JS 方案

CSS in JS 是在 JS 文件中，使用内联的方式编写样式，同样能达到 CSS 模块化的效果。其原理是在编写了内联样式的元素上，自动添加一个唯一的类名，然后生成一个 CSS 文件，将对应的类名和样式放入其中。Gatsby 官网中推荐了两个库：Emotion 和 Styled Component。

以 Emotion 为例，首先引入插件：

```js gatsby-config.js
plugins: [
  //...
  `gatsby-plugin-emotion`,
];
```

使用：

```jsx
// styled写法
import styled from "@emotion/styled";

// styled 返回的是一个组件
const Title = styled.h1`
  display: inline；;
`;
export default () => <Title>这是一个h1标题</Title>;
```

```jsx
// css写法
import { css } from "@emotion/core";

// css 返回的是样式对象
const inline = css`
  display: inline；;
`;
export default () => <h1 css={inline}>这是标题</h1>;

// 或者使用内联方式
export default () => (
  <h1
    css={css`
      display: inline；;
    `}
  >
    这是标题
  </h1>
);
```

#### styled-jsx 方案

styled-jsx 是一个支持 jsx 方式编写的 css-in-js 插件，引入插件后便可直接使用：

```js gatsby-config.js
plugins: [
  //...
  `styled-jsx`,
];
```

```jsx
export default () => (
  <div>
    <h1 className="title">The title is red</h1>
    <style jsx>{`
      .title {
        color: red;
      }
    `}</style>
  </div>
);
```

### 2、移动端 click 事件存在 300ms 延迟

这个问题在移动端是普遍存在的（不限于 Gatsby 项目）。最早出现在 ios safari 上，为了判断用户的操作是单击还是双击，浏览器在 click 后等待 300ms 以判断用户是否会再次点击，此后 Android 也实现了这种机制。

解决办法：

#### 用 touchend 事件代替

移动端触摸事件的响应顺序为 touchstart --> touchmove --> touchend --> click，所以可以绑定 touchstart 或者 touchend 事件来代替 click 事件，以 达到加快事件响应的效果，从而解决 300ms 延迟的问题。

#### 引入开源模块

- fastclick 模块：在检测到 touchend 事件的时候，会通过 DOM 自定义事件模拟一个 click 事件并立即触发执行，并把浏览器在 300ms 之后真正的 click 事件阻止掉。
- zepto 的 touch 模块：模块里定义了一个 tap 事件，通过绑定 tap 事件可以实现 click 立即触发的功能。使用这个模块需要注意“点透”问题。

### 3、注意代码执行环境

在代码中，需要谨慎使用 window、document 等 browser 上才有的对象，因为我们在开发时使用的是浏览器环境，代码构建时是 node 环境。

解决办法：

- 先判断当前的环境是否为 browser 环境再使用：

```jsx
if (typeof window !== "undefined") {
  const language = window.navigator.language;
}
```

- 在 hooks 中使用：

```jsx
// ...

const [language, setLanguage] = useState();
useEffect(() => {
  setLanguage(window.navigator.language);
}, []);
```

#### TIPS

- 当项目中没有使用到 window 等对象时，若编译还是报错`window is not defined`，可以排查一下项目中使用的依赖项是否使用了该的对象

### 4、侧边栏组件在 ios 环境下影响 body 样式

在移动端使用 gastby + material-ui 时，如果使用侧边栏组件，在 IOS 机器下会出现 body 被莫名推出可视区域的情况。

解决办法：

给 gatsby 容器添加`overflow: hidden`样式：

```css
#___gatsby {
  overflow: hidden;
}
```

### 5、路由问题

Gatsby 项目在开发环境中，访问页面 `pageA`，使用路径`/pageA` 和`/pageA/`均可以，但若是线上环境则必须使用`/pageA/`，同时 Link 的 `active` 样式也会受到 url 的影响。这个问题是因为 Gatsby 把`/src/pages/pageA.js`编译成了`/public/pageA/index.html`。

解决办法：

- 修改 nginx 配置，实现`/pageA`到`/pageA/`的重定向，或者`try_files $uri $uri/ $uri/index.html =404;`
- Link 的 active 样式问题，可以设置`partiallyActive={true}`采用非完全匹配模式解决

```jsx
// ...

<Link
  activeClassName={headerStyle.nav__linkActive}
  partiallyActive={true}
  to="/pageA"
>
  PAGE A
</Link>
```

### 6、Link 与 a 的区别

Gatsby 的 Link 实际上和 react 的 Route 类似，会做优化和预加载，从而加快页面跳转。通过 Link 跳转的话，其实用的是 react 特有的 diff 渲染，这种渲染只会渲染两个页面不同的部分。使用 a 的话，会强制更新整个页面，也就没有了预加载的优势，导致跳转不够快、不够优雅。

#### TIPS

- 如果是跳转当前页面并且要更新数据（比如修改页面 url 的 query 时），则必须使用 a 标签

### 7、静态资源上传 CDN

- 项目在发布打包时需要先清理缓存，否则缓存文件会导致上传 cdn 出错。
- 使用 webpack 插件将静态资源上传 CDN 时，Gatsby 会将已上传 CDN 的资源重新定向到本地资源，因此在构建完以后需要重新将资源地址修改为 CDN 资源的地址。
- 动态渲染的部分，Gatsby 会以脚本的形式生成所需要的 dom，其中涉及到的静态资源路径会以如下的方式返回：

```jsx
// 编译前
// ...
import banner from "../assets/banner.png";
import skeleton from "../assets/skeleton.jpg";

const img0 = <img className="test-skeleton" src={banner} />;
const img1 = <img className="test-skeleton" src={skeleton} />;
// ...
{
  bool ? img0 : img1;
}
// ...
```

```js
// 编译后
// ...
{
  "7CQL": function (e, t, n) {
    e.exports = n.p + "static/banner-d9b4af4154903bc4909cc0d613fbb577.png";
  },
  drUt: function (e, t, n) {
    e.exports = n.p + "static/skeleton-ad6c4f26c61c9f0cbf66bf871ba09d4e.jpg";
  }
  // ...
  var c = n("q1tI"),
    a = n.n(c),
    s = n("7CQL"),
    r = n.n(s),
    i = n("drUt"),
    o = n.n(i),
    b = a.a.createElement("img", { className: "test-banner", src: r.a }),
    l = a.a.createElement("img", { className: "test-skeleton", src: o.a });
  // ...
}
// ...
```

从中可以看到静态资源路径是由字符串拼接所得，这就导致我们将静态资源上传 cnd 后，无法对编译后的文件中的静态资源路径进行替换。

解决办法：

以相对路径的方式直接引用资源，这样编译后的文件中资源路径不会被处理（还是原始字符串形式的相对路径，具体见下方），因此资源上传 cdn 后可以轻松替换路径。需要注意的是这种方式引用的资源不会被打包进最终的 public 文件夹中。

```jsx
// 编译前
// ...
const img0 = <img className="test-banner" src="assets/banner.png" />;
const img1 = <img className="test-skeleton" src="assets/skeleton.jpg" />;
// ...
{
  bool ? img0 : img1;
}
// ...
```

```js
// 编译后
// ...
var c = s.a.createElement("img", {
    className: "test-banner",
    src: "assets/banner.png",
  }),
  r = s.a.createElement("img", {
    className: "test-skeleton",
    src: "assets/skeleton.jpg",
  });
// ...
```

更多方案有待探索

### 8、路由切换时打点

若需在路由切换时发送打点，打点逻辑可以在 gatsby-browser.js 中设置：

```js gatsby-browser.js
exports.onRouteUpdate = () => {
  // sendLog
};
```

### 9、移动端响应布局

对于移动端开发，使用 flex + vw 可以满足大部分移动端页面响应式布局的需求，并且相比于媒体查询、rem 来说更加便捷高效。

### 10、js 主文件导出方式

Gatsby 项目的页面都由 pages 目录下的 js 文件构建而成，导出时必须要导出 default 形式，否则无法构建生产版本，同时非构建页面的 js 文件不能放在 pages 目录下。

```jsx
// ...
const IndexPage = () => (
  <Layout>
    <h1>IndexPage</h1>
  </Layout>
);

export default IndexPage;
```

### 写在最后的话

以上就是团队的经验分享，希望能给大家的踩坑之旅提供一点帮助，也欢迎大家挖坑补坑！
