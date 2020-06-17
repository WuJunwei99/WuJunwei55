---
layout: post
title: "使用github+jekyll搭建个人博客"
date: 2020-06-11 16:38:12 +0800
categories: technology
tags: git jekyll
img: https://www.z4a.net/images/2020/06/11/u20014715983734833766fm15gp0.md.png
---
个人网站的搭建（基于GitHub和Jekyll主题，以自己的博客为例）


## 关于github

gitHub是一个面向开源及私有软件项目的托管平台，因为只支持git 作为唯一的版本库格式进行托管，故名gitHub。

首先，github是一个版本管理器。当你在做一个项目时，你可以每做一次修改就保存一次版本，当你想回退到某一版本时，就可以很方便地回退了。

其次，github为开源项目的改进做了很大贡献。我们可以把一些开源的项目拿来自己用，也可以加以修改，这样一个开源的项目经过别人的无数次修改，就会变得更好。

再有就是，github使多人开发更加方便，做一个项目的时候，可以把项目的管理权限分发给一起做项目的小伙伴，然后大家就都可以编辑这个项目啦！

最近看到别的同学的博客，感觉很不错的样子，就想着建立一个属于自己的个人博客，通过搜索了很多资料，发现一个免费而又简单的搭建博客的方法。就是利用github下面的github pages+jekyll一个简单的免费的Blog生成工具。

## 注册并设置github

 注册一个github

地址: https://github.com/

输入账号、邮箱、密码,然后点击注册按钮.

之后需要在邮箱完成相关验证即可注册成功



## 挑选并fork主题

我们可以通过访问[jekyll的主题商店](http://jekyllthemes.org/)去挑选一个自己喜欢的主题，我用的是[jekyll-theme-mdui](http://jekyllthemes.org/),通过点击homepage可以访问到作者的此仓库，我们选择clone即可把这个仓库克隆到自己的远程库。

![](https://www.z4a.net/images/2020/06/11/theme_clone.md.jpg)


## 修改仓库name

转到Settings，然后修改项目名字，将那么修改为自己的github名字+github.io；例如我是wujunwei99，所以这里就是wujunwei99.github.io

![](https://www.z4a.net/images/2020/06/11/git_theme_settings.md.png)

接下来在地址栏输入：用户名.http://github.io我们就可以访问页面了。


## 修改相关配置

我们此时可以将该仓库拉取到本地进行相关的修改。

因为我选择的主题已经给出了较为详细的修改说明，您可以访问他的[中文说明](http://jekyllthemes.org/)也可通过我的叙述完成相关个性化设置。

1. config文件

编辑_config.yml，修改其为自己的信息即可。

title:站点名称，会显示在导航栏左侧。

author:作者。

注意：theme字段必须为theme: jekyll-theme-mdui，否则将无法使用本主题。

2. _data/menus.yml文件

读者可以修改相关的菜单栏为自己的菜单内容；

name：定义名称，名称会显示在导航栏中。

url：定义文件路径。

3. _data/category.yml文件

修该文件可以对category分类页面显示的分类卡片进行具体设置；

name：分类卡片的名称

img：分类卡片显示的背景图片

说明:如果未对此进行设置，则背景为默认的黑色

4. category文件

在创建相关的分类卡片之后，需要在category文件夹下创建相应的markdown文件，否则将无法实现点击的跳转功能；

文件名为：分类卡片的名称+.markdown

内容为：

    ---
    layout: category_content
    ---

5. _post文件

_post文件夹下存放了显示的具体文章；用户可以创建自己的markdown文件；

layout：要使用的布局，必须为post

title：文章标题

date：文章创建日期

categories：文章所属分类，可使用空格添加多个分类

tags：文章包含的标签，可使用空格分开多个标签

img：文章头部图像

describe（可选）：为文章添加一段简介，简介将出现在文章卡片的下方；若不添加，则将默认使用文章开头的第一句话作为简介


## 上传文件

修改结束后记得保存，之后就可以用git命令将修改后的文件上传至远程仓库，关于git命令不熟悉的可以参考我的另一篇文章：[git命令行操作](https://wujunwei99.github.io/notes/2020/06/08/git-command.html)。

等待一会之后，刷新博客主页，即可看到我们自己的博客啦！

![](https://www.z4a.net/images/2020/06/11/blog_01.md.jpg)


## 自定义博客（补充）

1. 新建一个仓库

输入仓库名，点击创建(仓库名最好和用户名一致，因为我已经创建过了，所以会提示错误)

![](https://www.z4a.net/images/2020/06/11/git_resig.md.jpg)

2. 选择主题

仓库创建成功后，点击settings进入设置界面拉到末尾，找到choose theme选择一个自己喜欢的主题，再次返回顶部，接下来在地址栏输入：用户名.http://github.io我们就可以访问页面了。
