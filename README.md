# Allen 的个人博客

hexo 命令[链接](https://hexo.io/docs/commands)

## 创建文档并发布

创建有一个文档：

```shell
hexo new [layout] <title>
```

添加完文档，直接push到github自动构建部署。

## 创建草稿

可以先创建一个草稿，然后再发布。

```shell
hexo new draft <title>
```

下面将草稿发布为正式文章：

```shell
hexo publish <title>
```