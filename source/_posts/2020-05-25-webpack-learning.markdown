---
layout: post
title: webpack 那些事儿
date: 2020-05-25
categories: 前端 
tag: webpack
---

最近看了些webpack相关的视频，对webpack有了进一步的学习，这里做了些总结，可能不全面、片面，欢迎大家指出，一起学习讨论。

提到webpack先想到的是文件的打包，对代码进行构建，生成的结果用于生产环境。下面以自问自答的形式进行记录：

### 1. 为什么需要对前端代码进行打包呢

大概有以下几点原因：

- 开发分工的变化，早期前端开发负责页面的样式实现，简单的交互逻辑。随着单页面应用的盛行，不仅要还原设计稿，较好的过渡效果，动画效果，还需要关注页面的路由变化，页面复杂的交互逻辑，乃至部分接口数据处理逻辑。
- 框架的变化，早期js没有统一的规范，项目中引入js库方便浏览器的兼容。随着技术的发展变革，模块化编程思想的普及，Angular，React，Vue相继登场。
- 语言的变化，逐步规范化，各种预处理器的诞生，eg： sass，typescript
- 环境的变化，早期页面在移动端和浏览器中运行，现在还可以在服务端渲染SSR
- 工具的变化，现在的前端框架有相对完整生态，配套的cli，各种loader，插件，在这些影响下进行模块化开发，项目的文件数量增多，如：采用jade开发模版，sass编写css，typescript实现交互逻辑，可以大大的提高开发效率，那么语言编译就不可或缺了，此时webpack可以解决你的无后顾之忧。
<!--more-->
### 2. 既然代码的打包势在必行，那webpack都能做哪些事情呢

  webpack的功能还是很强大的，如：代码转换（各种loader）、模块热替换、tree-shaking、代码分离、懒加载、模块合并（创建library）、代码发布，这些都是官网列出的，具体内容可以查看：[https://www.webpackjs.com/guides](https://www.webpackjs.com/guides)

### 3. webpack功能很多，不足有哪些呢

凡事都是有利有弊的，代码打包除了webpack，还有rollup，gulp等，有竟品的存在，就代表webpack有缺点：

- 配置复杂；虽然在webpack4.0的版本中推出了零配置的方案，但大多数的使用场景，这个零配置是不满足的，需要开发者根据项目情况引入相应的loader、plugin进行配置。
- 体积大；这个体积是指打包的结果，网上可以查到很多解决该问题的方法，提供了相关的组件进行处理。
- 编译时间长；开发者都会选择实现新语法进行开发，所以编译是需要对代码进行babel处理，同步处理多文件，时间自然少不了，可以引入happyPack插件实现多进程并行处理，时间会有些优化。

### 4. webpack构建流程是怎样的呢

整个流程是一个串行的过程，流程如下：

- 根据webpack.config.json中配置进行参数初始化
- 通过配置中的 entry 找出所有的入口文件
- 从入口文件出发，调用loader对模块内容进行编译处理，并递归的解析出模块依赖树，一一进行解析
- 解析完成得到最终的内容以及它们之间的依赖关系
- 根据输出模块配置，将内容组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表
- 根据配置中的输出路径和文件名，把文件内容写入到文件系统

### 5. 如何开发自定义loader

(1) loader定义
webpack提倡模块化，loader就是文件的加载器，将所有文件处理成可加载的模块。loader可以链式调用，即一个文件可以被多个loader进行处理，loader的执行与配置中传入数组的顺序相反。在下面的示例中，从 sass-loader 开始执行，然后继续执行 css-loader，最后以 style-loader 为结束。

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          { loader: 'style-loader' },
          {
            loader: 'css-loader',
            options: {
              modules: true
            }
          },
          { loader: 'sass-loader' }
        ]
      }
    ]
  }
};
```

(2) loader的实现
以一个简单需求为例：处理txt文件，字符串以空格分割进行翻转

- 创建loader/reverse.js

```javascript
module.exports = function(src) {
  if(src) {
	  src = src.split(' ').map(item => (item.charAt(0).toUpperCase() + item.slice(1))).join(' ')
		return src
	}
}
```

- 在配置中增加

```javascript
rules: [{
	test: /\.txt$/,
	use: [
	  './loader/reverse.js',
	]
}]
```

这样一个简单的loader就实现了，想要看loader的执行顺序，可以在loader中增加调试信息打印查看即可.异步loader需要使用异步方法，可在API官网查看：[https://webpack.js.org/api/loaders/](https://webpack.js.org/api/loaders/)

### 6. 如何开发插件

- 插件定义：
 Webpack 会在特定的时间点广播出特定的事件,插件在监听到感兴趣的事件后会执行特定的逻辑,并且插件可以调用 Webpack 提供的 API 改变 Webpack 的运行结果。
webpack 插件由以下组成：
一个 JavaScript 命名函数。
在插件函数的 prototype 上定义一个 apply 方法。
指定一个绑定到 webpack 自身的事件钩子。
处理 webpack 内部实例的特定数据。
功能完成后调用 webpack 提供的回调。
- 插件实现
以一个简单需求为例：打包生成html中追加一段js

```javascript
class AddScriptPlugin {
  // 构造方法
  constructor(options) {
    console.log('AddScriptPlugin constructor:', options)
  }
  // 应用函数
  apply(compiler) {
    // 绑定钩子事件
    compiler.plugin('emit', function (compilation, callback) {
      const fx = `(function(){console.log("I'm fx code, I'm here, hahaha ~")})()`
      const newContent = compilation.assets['main.js'].source() + fx
      compilation.assets['main.js'] = {
        source: function () {
          return newContent;
        },
        size: function () {
          return newContent.length;
        }
      };

      callback();
    })
  }
}

module.exports = AddScriptPlugin
```

在配置中将其引入，查看打包结果即可。

### 7.如何编写简单的打包工具，实现代码同步、异步的加载

简单流程如下:

- 将代码处理成commonjs的形式，浏览器可以加载运行
- 解析文件间的依赖关系，采用键值对的形式，以文件路径为key，其值是文件内容
- 同步的脚本在加载时通过文件名进行查找加载
- 异步的加载通过jsonp的方式，针对使用require.ensure加载的问题，单独打包成jsonp进行加载

### 8.介绍webpack-dev-server的原理

下图是参考文章中截取，详细流程参见[https://juejin.im/post/5de0cfe46fb9a071665d3df0#heading-14](https://juejin.im/post/5de0cfe46fb9a071665d3df0#heading-14 "探索 webpack5 新特性Module federation在腾讯文档的应用")

![](http://p2.qhimg.com/t0172b886aa2215d864.jpg)

### 9.webpack5.0中Module Federation的介绍

使用官网的demo进行体验：[module-federation-examples/basic-host-remote](https://github.com/module-federation/module-federation-examples "module-federation-examples/basic-host-remote")
项目的结构如下：

![](http://p406.qhimgs4.com/t0148c597401c530728.png)

项目中有app1，app2两个文件夹也就是两个有“牵扯”的项目，分别看下配置文件

![](http://p406.qhimgs4.com/t011959605aa798c4a0.png)
![](http://p406.qhimgs4.com/t012fa5055eee81e629.png)

配置属性：
name，必须，唯一 ID，作为输出的模块名，使用的时通过 ${name}/${expose} 的方式使用；
library，必须，其中这里的 name 为作为 umd 的 name；
remotes，可选，表示作为 Host 时，去消费哪些 Remote；
exposes，可选，表示作为 Remote 时，export 哪些属性被消费；
shared，可选，优先用 Host 的依赖，如果 Host 没有，再用自己的；

可以看出app2项目中依赖app1中的button组件，且直接依赖app1中打包生成的文件，使用情况如下：

![](http://p406.qhimgs4.com/t019af27758b3772b4e.png)

代码解析见[https://mp.weixin.qq.com/s/iS-prT1xZPV6cpH7MHRRdQ](https://mp.weixin.qq.com/s/iS-prT1xZPV6cpH7MHRRdQ)

app项目配置插件后，打包index.html时头部会追加remoteEntry.js，它用于加载app2项目中的公用资源。app1项目依赖app2主要分两部分：

- 对于shared部分，app1中会自己打包，只是先判断，app2已经加载注册则直接使用，没有的话，就加载打包的内容；
- 对于expose部分，如果app2项目挂掉，会影响app1中的使用，因为app1中无button的内容。

在生产环境的使用中，对于shared内容通过插件使用，可以使用缓存减少加载时间，但对于exposed的内容，可能需要慎重考虑。

### 参考

- 热更新：[https://juejin.im/post/5de0cfe46fb9a071665d3df0](https://juejin.im/post/5de0cfe46fb9a071665d3df0)
- webpack流程：[https://juejin.im/post/5dc01199f265da4d12067ebe](https://juejin.im/post/5de0cfe46fb9a071665d3df0)
- webpack流程：[https://mp.weixin.qq.com/s/LI-SkBoPA94Ply6Qes92PA](https://juejin.im/post/5de0cfe46fb9a071665d3df0)
- 编写loader：[https://segmentfault.com/a/1190000012718374](https://segmentfault.com/a/1190000012718374)
- 编写plugin：[https://www.webpackjs.com/contribute/writing-a-plugin/](https://www.webpackjs.com/contribute/writing-a-plugin/)
- webpack5.0: [https://mp.weixin.qq.com/s/iS-prT1xZPV6cpH7MHRRdQ](https://www.webpackjs.com/contribute/writing-a-plugin/)
- webpack5.0：[https://developer.aliyun.com/article/755252](https://www.webpackjs.com/contribute/writing-a-plugin/)
- webpack5.0:  [https://juejin.im/post/5e9eb3de6fb9a03c7d3d1647](https://www.webpackjs.com/contribute/writing-a-plugin/)
