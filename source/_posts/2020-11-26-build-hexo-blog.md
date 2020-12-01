---
layout: post
title: 博客之旅：从 Jekyll 到 Hexo
date: 2020-11-26
categories: 教程 
tags:
  - Hexo
  - Github
  - Jekyll
  - Travis
  - 博客
---

![](https://user-images.githubusercontent.com/16272760/99784261-872d3200-2b56-11eb-807c-869042d1f6e8.png)

初出茅庐之时，看到他人的博客，总会产生一丝新鲜感，好奇之心让你不知不觉跟随着他人的脚步，在写作平台开始展露拳脚。

发布了两三篇文章后，又发现有些人并非在写作平台写作，而是在拥有自己域名的博客平台发布。就像小时候看到邻居小孩吃糖，不知甜不甜就想着自己也能吃一块。然后就开始折腾了，购买域名空间，独立搭建博客。

从写文章到搭建博客这思路是对的，但不乏带有点跟风之意。博客，技术圈的个人名片。特别是在招聘面试时，部分面试官会特别关注面试者是否有个人博客，一来从博客上能看出面试者的表达能力和涉猎，二来能看到个人的代码水平和编码习惯。

此外，博客的书写对个人也是一种技能锻炼。资料的整理，特别是技术问题的整理，有助于自己下一次遇到类似的问题时能否更好的回忆细节和复用相关代码。

但如何能又快又好的搭建，这是讲究方法的。

<!--more-->

笔者在开发博客前，简单做了调研，了解到当下最受欢迎的搭建模式就是借助[GitHub Page](https://github.com/)和静态博客框架。接下来向大家介绍整个博客的搭建流程。

# Github Pages 入门

1. 首先，[注册Github帐号](https://github.com/join)。
![注册Github帐号](https://p1.ssl.qhimg.com/t01b00009360875756d.png)
2. 然后，将代码托管在 Github 上。
新建一个 repository。如果你希望你的站点能通过 <你的 GitHub 用户名>.github.io 域名访问，你的 repository 应该直接命名为 <你的 GitHub 用户名>.github.io。例如，`qcft.github.io`。
3. 最后配置 Github Pages，具体细节详见 [Getting Started with GitHub Pages](https://guides.github.com/features/pages/)。


# 静态博客框架

根据笔者调研，网上讨论最多的博客框架当属 Jekyll 和 Hexo。有人说，[Jekyll](https://jekyllrb.com/) 就是为了写博客打造的。Github Pages 中提供的 Jekyll 主题似乎也验证了这点。早期组内的博客也是基于 Jekyll 搭建的，其原因我想大概是 GitHub 自带 Jekyll 引擎，无需部署静态页面，只放项目源码即可。后因开发环境依赖问题不便维护，便弃 Jekyll 从 Hexo。据说 Hexo 的出现是因为 Jekyll 比较慢，大概是早期 Hexo 还不够成熟，还不支持自动化部署，也给使用者带来了不便，但那个时候已经过去了。接下来向大家具体介绍 Jekyll 和 Hexo。 

## Jekyll


```bash
gem install bundler jekyll

jekyll new my-awesome-site

cd my-awesome-site

bundle exec jekyll serve

# => Now browse to http://localhost:4000
```

通过以上几个命令可以看出，Jekyll 是基于 Ruby 实现的，安装 Jekyll 需要搭建 Ruby 环境，但在 Windows 搭建 Ruby 环境并不被推荐。若你是 Windows 用户，那可能一开始就跌倒在安装 Jekyll 的起跑线上。

我们组的早期博客基于 Jekyll，运行了小一年时间，但用户体验不是很好。受环境搭建影响，多数成员无法启动本地服务器，都是盲写文章，推送上线后，发现问题再修复。有一次把文章日期提前了一天，上线后新文章一直未被构建。无法本地预览，一定程度上影响排查效率。后面求助了同事，也没找到原因，但在第二天早上被告知文章构建成功了。笔者一时蒙蔽，后查阅了各手资源，才得知以下这个答案：

> Jekyll 是不会构建未来日期的文章的！

在主题方面，有想法的你可自行选择[Jekyll主题](https://github.com/topics/jekyll-theme)。按照安装提示，或基于gem，或只取远程主题，或直接直接拷贝至项目中，然后配置 `_config.yml` 文件即可。

```yml _config.yml
theme: minimal-mistakes-jekyll
```

-------

厌倦了 Jekyll 的那段爱恨情仇，最终投向了 Hexo 的怀抱。

-------

## Hexo

> A fast, simple & powerful blog framework
> 快速、简洁且高效的博客框架

Hexo 是基于 NodeJs 实现，在 Windows 上安装 NodeJs 开发环境可谓是相当简单。何其美哉！乐哉！

Hexo 默认的主题是 `landscape`，接触过博客园的童鞋可能觉得该主题有它的味道。萝卜青菜各有所爱，你可自行选择[主题](https://hexo.io/themes/)。关于 `Hexo` 的主题，笔者想说是我喜欢的调调😉，本组博客应用的主题是[NexT](https://theme-next.js.org/)


### 建站

啥也不说了，搬砖吧。

#### 安装

```bash
npm install -g hexo-cli
```

#### 初始化

```
hexo init <folder>

cd <folder>

npm install

npm server
```

> M：咦? 这一套指令好像在哪见过👀。
> H：🙊 vue ?
> M：嗷呜...

项目初始化完成后，指定文件夹的目录如下：

```
.
├── _config.yml # 网站的 配置 信息
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes

```

- scaffolds: 模版文件夹, 当你新建文章时，Hexo 会根据 scaffold 来建立文件
- source: 资源文件夹，存放用户资源的地方。除 _posts 文件夹之外，开头命名为 _ (下划线)的文件 / 文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 public 文件夹，而其他文件会被拷贝过去。
- themes: 主题文件夹，把你想要的主题拷贝至当前目录即可。
- _config.yml: 网站的配置信息，Hexo 这个点很友好，它给相关配置都做了注释和[文档说明](https://hexo.io/zh-cn/docs/configuration.html)。
  - theme: 你选用的主题，默认为 `landscape`

### 写作

项目创建完成后，我们就可以开始写作了。关于写作，Hexo cli 也提供了相关指令，可快速创建一篇新文章或者新的页面。

```bash
hexo new [layout] <title>
```

- layout，Hexo 有三种默认布局：post、page 和 draft。基于不同类型创建的文件会存储在各自对应的路径中，你可以编辑 `scaffolds/*` 下的模板文件，保证新页面的规范性。
- title，Hexo 默认以标题做为文件名称，但你可编辑 new_post_name 参数来改变默认的文件名称，举例来说，设为 :year-:month-:day-:title.md 可让你更方便的通过日期来管理文章。

```yml _config.yml
new_post_name: :year-:month-:day-:title.md
```

执行指令时只需输入文件名，创建成功会自动带上当天日期，文件内容也会按照模板格式显示。

```bash
hexo new test # ==> source/_posts/2020-12-01-test.md
```


若你想了解更多使用方法，可前往[hexo官网](https://hexo.io/zh-cn/docs/)。

### 部署
Hexo 官方介绍了三种部署方式。由于我们代码托管在了 Github 上，那笔者就部署到 `GitHub Pages` 展开介绍。

1、将你本地的 Hexo 站点文件夹推送到 repository（`<你的 GitHub 用户名>.github.io`） 中。默认情况下 public 目录将不会被推送到 repository 中，你应该检查 `.gitignore` 文件中是否包含 public 一行，如果没有请加上。
2、将 [Travis CI](https://github.com/marketplace/travis-ci) 添加到你的 GitHub 账户中。
![选择一个账号](https://p4.ssl.qhimg.com/t012a4eb716b67ab0e7.png)
![授予此应用访问权限](https://p3.ssl.qhimg.com/t0150046ab66f6286a6.png)
![授予项目访问权限](https://p0.ssl.qhimg.com/t0177f583632427bf95.png)

3、前往 GitHub 的 [Applications settings](https://github.com/settings/installations)，配置 Travis CI 权限，使其能够访问你的 repository。若你在第2步忘记了设置 `Travis CI` 的仓库权限，也可在此重新配置。

![配置 Travis CI 权限](https://p5.ssl.qhimg.com/t01a8332cda870c3db1.png)

![设置 Travis CI 的仓库权限](https://p2.ssl.qhimg.com/t01867de50a8cd84260.png)


4、保存设置后你应该会被重定向到 Travis CI 的页面。如果没有，请 [手动前往](https://travis-ci.com/)。
5、在浏览器新建一个标签页，前往 GitHub [新建 Personal Access Token](https://github.com/settings/tokens)，只勾选 `repo` 的权限并生成一个新的 Token。Token 生成后请复制并保存好。
![新建 Personal Access Token](https://p5.ssl.qhimg.com/t012b03686dfd70e308.png)
6、回到 Travis CI，前往你的 repository 的设置页面，在 **Environment Variables** 下新建一个环境变量，**Name** 为 `GH_TOKEN` ，**Value** 为刚才你在 GitHub 生成的 **Token**。确保 **DISPLAY VALUE IN BUILD LOG** 保持 **不被勾选** 避免你的 Token 泄漏。点击 **Add** 保存。
![](https://p4.ssl.qhimg.com/t01d2f70045349a4ccf.png)
7、在你的 Hexo 站点文件夹中新建一个 .travis.yml 文件：

```yml .travis.yml
sudo: false
language: node_js
node_js:
  - 10 # use nodejs v10 LTS
cache: npm
branches:
  only:
    - master # build master branch only
script:
  - hexo generate # generate static files
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GH_TOKEN
  keep-history: true
  on:
    branch: master
  local-dir: public
```

8、将 `.travis.yml` 推送到 repository 中。Travis CI 应该会自动开始运行，并将生成的文件推送到同一 repository 下的 `gh-pages` 分支下.
9、在 GitHub 中前往你的 repository 的设置页面，修改 `GitHub Pages` 的部署分支为 `gh-pages`。
![设置页面gh-pages](https://p1.ssl.qhimg.com/t01c28f4fde1a753294.png)
10、前往 `https://<你的 GitHub 用户名>.github.io` 查看你的站点是否可以访问。这可能需要一些时间。


或许你觉得这种基于 Travis CI 的部署方式配置繁琐，但你要知道一旦配置走通，后期部署都走自动化，对于团队博客而言能在一定程度上减轻心智负担。你要是真不想配置，推荐使用官方的[一键部署](https://hexo.io/docs/one-command-deployment)，但这种部署方式每次都要人工执行操作。

# 后记

还是那句话，萝卜青菜个有所爱。从 Jekyll 到 Hexo，这趟旅行虽说不易，但旅途中的风景也是令人难忘。若你还有更好用的博客框架，可留言推荐给笔者和大家。