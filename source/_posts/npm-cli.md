---
title: npm常用命令
date: 2025-07-28 21:43:55
tags: npm
---

## 查看node的安装位置

查看当前node安装目录，当你发现 vscode 和 shell里面 node 版本不一致的时候，你应该是使用了多种包管理工具安装了 node，例如nvm、brew、pnpm等。如何查看当前 terminal 内 node 的安装目录呢？可以使用下面的命令：

```shell
which node
```

## npm全局包

查看npm全局包所在目录，如果你使用 nvm，下面命令会输出 `/Users/allen/.nvm/versions/node/v20.9.0/lib/node_modules`

```shell
npm root -g
```

查看npm全局包有哪些

```shell
npm ls -g
```

仅查看全局包的一级目录

```shell
npm ls -g --depth 0 | grep packageName
npm list -g --depth 0 | grep packageName
```

## link的使用

### 创建symlink到全局包目录

进入到项目目录，执行

```shell
npm link
```

这个命令会在会在全局，`/Users/allen/.nvm/versions/node/v20.9.0/lib/node_modules` 目录下生成一个软链接，指向当前的项目，如果当前的项目 package.json 中有 bin 配置，如下所示：

```json
{
  "name": "bai-store",
  "version": "0.0.1",
  "private": true,
  "description": "takumi练习",
  "license": "MIT",
  "bin": {
    "bai": "./cli.mjs"
  }
}
```

它还会在全局注册这个命令。你会在 `/Users/allen/.nvm/versions/node/v20.9.0/bin` 内看到一个 `bai` 的软链接，指向当前项目的 `cli.mjs` 文件。

cli.mjs 文件内容如下：

```js
#!/usr/bin/env node

// 检查Node版本，确保只有Node 18及以上版本才能运行
const nodeVersion = process.versions.node;
const majorVersion = parseInt(nodeVersion.split('.')[0], 10);

console.log(`\x1b[31m当前Node版本: v${nodeVersion}\x1b[0m`);

if (majorVersion < 18) {
  console.error('\x1b[31m错误: 当前Node版本过低\x1b[0m');
  console.error(`当前版本: v${nodeVersion}`);
  console.error('\x1b[33mKMI AI需要 Node.js 18 或更高版本才能运行\x1b[0m');
  console.error('请升级您的 Node.js 版本: https://nodejs.org/');
  process.exit(1);
}

```

### 建立链接

上面我们已经在全局目录下建立了软链接，现在我们进入到另外一个项目，执行如下命令：

```shell
npm link bai-store
```

这样，我们就不需要真的安装 bai-store 了，可以直接在项目里面 import bai-store 来使用。方便我们本地对 bai-store 的调试开发。

### 删除链接

下面这个方法可以删除链接，同时也可以用来删除全局包。

```shell
npm rm <package-name> -g
```

## 查看当前项目可执行的命令

```shell
npm run
```

## peerdependencies相关

列出某个包的 peerdependencies：

```shell
npm info "<pkg>@latest" peerDependencies
```

安装某个包的 peerdependencies：

```shell
npx install-peerdeps --dev <pkg>
```

## 查看包信息

```shell
npm view <package>              # 查看包的所有版本
npm home <package>              # 查看包的官网
npm repo <package>              # 查看包的仓库
npm docs <package>              # 查看包的文档
```

```shell
npm list                 # 列出当前仓库依赖的包
npm info <package>       # 检查当前仓库下某个包的信息
```

## 包安装

```shell
npm install                   # 安装包
npm install <package>@latest  # 安装 latest stable release
npm install <package>@beta    # 安装 latest beta release
npm install --production      # 只安裝 dependencies，不會安裝 devDependencies
npm install --dev        # 只會安裝 devDependencies，不會安裝 dependencies
```

## 包卸载

```shell
npm uninstall <packageName>   # 卸载包
npm prune                     # 移除没用到的包
npm dedupe                    # 移除重复的包
```

## 包更新

```shell
npm update 
npm update <packageName> 
```

## 参考

https://pjchender.dev/npm/npm-cli-and-package-json/