## 模版说明

* 开始
	* [环境要求](#environment)
	* [开始](#get-started)
	* [写一篇博文](#write-posts)
* 各组成部分
	* [侧边栏](#sidebar)
	* [mini-about-me](#mini-about-me)
	* [标签云](#featured-tags)
	* [好友链接](#friends)
	* [HTML5演示文档布局](#keynote-layout)
* 评论与 Google/Baidu Analytics
	* [评论](#comment)
	* [网站分析](#analytics) 
* 高级部分
	* [自定义](#customization)
	* [标题底图](#header-image)
	* [多语言支持](#multilingual)

### Environment

如果你安装了jekyll，那你只需要在命令行输入`jekyll serve`就能在本地浏览器预览主题。你还可以输入`jekyll serve --watch`，这样可以边修改边自动运行修改后的文件。

经 [@BrucZhaoR](https://github.com/BruceZhaoR) 的测试，好像两个命令都是可以的自动运行修改后的文件的，刷新后可以实时预览。官方文件是建议安装bundler，这样你在本地的效果就跟在github上面是一样的。详情请见这里：https://help.github.com/articles/using-jekyll-with-pages/#installing-jekyll

### Get Started

你可以通用修改 `_config.yml`文件来轻松的开始搭建自己的博客。

Jekyll官方网站还有很多的参数可以调，比如设置文章的链接形式...网址在这里：[Jekyll - Official Site](http://jekyllrb.com/) 中文版的在这里：[Jekyll中文](http://jekyllcn.com/).

### write-posts

要发表的文章一般以markdown的格式放在这里`_posts/`，你只要看看这篇模板里的文章你就立刻明白该如何设置。

yaml头格式如下:

``` yaml
---
layout: post # 可选post, keynote(幻灯片模式)
title: "" #文章标题
subtitle: "" #文章副标题
date: 2006-01-02 15:04:05 #文章发表时间
author: "" #作者
header-img: "img/..." #文章头图路径
header-style: "" #头部类型，可选："", "text"
catalog: true #是否显示目录
multilingual: true #是否多语言
mathjax： true #是否开启Mathjax
previous: #上一篇文章
  url: ""
  title: ""
next: #下一篇文章
  url: ""
  title: ""
tags: #文章标签
  - ""
  - ""
---
```

### SideBar

设置是在 `_config.yml`文件里面的`Sidebar settings`那块。

``` yaml
# Sidebar settings
sidebar: true  #添加侧边栏
sidebar-about-description: "简单的描述一下你自己"
sidebar-avatar: /img/avatar-bg.jpg     #你的大头贴，请使用绝对地址.
```

侧边栏是响应式布局的，当屏幕尺寸小于992px的时候，侧边栏就会移动到底部。具体请见bootstrap栅格系统 <http://v3.bootcss.com/css/>

### Mini About Me

Mini-About-Me 这个模块将在你的头像下面，展示你所有的社交账号。这个也是响应式布局，当屏幕变小时候，会将其移动到页面底部，只不过会稍微有点小变化，具体请看代码。

### Featured Tags

标签系统使用了这个网站 [Medium](http://medium.com) 的标签云。
这个模块现在是独立的，可以呈现在所有页面，包括主页和发表的每一篇文章标题的头上。

``` yaml
# Featured Tags
featured-tags: true  
featured-condition-size: 1     # A tag will be featured if the size of it is more than this condition value
```

唯一需要注意的是`featured-condition-size`: 当达到这个值的标签才什成为featured tag
 
内部有一个条件模板 `{% if tag[1].size > {{site.featured-condition-size}} %}` 是用来做筛选过滤的.

### Friends

友情链接部分。这会在全部页面显示。

设置是在 `_config.yml`文件里面的`Friends`那块，自己加吧。

``` yaml
# Friends
friends: [
    {
        title: "Foo Blog",
        href: "http://foo.github.io/"
    },
    {
        title: "Bar Blog",
        href: "http://bar.github.io"
    }
]
```

### Keynote Layout

这部分是用于占用html格式的幻灯片的，一般用到的是 Reveal.js, Impress.js, Slides, Prezi 等等。

其主要原理是添加一个 `iframe`，在里面加入外部链接。你可以直接写到头文件里面去，详情请见下面的yaml头文件的写法。

``` yaml
---
layout:     keynote
iframe:     "http://huangxuan.me/js-module-7day/"
---
```

iframe在不同的设备中，将会自动的调整大小。保留内边距是为了让手机用户可以向下滑动，以及添加更多的内容。

### Comment

博客不仅的多说[Duoshuo](http://duoshuo.com)评论系统，也支持disqus[Disqus](http://disqus.com)评论系统。

disqus国际比较流行，界面也很大气、简介，如果有人评论，还能实时通知，直接回复通知的邮件就行了。缺点是评论必须要去注册一个disqus账号，分享一般只有Facebook和Twitter，另外在墙内加载速度略慢了一点。想要知道长啥样，可以看以前的版本点[这里](http://brucezhaor.github.io/about.html) 最下面就可以看到。

多说国内主流社交软件都有分享按钮，登陆方便，比较好管理，就是界面丑了一点。当然你是可以自定义界面的css的，详情请看多说开发者文档。

**首先**，你需要去注册一个账号，不管是disqus还是多说的。

**其次**，你只需要在下面的yaml头文件中设置一下就可以了。

``` yaml
duoshuo_username: _你的用户名_
# 或者
disqus_username: _你的用户名_
```

**最后**多说是支持分享的，如果你不想分享，请这样设置：`duoshuo_share: false`。你可以同时使用两个评论系统，不过个人感觉怪怪的。

### Analytics

网站分析，现在支持百度统计和Google Analytics。需要去官方网站注册一下，然后将返回的code贴在下面：

``` yaml
# Baidu Analytics
ba_track_id: xxxxxxxx

# Google Analytics
ga_track_id: 'UA-xxxxxxx-xx'            # Format: UA-xxxxxx-xx
ga_domain: yourname.github.io
```

### Customization

如果你喜欢折腾，你可以去自定义这个模板的code，[Grunt](gruntjs.com)的环境已经搭好了。

There are a number of tasks it performs like minification of the JavaScript, compiling of the LESS files, adding banners to keep the Apache 2.0 license intact, and watching for changes. Run the grunt default task by entering `grunt` into your command line which will build the files. You can use `grunt watch` if you are working on the JavaScript or the LESS.

**Try to understand code in `_include/` and `_layouts/`, then you can modify Jekyll [Liquid](https://github.com/Shopify/liquid/wiki) template directly to do more creative customization.**

### Header Image

标题底图是可以自己选的，看看几篇示例post你就知道如何设置了。
但是需要注意的是本模板的标题是**白色**的，所以背景色要设置为**灰色**或者**黑色**，总之深色系就对了。

### multilingual

如需自己的文章支持多语言，请在post的yaml头信息添加：

``` yaml
multilingual: true
```

并将中文内容与英文内容分别放入对应容器中：

``` html
<!-- Chinese Version -->
<div class="zh post-container">
<-- zh_content -->
</div>

<!-- English Version -->
<div class="en post-container">
<!-- en_content -->
</div>
```

可以将中文与英文的内容分别添加到_includes文件夹中，然后在post页面引入：

``` html
<!-- Chinese Version -->
<div class="zh post-container">
    {% capture article_zh %}{% include posts/2019-01-01-article/zh.md %}{% endcapture %}
    {{ article_zh | markdownify }}
</div>

<!-- English Version -->
<div class="en post-container">
    {% capture article_en %}{% include posts/2019-0101-article/en.md %}{% endcapture %}
    {{ article_en | markdownify }}
</div>
```

## 发布到 Github Pages

1. Fork本仓库。
2. 重命名项目：点击 Settings 按钮打开设置页面，重命名项目名称为：{github_username}.github.io。
3. 配置`_config.yml`与修改项目内容
4. 设置GitHub Pages：点击 Settings 按钮打开设置页面，页面往下拉到 GitHub Pages 相关设置，在 Source 下面的复选框中选择 master branch ，然后点击旁边的 Save 按钮保存设置。
5. 如需使用自己的域名，请设置你的Github Pages > Custom domain为你的域名
6. 提交post以上传自己的文章
7. Enjoy!!!

## 发布到 Gitee Pages

1. Fork本仓库。
2. 配置`_config.yml`与修改项目内容
3. 设置Gitee Pages：点击 Service - Gitee Pages 进入Gitee Pages配置页面，在Choose brance中选择master，然后点击下面的Update按钮进行设置。
4. 提交post以上传自己的文章
5. Enjoy!!!

## 致谢

1. 这个模板是从这里[Hupro/huxpro.github.io](https://github.com/Huxpro/huxpro.github.io)  fork 的。感谢！
2. 本文档主要内容取自：https://github.com/Huxpro/huxpro.github.io/blob/master/README.md ，已由[@BrucZhaoR](https://github.com/BruceZhaoR)翻译为中文，感谢。
3. 感谢 Jekyll、Github Pages、Gitee Pages 和 Bootstrap!