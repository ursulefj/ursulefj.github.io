# 配置

https://hexo.io/zh-cn/docs/configuration

# 指令

https://hexo.io/zh-cn/docs/commands

## new

```shell
$ hexo new [layout] <title> 
```

新建一篇文章。如果没有设置 layout 的话，默认使用 `_config.yml` 中的 `default_layout` 参数代替。如果标题包含空格的话，请使用引号括起来。

```shell
$ hexo new "post title with whitespace"
```

参数	描述
-p, --path	自定义新文章的路径
-r, --replace	如果存在同名文章，将其替换
-s, --slug	文章的 Slug，作为新文章的文件名和发布后的 URL
