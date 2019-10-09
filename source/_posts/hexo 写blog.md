---
title: hexo 写blog
date: 2016-05-12 15:20:34
categories: hexo
tags: hexo
---
# hexo 写blog

# 写文章

```java
hexo new [layout] <title> 

```

> 也可以不用这样，直接把markdown文章放到 `source/_post` 文件夹下,可以在命令中指定文章的布局（layout），默认为 post，可以通过修改 `_config.yml` 中的 `default_layout` 参数来指定默认布局。

Hexo 默认以标题做为文件名称，但可编辑 new_post_name 参数来改变默认的文件名称，举例来说，设为` :year-:month-:day-:title.md` 可让您更方便的通过日期来管理文章。

# 添加标签 about 页面
以 about 标签页为例

```java
title: 标签
date: 2017-2-22 12:39:04
type: "about"
comments: true
---
```

# 调试和部署

```java
hexo g    //根据模板编译生成文件
hexo s    //本地启动服务

hexo deploy  //发布到github
```

生成文件位置：`.deploy_git/`

> 每次部署会把 `CNAME` 和 `README.md` 删掉，可以把这两个文件放到 `source` 文件夹下就可以了。

# 常见问题

`# 标题`  注意显示标题时，在 # 后面加个空格就可以了。


# 参考资料

<a href=" https://hexo.io/zh-cn/docs/" target="_blank">hexo中文文档</a>

<a href="http://www.ezlippi.com//blog/2016/02/jekyll-to-hexo.html" target="_blank">Jekyll迁移到Hexo搭建个人博客</a>

