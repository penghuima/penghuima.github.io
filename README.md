# Welcome to PhM's Blog!

欢迎大家！

- [博客主题预览](#博客主题预览)
- [项目目录](#项目目录)
  - [主页](#主页)
  - [项目目录结构](#项目目录结构)

* [Fork修改指南](#Fork修改指南)
  * [修改全局配置文件](#修改全局配置文件)
  * [如何发布一篇博客](#如何发布一篇博客)
  * [修改网站图标](#修改网站图标)
  * [修改评论模块](#修改评论模块)

## 博客主题预览

博客网站的主要展示界面如下，分为主页、归档、分类、标签、收藏、关于、搜索等几个模块，大家可以进入我的 [个人博客](https://penghuima.github.io/) 点击了解

<img align="center" src="https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBedimage-20210615151017868.png" width="75%">

博客网站页尾界面如下，主要是一些网站浏览统计量显示，和主题作者信息

<img align="center" src="https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBedimage-20210615154745099.png" width="75%">

> 浩阳大神牛逼,也有其它很多好的 Jekyll 主题模板，但我比较倾向于这一款的布局设计

## 项目目录

### 主页

主页的源码在项目文件夹 index.html 里，如果想更改相关主页页面布局在该文件夹里更改

### 项目目录结构

主要项目文件夹及其作用描述，全部介绍可查看 [Jekyll](https://www.jekyll.com.cn/docs/structure/) 官方目录结构

- _posts： 存放我们原始博文内容的地方
- _includes： 你可以加载文件夹里的 html 内容到你的布局或者文章中以方便重用
- _layouts： 布局是包裹在博文内容外部的模板，布局模板可以在YAML头信息中根据不同文章进行选择
- _site： 一旦 Jekyll 转换完成，就会将生成的页面存放在这里
- _config.yml： 全局配置文件，基本上要将其中与个人信息相关的部分替换成自己的

## Fork修改指南

### 修改全局配置文件 

可以部署本地项目运行环境，然后对照着改，全局配置文件 _config.yml 里的注释也很清楚 

 **网页浏览量统计模块**，换成自己的 id  ，网上教程很多，这里随便给大家分享一个  [访问统计](https://blog.csdn.net/weixin_33782386/article/details/94516630)

<img align="center" src="https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBedimage-20210615220234426.png" width="72%">

### 如何发布一篇博客

对本博客主题最快的掌握就是懂得如何发布一篇自己写的博文

- 在撰写 markdown 格式博文时，先在文档开头以 yaml 格式下备注好 布局模板、博文标题、博文分类、博文标题，多说一句，博客主页上显示的**分类**板块和**标签**板块的数据都是通过这种博文开头定义输入的数据。

<img align="center" src="https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBedimage-20210615222548774.png" width="72%">

- 将博文目录调出来，这里的目录对应浏览博文时右边框显示的目录内容(浏览博客全部内容时，自己体会)，下面两行代码就可以调出目录

```markdown
* content
{:toc}
```

- 摘要分隔符，这里的摘要分隔符是在 _config.yml 文件里自定义的，主要作用是用于在主页上定义每篇博文要展示的内容摘要（自己体会）

```markdown
<!--more-->
```

- 添加到本地工作区 _posts 里，然后使用 Git 上传到 GitHub 更新博客，不同使用习惯，命令行可能不同， `git commit -a `  只对文件的删除、修改状态可追踪，但新创建的文件却无法跟踪，需要 `add` 来添加到缓冲区

```sh
$ git add .
$ git commit -a
$ git push origin master
```

- 博文文档的命名，命名格式如下，注意两点，不要使用中文和命名不要太长，不要超过50个字符（<50），这是由于 Gitalk 评论模块造成的 

```sh
2016-10-20-how-to-use-markdown.md
```

至此你就可以发布一篇自己写的博客了，发布之后你会发现博客**主页显示**，以及**分类板块**、**标签板块**等都会增加你新加入的相关内容。

**注意：**在将博客 push 到仓库之前，最好先在本地环境测试一下，避免重复修改提交

```shell
jekyll s
```

### 修改网站图标

网站 ico 图标主要由 favicon.ico 文件决定的，你可以先用 [favicon](https://tool.lu/favicon/) 制作工具来生成 16*16 大小的图标，在本地工作区里换掉原图标文件，但是会出现一种情况，那就是图标没有变化，而且还变成了原图标的样子，这是由于缓存造成的，可以将 _site 里的图标文件删掉，然后浏览器刷新等待即可（多给它些时间）

### 修改评论模块

评论模块有很多可以选择如 disqus、duoshuo、gitment 、gitalk 等，本博客选择的是最近比较流行的 Gitalk

相关配置文件路径 /_includes/gitalk.html ，主要更改的内容就是 clientID、clientSecret、repo、owner、admin 博客 Gitalk 模块网上有很多教程，这里随意给大家列一个 [相关教程](https://blog.csdn.net/zy13651953784/article/details/104813021)

```ruby
var gitalk = new Gitalk({
  clientID: 'aa71666defbb191d5311',
  clientSecret: '5f859a84c16713f31688c23cf61b43ad17836a3a',
  repo: 'penghuima.github.io',
  owner: 'penghuima',
  admin: ['penghuima'],
  language :'zh-CN',
  id: location.pathname,
```

评论模板修改之后，要授权登录自己的 GitHub账号，为每一篇博文创建一个 Issue 面板让大家留言，你会发现自己的仓库里有很多自动生成的 Issue
