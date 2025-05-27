---
title: npm 发布注意事项
date: 2025-05-27 21:07:23
tags:
---


偶尔发布npm包时，会忘记流程，因此做个简单的记录，方便快速回忆整个操作。

## 1. 注册npm账号

如果你要发布到npm，需要先注册一个账号（有账号的忽略这步骤）。链接：[https://www.npmjs.com/](https://www.npmjs.com/)。

如果你要发布到公司内私有的包管理仓库，就根据公司内部的方式注册账号。

## 2. 检查npm registry

在发布前，需要检查npm registry是否是你要发布的地址。命令行进入项目目录，运行以下命令查看：

```shell
# 在需要发布包的项目路径内运行下面命令
npm config get registry
```

如果结果不是你想要发布的地址，可以通过以下命令修改：

```shell
npm config set registry=https://www.npmjs.com/
```

如果你不想每次发布都要修改一遍，可以创建一个`.npmrc`文件，内容如下：

```text
registry=https://www.npmjs.com/
```

设置了这个文件后，每次发布和安装包时，npm都会自动使用这个地址。

## 3. 命令行登录npm账号

使用`npm login`命令登录npm账号，会提示输入账号和密码。


```shell
# 登陆
npm login
```

如果公司内私有包管理仓库提供了web的登录方式，可以用以下命令登录：

```shell
npm login --auth-type=web
```

## 4. 版本管理

发布包时，需要指定版本号。npm提供快捷的版本修改方式，可以使用`npm version`命令。

```shell
npm version [<newversion> | major | minor | patch | premajor | preminor | prepatch | prerelease [--preid=<prerelease-id>] | from-git]
```

例如你只是修复了bug，可以使用`patch`修改版本号：

```shell
npm version patch
```

它没有任何参数时，会自动修改`package.json`中的`version`字段，依据`semver`规范，`patch`会自动加1。例如 `1.0.1`会更改为`1.0.2`。

更新完版本后，你需要通过git将变更提交，同时使用`tag`命令打上版本标签，方便以后根据包版本定位到对应的代码版本：

```shell
# 把修改了版本号的package.json 添加到git缓存区
git add .
# 提交修改
git commit -m 'update version'
# 打上版本标签
git tag -a 1.0.2 -m 'version 1.0.2'
```

## 5. 发布

万事具备，可以发布了。

```shell
npm publish
```