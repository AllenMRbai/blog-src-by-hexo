---
title: 苹果M2芯片mvn无法安装node14
date: 2025-07-08 17:10:53
tags:
---

当你使用苹果的M芯片mvn命令安装node14的时候，会遇到以下问题：

```
Downloading and installing node v14.21.3...
Downloading https://nodejs.org/dist/v14.21.3/node-v14.21.3-darwin-arm64.tar.xz...
curl: (22) The requested URL returned error: 404
```

node14 并没有提供 `node-v14.21.3-darwin-arm64.tar.xz` 安装包，所以这里下载失败了。

可以通过 arch 命令修改当前计算机架构的名称，然后再使用 mvn 命令安装 node14就可以了

```shell
arch -x86_64 zsh
nvm install 14
```

为什么这样可以解决问题呢？其实我也不太清楚。