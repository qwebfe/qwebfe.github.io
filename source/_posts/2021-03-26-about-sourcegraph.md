---
layout: post
title: 手把手教你搭建一个基于 sourcegraph 代码搜索网站
date: 2021-03-26
categories: 教程
coauthor: mileOfSunshine
tags:
  - Sourcegraph
  - Github
  - GitLab
  - 授权登录
  - 智能搜索
---

现有企业的代码一般都是托管在 GitLab 上，其理由，无非是免费、可以部署到自己的服务器上，所有信息都掌握在自己手中，非常适合团队内部协作。而 Github 虽也致力于免费托管开源代码，但如需建立私有仓库就需付费，看到付费两字，很多人都望而却步。但在代码搜索方面，Github 做的比 GitLab 精彩。

如何弥补 GitLab 在智能搜索方面的缺憾呢，笔者想推荐个工具给你 ——— [Sourcegraph](https://about.sourcegraph.com/get-started)，一款开源的代码搜索浏览工具。检索速度也是毫秒级的。

如何搭建呢？你听我娓娓道来。🥰

![](https://p5.ssl.qhimg.com/t0139aea3fa431d9ea6.png)

<!--more-->

> 💡小贴士：
> 本文是在以下环境中进行实践的。
>
> 操作系统：Linux
> 环境要求：安装了 docker


## 🛠️安装 Sourcegraph 

目前官方就提供了一个使用 Docker 安装的示例，命令也是相当简短：

```bash
docker run --publish 7080:7080 --publish 127.0.0.1:3370:3370 --rm --volume ~/.sourcegraph/config:/etc/sourcegraph --volume ~/.sourcegraph/data:/var/opt/sourcegraph sourcegraph/server:3.26.0
```

笔者将其进行了优化处理，为容器指定了个名称，更换了两个本机端口，以后台模式启动容器，方便后续说明。

```bash
docker run -d \
 --name sgdev2 \
 --publish 27080:7080 \
 --publish 127.0.0.1:23370:3370 \
 --rm \
 --volume ~/.sourcegraph/config:/etc/sourcegraph \
 --volume ~/.sourcegraph/data:/var/opt/sourcegraph \
 sourcegraph/server:3.26.0
```

> Usage：`docker run [OPTIONS] IMAGE [COMMAND] [ARG...]` 创建一个新的容器并运行一个命令
- `-d/--detach`：后台模式启动一个容器
- `--name`：为容器指定一个名称
- `--rm`：退出时自动移除容器
- `-p/--publish`：指定端口映射，格式为：主机(宿主)端口:容器端口。关于端口映射的具体说明，可参考文章[Docker端口映射](https://blog.csdn.net/qq_29994609/article/details/51730640)。官方示例中就是将本机的两个端口7080和3370映射到容器的端口7080和3370上，若是本机端口7080和3370被占用，也可更改为未占用的端口号，例如27080和23370，就如笔者优化后的示例。
- `-v/--volume`：指定容器卷。上面的命令指定了两个卷，即在本机创建数据卷 `~/.sourcegraph/config`（配置） 和 `~/.sourcegraph/data`（数据）（题外话，可以改成任何你想放置的位置，例如：`~/.sourcegraph2/config`、`~/.sourcegraph2/data`），分别挂载到容器的 `/etc/sourcegraph` 和 `/var/opt/sourcegraph` 路径上。这样容器运行过程中，在容器中生产的数据会被保存到容器所在的节点上（`~/.sourcegraph/config` 和 `~/.sourcegraph/data`）。

若你不设置 `-d`，执行成功后，会出现以下提示，让你访问 `http://127.0.0.1:7080`，但上述命令我们已经将 7080 映射到 27080 端口。故正确的访问地址是 `http://127.0.0.1:27080`。

![](https://p3.ssl.qhimg.com/t0168e932b15a6f1c98.png)

若你是在服务器上执行 docker 命令，那我们还需进行 nginx 配置才能访问。

```conf nginx.conf
server {
  listen 80;
  server_name sgdev2.example.com;

  location / {
    proxy_pass http://127.0.0.1:27080;
    proxy_set_header X-Real-IP  $remote_addr;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_set_header Host $host;
  }
}
```

修改完配置，我们需重启下nginx `/usr/sbin/nginx -s reload`（没权限就加sudo）。接着在客户端上配置下 hosts，假设服务器IP为 `10.11.xx.xx`，配置完后直接访问 `http://sgdev2.example.com`。出现如下界面就成功了，第一次访问页面注册的是管理员，注册完登录就可以进行站点设置。

``` hosts
10.11.xx.xx sgdev2.example.com
```

![](https://p5.ssl.qhimg.com/t011017c72c20b43768.png)

>💡docker 小贴士：
>
> ```bash
> # sgdev2 为容器名
> 
> docker ps # 查看运行中的容器
> 
> docker restart sgdev2 # 重启容器 sgdev2
>
> docker start sgdev2 # 启动容器 sgdev2
>
> docker stop sgdev2 # 停止运行中的容器 sgdev2
>
> docker kill -s KILL sgdev2 # 杀掉一个运行中的容器
> ```

## ⚙️配置 Sourcegraph 网站 

### 仓库设置

入口：`Site admin` > `Repositories` > `Manage code hosts`

sourcegraph 提供了多种仓库供你选择。笔者选择了自己接触最多的 Github 和 GitLab 分别进行配置、解说。

![](https://p4.ssl.qhimg.com/t01e985c2331e5932ea.png)

#### 配置 Github 仓库

点击 Github，进入仓库配置页面：

```json
{
  "url": "https://github.com",
  "token": "<access token>",
  "orgs": [
    "<你的github用户名>"
  ]
}
```
`<access token>` 生成步骤详见 [Create a GitHub access token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token)，授权范围（scope）设置为 **repo**。

设置成功后，点击 `Repositories` > `Repository status` 就能查看到所有授权的仓库。开发者可根据需要重复上述操作即可。

#### 配置 Gitlab 仓库

点击 GitLab，进入仓库配置页面：

```json
{
  "url": "https://<你的gitlab域名>",
  "token": "<access token>",
  "projectQuery": [
    "projects?membership=true&archived=no"
  ]
}
```
`<access token>` 生成步骤详见 [Create a GitLab access token](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html#creating-a-personal-access-token)，授权范围（scope）设置为 **api**

> 若你配置了 GitLab 授权登录（下文即将介绍），希望**查找的仓库权限能跟授权账号走**。可以在仓库配置中加入 `Enforce permissions (OAuth)`
>
> ```json
> {
>   "authorization": {
>     "identityProvider": {
>       "type": "oauth"
>     }
>   }
> }
> ```

### 用户授权

入口：`Site admin` > `Configuration` > `Site configuration`

进入配置页，我们可以看到默认认证方式是用户注册登录，管理员可以直接在后台添加用户，此外网站还支持 GitLab / Github 授权登录。接下来笔者着重对这两种授权方式展开说明。

#### GitLab 授权登录

首先，创建一个 [GitLab 授权应用程序](https://docs.gitlab.com/ee/integration/oauth_provider.html)。

> 💡小贴士：
> 1. 在 GitLab 右上角选择你的头像
> 2. 选择 **Settings** （或是 **Edit profile**）
> 3. 在左侧边栏选择 **Applications**
> 4. 输入 **Name**，**Redirect URI**，**Scopes**(授权范围设置为`api`、`read_user`)。**Redirect URI**是用户授权回调的地址，形如：`http://sgdev2.example.com/.auth/gitlab/callback`
> 5. 点击保存，就能看到 `Application ID` 和 `Secret`。

![](https://p5.ssl.qhimg.com/t012dfa80c5385b4daa.png)

然后，回到 sourcegraph 网站站点进行配置。

> 💡小贴士：
>  1. 在你的 sourcegraph 站点的右上角选择你的头像
>  2. 选择 **Site admin**
>  3. 在左侧边栏选择 **Site configuration**
>  4. 点击 **Add GitLab sign-in**，会自动添加如下授权代码。

```json Site configuration
{
  "auth.providers": [
    {
      // See https://docs.sourcegraph.com/admin/auth#gitlab for instructions
      "type": "gitlab",
      "displayName": "GitLab",
      "url": "<GitLab URL>",
      "clientID": "<client ID>",
      "clientSecret": "<client secret>"
    }
  ]
}
```
`type` 和 `displayName` 保持默认。`<GitLab URL>` 改为你的 GitLab 地址，`<client ID>` 改为之前获得的 `Application ID`，`<client secret>` 改为 `Secret`，保存即可。

此时你退出，再登录就会出现，GitLab 授权按钮。

![](https://p0.ssl.qhimg.com/t01fe2400ac3070f849.png)


#### Github 授权登录

与 GitLab 授权登录类似，先创建一个[Github 授权应用程序](https://developer.github.com/apps/building-oauth-apps/creating-an-oauth-app/)

> 💡小贴士：
> 1. 在 Github 右上角选择你的头像
> 2. 选择 **Settings**
> 3. 在左侧边栏选择 **Developer settings**
> 4. 在左侧边栏选择 `OAuth Apps`，点击 `Register a new application`
> 4. 输入 **Application Name**, **Homepage URL**，**Authorization callback URL**是用户授权回调的地址，形如：`http://sgdev2.example.com/.auth/github/callback`
> 5. 点击 `Register application`，就能看到 `Client ID` 和 `Client secret`。


![](https://p2.ssl.qhimg.com/t016bab50fbb17519d9.png)

![](https://p4.ssl.qhimg.com/t0101a836846565dc69.png)

同样，回到我们的 sourcegraph 配置页。点击 **Add Github sign-in**，会自动添加如下授权代码：



```json Site configuration
{
  "auth.providers": [
    {
      // See https://docs.sourcegraph.com/admin/auth#github for instructions
      "type": "github",
      "displayName": "GitHub",
      "url": "https://github.com/",
      "allowSignup": true,
      "clientID": "<client ID>",
      "clientSecret": "<client secret>"
    }
  ]
}
```
`type` 和 `displayName` 保持默认。`url` 改为你的 Github 地址，`<client ID>` 改为之前获得的 `Client ID`，`<client secret>` 改为 `Client secret`，保存即可。


## 小结❤️

按理安装完 Sourcegraph，进行仓库配置后，站点就能正常投入使用了。关于用户授权，是笔者经历一番探索的总结，为有此需求的人在阅读官方文档中即将陷入迷茫提供的一个提示。


温馨提示：

- 若是你通过 GitLab 授权管理用户，需确保网站管理员为代码仓库的管理员，这样能保证用户的搜索池全面。

- 免费版的 Sourcegraph 只能注册10个用户哦！

- 升级应该在 Sourcegraph 的连续次要版本之间进行。例如，如果您正在运行Sourcegraph 3.1，并且想要升级到3.3，则应该先升级到3.2，再升级到3.3。


> 本文作者： mileOfSunshine
> 本文链接： https://qwebfe.github.io/2021/03/26/about-sourcegraph/
> 版权声明：文章是原创作品。转载请注明出处！