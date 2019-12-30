---
layout: post
title: 微信小程序工程化之持续集成方案
date: 2019-10-31
categories: article
---
![](https://p1.ssl.qhimg.com/t0166864b2322539b1d.jpg)

> 本文作者：韩永刚，360奇舞团 WEB前端开发高级工程师。

本文将简单介绍一下持续集成的概念，并手把手带你在你的微信小程序项目中完成属于你的持续集成方案。

# 什么是前端工程化

所有能降低成本，并且能提高效率的事情总称为工程化。在前端项目中能够减少重复工作、扩展 javascript\html\css 本身的语言能力、解决功能复用和变更问题、解决开发和产品环境差异问题、任何时间任何地点生成可部署的软件、解决发布流程问题，都属于前端工程化。

# 什么是持续集成

持续集成是前端工程化中的一部分，是一种软件开发实践，即团队开发成员经常集成他们的工作，通常每个成员每天至少集成一次，也就意味着每天可能会发生多次集成。每次集成都通过自动化的构建（包括自动化编译，自动化测试，自动化发布）来验证项目代码，从而尽早地发现错误。

# Web项目持续集成怎么做

Web项目的持续集成方案选择比较多，并且相对成熟，这里介绍一下 **gitlab-ci** 持续集成方案。

这种方式的原理就是为项目在自己的 **linux** 服务器安装并注册 **gitlab-runner** ，注册会有一个 **token** ，服务器上运行 **gitlab-runner** 后， **runner** 会轮询的发送带 **token** 的 **http** 请求给 **gitlab** ,如果 **gitlab** 有任务了，（一般是 **git push** ），那么会把任务信息返回给 **runner** ，然后 **runner** 就开始调用注册时选的 **Executor** 来执行项目根目录下的配置文件 **.gitlab-ci.yml** ，执行后把结果反馈给 **gitlab** 。

此时我们可以编写 **.gitlab-ci.yml** 脚本，比如设定当 **test** 分支发生 **push** 时，自动运行测试用例、自动构建代码、自动将代码更新到测试人员在测的环境等任何你想在提测时需要做的事情。当 **merge** 到 **master** 时，自动更新线上代码完成上线等各种你想在上线时做的事情。

这里只要考虑的足够全面，那么之后的项目开发你只需要 **push** 到对应的分支，**gitlab-runner** 会自动完成你想做的所有构建、提测、上线操作。减少重复工作，这就是持续集成的意义所在。



# 手把手教你完成小程序的持续集成方案

小程序的持续集成可以继续使用 **gitlab-ci** 的方式，但由于小程序的构建、提测、提交体验版等操作都需要依赖于 **微信开发者工具** ，而微信开发者工具只有 **Windows** 和 **Mac** 版，所以我们需要一台 **Windows** 服务器来运行 **gitlab-runner**。

## 1. 准备工作

- 一台 **Windows** 服务器
- 一个权限为 **Maintainer** 的 **gitlab**项目

## 2. 安装必要软件

在这台 **Windows** 服务器上安装以下软件

- **Git**
- **Node.js**
- **微信开发者工具**

## 3. 配置gitlab-runner

1. 首先下载 **gitlab-runner**

   > https://docs.gitlab.com/runner/install/windows.html

   下载完成后将其移动到合适的路径后重命名为 **gitlab-runner.exe**

2. 在 **Windows** 服务器中打开 **powershell** 并进入 **gitlab-runner.exe** 所在目录，然后执行以下命令

```powershell
.\gitlab-runner.exe register
```

![](https://p1.ssl.qhimg.com/t015868f4a26eb1c187.png)

> 1. Please enter the gitlab-ci coordinator URL
>
>    打开想要设置 **CI** 的 **gitlab** 项目，进入页面 **settings** > **CI/CD** > **Runners** > **Expand**，找到 **Set up a specific Runner manually** ，输入 **Specify the following URL during the Runner setup:** 下的地址
>
> 2. Please enter the gitlab-ci token for this runner
>
>    输入 **Use the following registration token during setup:** 下的token字符串
>
> 3. Please enter the gitlab-ci description for this runner
>
>    输入一个描述
>
> 4. Please enter the gitlab-ci tags for this runner
>
>    输入一个标签，该标签对应该runner
>
> 5. Please enter the executor
>
>    这里输入 **shell** 就好
>

此时刷新 **gitlab** 页面会新增一个 **gitlab-runner**

![](https://p1.ssl.qhimg.com/t01365dad71b81d349e.png)

3. 执行命令 **install**

```powershell
.\gitlab-runner.exe install
```

4. 执行命令 **start**

```powershell
.\gitlab-runner.exe start
```

此时刷新 **gitlab** 页面，之前的 **gitlab-runner** 会更新为以下状态，表示 **gitlab-runner** 配置完成，已经可以开始工作。

![](https://p1.ssl.qhimg.com/t01ee74b28ae33622a9.png)

## 4. 修改gitlab-runner服务的登录账号

![](https://p1.ssl.qhimg.com/t0129f4e0808e3a95fd.png)

由于 **gitlab-runner** 服务默认登录账号为 **authority\system** ，而这个账号在执行微信开发者工具命令行时会出现报错，所以我们需要更改 **gitlab-runner** 服务的登录账号为正确账号并重启该服务。

> 右击计算机 -> 管理 -> 服务和应用程序 -> 服务
>
> 找到 **gitlab-runner** 服务
>
> 右击 **gitlab-runner** -> 属性 -> 登录 -> 此账号 -> 输入可以正确使用微信开发者工具命令行的账号和密码 -> 确定 -> 重启动此服务

修改完成后账号会正确被更改

![](https://p1.ssl.qhimg.com/t01a639e59da0c1654b.png)

## 5. 配置微信开发者工具

1. 使用你的微信账号登录开发者工具
2. 设置 -> 安全 -> 服务端口 -> 开启

![](https://p1.ssl.qhimg.com/t015159defb543bc007.png)

## 6. 配置.gitlab-ci.yml

在项目根目录创建 **.gitlab-ci.yml** 文件，并填写以下配置。

```ruby
stages: # 定义阶段用于执行任务
  - build
  - deploy

build_job: # 定义 build 任务，名称可以随意命名，只是为了方便理解和区分
  stage: build # 该任务属于 build 阶段，要严格与stages中定义的命名一致
  only:
    - master
  tags: # tags 指定运行在哪个 Runner 上，这里需要在我们刚注册的 Runner 运行，和注册时的 mp_win7 匹配
    - mp_win7
  before_script: # 执行script之前的钩子
    - whoami 
  script: # 执行下面脚本，这里可以自定义配置您的构建任务
    - echo "build" # 可以在这里执行您项目的构建编译操作

deploy_job: # 定义 deploy 任务，名称可以随意命名，只是为了方便理解和区分
  stage: deploy # 该任务属于 deploy 阶段
  only:
    - master
  tags: # tags 指定运行在哪个 Runner 上，这里需要在我们刚注册的 Runner 运行，和注册时的 mp_win7 匹配
    - mp_win7
  script: # 执行下面脚本，这里可以自定义配置您的部署任务
    - C:\software\wechatDevTool\cli.bat -u 0.1.0@"$PWD" --upload-desc 最新的描述 # 这里使用微信开发者工具提供的命令行工具进行上传体验版操作
```

修改完成后将代码 **push** 到远程仓库，会自动触发CI任务。

![](https://p1.ssl.qhimg.com/t019d51361cb922a675.png)

![](https://p1.ssl.qhimg.com/t01f0ac08aa114bca76.png)

此时登录微信小程序官网后台，就可以看到刚刚 **push** 代码时由 **gitlab-ci** 自动上传的体验版了。

![](https://p1.ssl.qhimg.com/t011ca19b9ff40db6dc.png)

有关 **.gitlab-ci.yml** 的详细配置方法可以参考以下文章。

> http://urlqh.cn/mK3tA
>
> http://urlqh.cn/mNjWs

## 7. 持续集成项目方案

> 以下为本人最近在做的 **360瞭望台** 小程序的持续集成方案，您可以根据自己的需要加以改进，并完成属于你的持续集成方案。

1. 由于编写 **.gitlab-ci.yml** 时需要用到微信开发者工具的命令行，所以为了便于团队成员使用，我开发了一个 **node.js** 脚本，并发布为 **npm** 模块，用于在 **.gitlab-ci.yml** 调用 **Windows** 虚拟机中的上传命令。

   ![](https://p1.ssl.qhimg.com/t01a2d503871bbc77ad.png)

2. 约定团队开发流程，开发新需求时创建 **feature/0.5.1**(需求版本号) 分支，当 **push** 代码时会自动触发 **CI** 任务，并在虚拟机修改为 **test** 环境后提交体验版。

3. 发完测试报告后可以将小程序提审时，我们需要将 **feature/0.5.1** 分支 **merge** 到 **audit** 分支，此时会自动触发 **CI** 任务，并在虚拟机修改为 **production** 环境后提交体验版。 **audit** 分支需要添加为受保护的分支，不允许直接 **push** 代码，如果审核没通过那么可以以 **audit** 为基础新建 **feature/0.5.2** 分支进行调整后重新 **merge** 到 **audit**。

4. 当审核通过后我们需要将 **audit** 分支代码合并到 **master** 分支，**master** 分支应该永远与线上代码保持同步。

# 相关资料

> http://urlqh.cn/mKi7T
>
> http://urlqh.cn/mJPCC
>
> http://urlqh.cn/mIWRk
>
> http://urlqh.cn/mJoY2
>
> http://urlqh.cn/mLCIB
