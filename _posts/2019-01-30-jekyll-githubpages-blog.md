---
layout: post
title: "使用Jekyll + Github Pages搭建个人博客"
subtitle: "每个程序员都应该有一个自己的个人博客"
date: 2019-01-30 18:28:16
author: "Echcz"
header-style: "text"
catalog: true
tags:
  - "Jekyll"
---
{% raw %}
## 相关概念

### Jekyll 介绍

Jekyll 是一个静态站点生产工具。它有一个模版目录，其中包含原始文本格式的文档，通过一个转换器（如 Markdown）和 Liquid 渲染器转化成一个完整的可发布的静态网站，你可以发布在任何你喜爱的服务器上。
Jekyll 具有以下特点：

* 简单： 不再需要数据库，不需要开发评论功能，不需要不断的更新版本——只用关心你的博客内容。
* 静态： 使用Markdown（或 Textile）、Liquid 和 HTML & CSS 构建可发布的静态网站。
* 博客支持： 支持自定义地址、博客分类、页面、文章以及自定义的布局设计。

### Github Pages 介绍

Github Pages 是面向用户、组织和项目开放的公共静态页面搭建托管服务，站点可以被免费托管在 Github 上，你可以选择使用 Github Pages 默认提供的域名 github.io 或者自定义域名来发布站点。Github Pages 支持自动利用 Jekyll 生成站点，也同样支持纯 HTML 文档。

使用Github Pages搭建个人站点有以下优点：

* 完全免费，其中服务器、流量、域名什么的都不用管，完全零费用搭建。
* 写博客就是提交代码，让写作和编程的体验保持一致
* 支持绑定自己的域名

当然也是有缺点的：

* 不支持动态内容，站点必须都是静态网页，一般会使用 Jekyll 来构建内容
* 仓库空间不大于1G
* 每个月的流量不超过100G
* 每小时更新不超过10次

## 项目搭建

### 安装 Jekyll

1. 在安装 Jekyll 之前需要确保你的系统里已安装如下软件：
* [Ruby](http://www.ruby-lang.org/en/downloads/)（including development headers, Jekyll 2 需要 v1.9.3 及以上版本，Jekyll 3 需要 v2 及以上版本）
* [RubyGems](http://rubygems.org/pages/download)
* [NodeJS](http://nodejs.org/), 或其他 JavaScript 运行环境（Jekyll 2 或更早版本需要 CoffeeScript 支持）
* [Python 2.7](https://www.python.org/downloads/)（Jekyll 2 或更早版本）

安装好以上环境后，执行以下命令安装 Jekyll：

``` shell
~ $ gem install jekyll
```

### 使用 Jekyll 搭建个人博客

一个 Jekyll 站点目录结构大致如下：

``` shell
.
├── _config.yml
├── _drafts
|   ├── begin-with-the-crazy-ideas.textile
|   └── on-simplicity-in-technology.markdown
├── _includes
|   ├── footer.html
|   └── header.html
├── _layouts
|   ├── default.html
|   └── post.html
├── _postsery-programmer-sh
|   ├── 2007-10-29-why-evould-play-nethack.textile
|   └── 2009-04-26-barcamp-boston-4-roundup.textile
├── _site
└── index.html
├── .jekyll-metadata
```

目录结构说明如下：

| 目录/文件 | 说明 |
|:--:|:-- |
|  _config.yml  | 保存配置数据，全局自定义常量也可以写在这里，通过 `site.val_name` 访问 |
|  _drafts  | 草稿，是未发布的文章。这些文件的格式中都没有 title.MARKUP 数据 |
| _includes | 包含部分，可以加载这些包含部分到你的布局或者文章中以方便重用。可以用这个标签  {% include file.ext %} 来把文件 _includes/file.ext 包含进来 |
| _layouts | 布局，是包裹在文章外部的模板。布局可以在 YAML 头信息中根据不同文章进行选择。 这将在下一个部分进行介绍。标签  {{ content }} 可以将content插入页面中 |
| _posts | 这里放的就是你的文章了。文件格式很重要，必须要符合: YEAR-MONTH-DAY-title.MARKUP。永久链接可以在文章中自己定制，但是数据和标记语言都是根据文件名来确定的 |
| _data | 格式化好的网站数据应放在这里。jekyll 的引擎会自动加载在该目录下所有的 yaml 文件（后缀是 .yml, .yaml, .json 或者 .csv ）。这些文件可以经由 ｀site.data｀ 访问。如果有一个 members.yml 文件在该目录下，你就可以通过 site.data.members 获取该文件的内容 |
| _site | 一旦 Jekyll 完成转换，就会将生成的页面放在这里（默认）。最好将这个目录放进你的 .gitignore 文件中 |
| .jekyll-metadata | 该文件帮助 Jekyll 跟踪哪些文件从上次建立站点开始到现在没有被修改，哪些文件需要在下一次站点建立时重新生成。该文件不会被包含在生成的站点中。将它加入到你的 .gitignore 文件可能是一个好注意 |
| index.html and other HTML, Markdown, Textile files | 如果这些文件中包含 YAML 头信息 部分，Jekyll 就会自动将它们进行转换。当然，其他的如 .html, .markdown, .md, 或者 .textile 等在你的站点根目录下或者不是以上提到的目录中的文件也会被转换 |
| Other Files/Folders | 其他一些未被提及的目录和文件如  css 还有 images 文件夹， favicon.ico 等文件都将被完全拷贝到生成的 site 中 |

1. 执行以下命令，使用 Jekyll 快速构建名为myblog的简单的 Jekyll 项目：

``` shell
~ $ jekyll new myblog
```

也可以到 [jekyllthemes.org](http://jekyllthemes.org/) 或者 [Github](https://github.com/) 中查找/下载 Jekyll 项目。

2. 修改_config.yul配置文件与站点内容，使之符合自己的要求

3. 进入你的 Jekyll 项目根目录，执行以下命令，在本地去年运行 Jekyll 站点：

``` shell
myblog $ jekyll serve
```

Jekyll会自己监测项目文件，自动更新站点，方便调试；如果不想自动监测，可以添加参数：`--no-watch`

## 发布个人博客到 Github Pages

1. 将自己的项目发布到 Github 上
2. 更改项目名为: `your_name`.github.io
3. 更改你的 Github Pages > Source 为项目的产品级版本分支
4. 如需使用自己的域名，请设置你的 Github Pages > Custom domain为你的域名

## 参考链接

* 关于 Jekyll 的说明与文档可以访问 [Jekyll](https://jekyllrb.com/) 或 [JekyllCN](https://jekyllcn.com/)
* 关于 Github Page 的说明与文档可以访问 [Github Pages](https://pages.github.com/)
* Jekyll 使用了 Liquid 模版引擎，关于 Liquid 的说明与文档可以访问 [Liquid Wiki](https://github.com/Shopify/liquid/wiki) 或 [简书-一曲广陵散-Liquid语法](https://www.jianshu.com/p/4224b8ea0ec0)
{% endraw %}