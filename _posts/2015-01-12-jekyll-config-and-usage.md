---
layout: post
title:  "Jekyll使用与配置"
keyword: "jekyll,github,blog"
date:   2015-01-12
categories: ass
tags:	jekyll github blog
---

Jekyll 是一个简单的博客形态的静态站点生产机器。它有一个模版目录，其中包含原始文本格式的文档，通过 Markdown （或者 Textile） 以及 Liquid 转化成一个完整的可发布的静态网站，你可以发布在任何你喜爱的服务器上。Jekyll 也可以运行在 GitHub Page 上，也就是说，你可以使用 GitHub 的服务来搭建你的项目页面、博客或者网站，而且是完全免费的。

一些程序员开始在github网站上搭建blog。他们既拥有绝对管理权，又可以享受github带来的便利。不管何时何地，只要向主机提交commit，就能发布新文章。这一切还是免费的，github提供无限流量，世界各地（除了某些地方）都有理想的访问速度。


###先让我们从Jekyll开始###

安装Ruby环境与gem工具: Ruby(>=1.9.3)与gem

{% highlight bash %}
~ $ gem install jekyll
~ $ jekyll new blog
~ $ cd blog
~/blog $ jekyll serve
{% endhighlight %}


###Jekyll目录结构###

*_config.yml*

保存配置数据。很多配置选项都可以直接在命令行中进行设置，但是如果你把那些配置写在这儿，你就不用非要去记住那些命令了。

*_drafts*

drafts 是未发布的文章。这些文件的格式中都没有 title.MARKUP 数据。学习如何使用 drafts.

*_includes*

你可以加载这些包含部分到你的布局或者文章中以方便重用。可以用这个标签  {&#37; include file.ext &#37;} 来把文件 _includes/file.ext 包含进来。

*_layouts*

layouts 是包裹在文章外部的模板。布局可以在 YAML 头信息中根据不同文章进行选择。 这将在下一个部分进行介绍。标签 {{ content }} 可以将content插入页面中。

*_posts*

这里放的就是你的文章了。文件格式很重要，必须要符合: YEAR-MONTH-DAY-title.MARKUP。 The permalinks 可以在文章中自己定制，但是数据和标记语言都是根据文件名来确定的。

*_site*

一旦 Jekyll 完成转换，就会将生成的页面放在这里（默认）。最好将这个目录放进你的 .gitignore 文件中。

*index.html和其他HTML, Markdown, Textile等文件*

如果这些文件中包含 YAML 头信息 部分，Jekyll 就会自动将它们进行转换。当然，其他的如 .html, .markdown, .md, 或者 .textile 等在你的站点根目录下或者不是以上提到的目录中的文件也会被转换。

*其他文件*

其他一些未被提及的目录和文件如  css 还有 images 文件夹， favicon.ico 等文件都将被完全拷贝到生成的 site 中。这里有一些使用 Jekyll 的站点，如果你感兴趣就来看看吧。


###全局变量设置###

编辑_config.yml
{% highlight yaml %}
url: ""
keywords: dulio,blog
# Build settings
paginate: 15
paginate_path: "/p/:num"
{% endhighlight %}


###托管Github Pages － 项目主页面###
新建Repo，如blog

把jekyll源码push到gh-pages分支（分支名必须为gh-pages）

{% highlight bash %}
git init
git add -f *
git checkout -b gh-pages
git commit -m 'first commit'
git remote add https://github.com/dulio/blog.git
git push -u origin gh-pages
{% endhighlight %}

访问dulio.github.io/blog (用户名.github.io/项目名)


###托管Github Pages - 用户主页面###
创建名为（用户名.github.io）的项目

推送jekyll代码到master分支下


###托管Github Pages - 用自己的域名###
项目目录下创建CNAME文件

CNAME文件内容填写域名名称

###托管到自有主机###
给项目添加Webhooks, Github →项目 → Settings →  Webhooks

主机添加hook脚本

在github push后执行jekyll build操作

配置web server，Jekyll项目下_sites目录
