---
title: conda 管理python环境
date: 2025-07-08 16:21:10
tags:
---

conda 是一个开源的 python 包管理和环境管理工具，可以方便的创建、删除、切换python环境。通过它我们能实现多个版本的python共存，避免版本冲突。

conda安装不再赘述，下面我简单介绍它的使用。

## 配置源

conda 默认的源不太好用，我们可以配置国内的源，比如清华源。

我们可以通过以下命令查看当前的源配置：

``` shell
conda config --show-sources

# 上面的命令会输出如下内容
==> /Users/allen/.condarc <==
channels:
  - defaults
show_channel_urls: True

==> envvars <==
allow_softlinks: False
```

defaults 是默认源，我们通过编辑器打开 `/Users/allen/.condarc` 文件，将默认源注释掉，同时添加清华源和中科大源，如下：

```yml
channels:
  - https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/
  - https://mirrors.ustc.edu.cn/anaconda/pkgs/free/
  - https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
  # - defaults
show_channel_urls: true
```

添加源也可以通过以下命令：

```shell
# 添加清华源
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/

# 添加中科大源
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/
```

## 多环境创建与切换

查看当前的环境：

```shell
conda env list
```

创建新的环境：

``` shell
# conda create --name <环境名> python=<版本号>
# 例子：创建一个叫 py3.10 的环境，它的python版本是 3.10
conda create --name py3.10 python=3.10
```

激活环境：

```shell
# conda activate <环境名>
# 激活 py3.10 的环境
conda activate py3.10
```

在激活的环境中安装包：

```shell
# pip install <包名>
# 或者
# conda install <包名>
```

退出当前环境，回到之前的环境：

```shell
conda deactivate
```

删除环境：

```shell
# conda remove --name <环境名> --all
```