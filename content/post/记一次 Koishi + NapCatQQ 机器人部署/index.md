---
title: "记一次 Koishi + NapCatQQ 机器人部署"
date: 2025-08-28T15:30:00+08:00
draft: false
description: "这么简单的事情给我干麻了。"
readingTime : true
categories:
- 技术分享
tags:
- 琐记
---

在给地形师茶馆搭建 QQ 和 Discord 互联 Bot 的过程中踩了不少坑，所以写一下搭建流程，有需要的朋友可以参考以下。

------

## 1Panel 的安装

我直接选择了腾讯云的 1Panel 镜像一键安装。如果不想用这种方式，可以参考 1Panel 官网的教程。安装好 1Panel 的同时，Docker 也会自动安装完成。

------

## Koishi 的安装

Koishi 官方不推荐使用 Docker 安装，并禁止在论坛和用户群中讨论 Docker 及 Onebot 相关内容。但我认为用 Docker 安装 Koishi 是最省事的方式。

既然我们已经安装了 1Panel，可以直接在“应用商店”中搜索并安装 Koishi，安装时记得勾选“开放端口”。

为什么推荐从应用商店安装？

其中的一部分原因是很多人使用无图形化界面的 Linux 系统安装 Koishi，而这样就无法直接在本机操作 Koishi 的控制台，并且 Koishi 默认监听 `127.0.0.1`，无法从公网访问，但在这种情况下你只有操作控制台才能开放公网访问从而使用控制台，于是你就得到了一个很神秘的死锁。而 1Panel 安装的 Koishi 会自动将监听地址改为 `0.0.0.0`，避免了这个问题。

这个问题已经被许多人反馈过，例如：[云服务器如何公网连接控制台 · Issue #1391 · koishijs/koishi](https://github.com/koishijs/koishi/issues/1391)

如果你坚持不使用 1Panel 安装，可以通过以下命令进行 SSH 端口转发：

```
ssh -N -L [本地端口]:localhost:5140 [用户名]@[服务器IP]
```

然后通过访问 `localhost:[本地端口]`进入控制台。

------

## NapCatQQ 的安装

NapCat 等 Onebot 框架同样禁止在 Koishi 官方群和论坛讨论，但这更多是由于腾讯的限制。

我们继续使用 1Panel 安装：

1. 进入“容器”选项卡，选择“创建容器” → “执行命令”；
2. 输入以下命令拉取拉取[一个民间镜像](https://github.com/NapNeko/NapCat-Docker)：

```
docker pull mlikiowa/napcat-docker
```

在“镜像”栏选择刚拉取的 `napcat-docker`，暴露端口填写 `6099`。

------

## NapCatQQ 的配置

配置前需先创建 Docker 网络：

1. 进入“容器” → “网络”，新建网络，子网填写 `10.11.0.0/16`，名称自定；
2. 回到“容器”页面，将 Koishi 和 NapCatQQ 的容器网络都切换为刚创建的网络。

安装完成后，访问 `[服务器IP]:6099`进入 NapCat 控制台。默认 token 是 `napcat`，建议修改以防他人登录。扫码登录机器人 QQ。

进入“网络”界面，新建 WebSocket 客户端：

- 名称：自定
- URL：`ws://1Panel-koishi-CRCm:5140/onebot`

保存即可。

------

## Koishi 的配置

访问 `[服务器IP]:5140`进入 Koishi 控制台。建议先启用 `auth`插件，设置登录密码，防止他人访问。

进入“插件市场”，若无法连接，需在“插件配置”中修改 `market`的源地址（界面提示清晰，不赘述）。

顺利进入后，安装 `adapter-onebot`插件。进入插件配置：

- 填写机器人 QQ 号；
- 若在 NapCat 中设置了 token，则填写，否则留空；
- 保存并启用。

至此，机器人基本配置完成，可进行测试。

------

## 我安装的其他插件推荐

以下是一些我安装并觉得好用的插件：

1. `adapter-onebot`
2. `bridge-qq-discord`
3. `github-opengraph`
4. `bing-daily-wallpaper`
5. `blockly-null`
6. `@summonhim/bili-parser`
7. `puppeteer`
8. `booru`
9. `booru-custom-api`
10. `pixiv-parse`
11. `pixiv-auth`

关于 `puppeteer`：建议在插件配置的 `args`中添加 `--disable-gpu`，否则 CPU 可能会被持续高占用。

------

希望这篇记录能帮到你。如有问题欢迎留言讨论。