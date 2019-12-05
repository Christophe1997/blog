+++
title = "使用hugo生成博客"
author = ["christophe"]
date = 2019-12-05T20:29:00+08:00
lastmod = 2019-12-05T20:31:52+08:00
categories = ["杂项"]
tags = ["Hugo", "Org-mode"]
draft = false
+++

之前一直有使用[Hexo](https://hexo.io/zh-cn/index.html)来生成静态博客，如今将博客迁移到了[Hugo](https://gohugo.io/)下。两种工具总体而言各有优势，个人此次转移到hugo的主要原因大概是希望能够重拾写博客的习惯，本文主要介绍了使用Hugo的大致流程。首先给出个人博客的文件结构：

```text
Root               # blog根目录
|
+-- archetypes
+-- content
 |-- posts         # 博客目录
 |-- _index.md     # 主页
+-- data
+-- layouts
+-- ox-hugo        # org文件位置
 |-- messy.org     # category为杂项的org文件
+-- public         # submodule
+-- resource
+-- static
+-- themes
 |-- xxx           # 主题文件夹, submodule
+-- .git
+-- config.toml
+-- .dir-locals.el # 自动将org文件转为markdown配置文件
+-- .gitmodules
```

其中只有 `ox-hugo` 为自定义文件夹, `.dir-locals` 为ox-hugo自动转换的配置文件，其他都是自动生成的。接下来个人就hugo生成博客关键部分展开介绍下。


## 主题的选择 {#主题的选择}

首先搭建博客面对的问题就是主题，如果你非常擅长网页设计可以自己DIY一款主题。大多数还是会选择[已有的主题](https://themes.gohugo.io/)，我们在选择主题时，除了最重要的UI外，还应当考虑以下的因素：

-   是否支持写作的基本功能，如公式的显示（通常是MathJax）；
-   是否支持博客网站的基本功能，如分类，标签，分享，评论等。

尤其是评论这块，个人之前用的disqus, 但好像在国内的访问存在一些问题，目前比较推荐的方案是使用git issues来实现评论功能，如gitment。像我这样的懒人当然是选择不支持评论功能啦：）


## 博客的托管 {#博客的托管}

比较推荐的方案是将整个blog作为一个git项目，并且将 `public` 和 `themes/xxx` 作为该项目的submodule。其中 `public` 对应于你的要作为[Github pages](https://help.github.com/articles/user-organization-and-project-pages/#user--organization-pages)的
git项目地址，通常建议是 `<username>.github.io` 这个项目，这样Github会自动将该项目作为 `http://<username>.github.io` 页面的文件，完整的将博客自动托管到Github的流程可以参考hugo上的[教程](https://gohugo.io/hosting-and-deployment/hosting-on-github/)；而 `themes/xxx` 则是对应于主题的项目地址。这样使用submodule的一个好处是可以保证主题的同步以及public的自动托管与发布。


## 使用org-mode写作 {#使用org-mode写作}

如果你对于emacs的[org-mode](https://orgmode.org/)向往已久并且希望使用其来写博客的话，一个推荐的方案是使用[ox-hugo](https://ox-hugo.scripter.co/)自动地将你用org-mode写的内容转为markdown，虽然hugo本身也有org-mode
的引擎，但好像支持的不是很好。在这样的工作模式下建议的方案是将所有相同category的且不是很长的博客都集中到一个org文件中，文件的开头配置大致如下：

```text
#+STARTUP: content
#+hugo_base_dir: ../       # blog根目录
#+hugo_section: ../content/posts    # ox-hugo生成md文件的目录
#+filetags: @<category>    # 该org文件中所有文章的分类
#+hugo_auto_set_lastmod: t
#+hugo_locale: zh-cn
```

每次新建一个博客时就在该文件中加入一个todo事项，如本博客的开头为：

```text
** TODO 使用hugo生成博客
 :PROPERTIES:
 :EXPORT_FILE_NAME: 使用hugo生成博客
 :END:
```

这些可以利用[org-capture](https://orgmode.org/manual/Capture.html)自动生成的，具体可以参考ox-hugo上的[文档](https://ox-hugo.scripter.co/doc/org-capture-setup/)。另外建议将文件加入到[org-agenda](https://orgmode.org/manual/Agenda-views.html)文件列表中，这样可以快速定位未完成的博客（即TODO状态下的博客）。完成一篇博客时只需要将博客标记为DOWN即可。ox-hugo还会将org的tag转为相应的分类和标签，其中 `@` 开头的为分类, ox-hugo的自动生成可以参考相应的[文档](https://ox-hugo.scripter.co/doc/auto-export-on-saving/)。

以上就是我在使用hugo生成博客时遇到的一些主要问题。如果你有什么问题，欢迎[邮件](mailto:hey%5Fchristophe@outlook.com)我。
