---
title: 使用 pnpm 替代 nvm 进行 Node 版本管理
date: 2025-04-24 19:20:05
tags: pnpm
---

## 背景

日常工作中，我们经常需要在不同的项目中使用不同的 Node.js 版本。pnpm 提供了一个非常方便的命令 `pnpm env use <node-version>` 来切换 Node.js 版本。可以帮助我们轻松地在不同的项目中切换 Node.js 版本，这样我们就不需要单独下载 nvm 了。

## 安装 pnpm

有个注意事项，你通过 `npm i -g pnpm` 的形式安装是没有 `pnpm env use` 命令的。需要通过以下几种方式安装才行（如果你已经安装了 nvm，可以参考这个[链接](https://github.com/nvm-sh/nvm/issues/298)删除 nvm，然后再安装 pnpm）：

### 在 Windows 上安装 pnpm

使用 PowerShell：

```shell
Invoke-WebRequest https://get.pnpm.io/install.ps1 -UseBasicParsing | Invoke-Expression
```

[原官网文档地址](https://www.pnpm.cn/installation#%E5%9C%A8-windows-%E4%B8%8A)

### 在 POSIX 系统上

```shell
curl -fsSL https://get.pnpm.io/install.sh | sh -
```

如果你没有安装 curl，也可以使用 wget：

```shell
wget -qO- https://get.pnpm.io/install.sh | sh -
```

[原官网文档地址](https://www.pnpm.cn/installation#%E5%9C%A8-posix-%E7%B3%BB%E7%BB%9F%E4%B8%8A)

## 使用

安装完毕需要重启下命令行工具（或者用 `source ~/.bashrc` or `source ~/.zshrc` or `source ~/.bash_profile` 等命令使配置生效），然后执行 `pnpm -v` 就可以看到版本号了。

如果你要切换 node 到版本 22 可以执行：

```shell
pnpm env use --global 22

node -v # v22.15.0
```

切换 node 到版本 16 可以执行：

```shell
pnpm env use --global 16

node -v # v16.20.2
```

还有一个小技巧，如果某个项目安装命令需要 pnpm 版本为 8，node 版本为 16，那么可以执行：

```shell
pnpm env use --global 16

npx pnpm@8 install
```