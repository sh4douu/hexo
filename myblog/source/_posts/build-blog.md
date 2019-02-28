---
title: 博客搭建记录 Node.js + Hexo + Yelee + Git Pages
date: 2019-02-21 14:32:04
tags: 
	- 技术 
	- 环境搭建
categories: 
	- Misc
---

## 0x00 概述

本博客采用Node.js + Hexo + Yelee + Git Pages方案。

博客运作的原理是，通过Hexo将写好的Markdown文章渲染为静态网页，然后将静态网页托管到Github上，使用Github的Git Pages来部署博客网站。

博客搭建可分为四部分：

- Hexo安装及使用
- Hexo配置
- Hexo联合Git Pages部署
- 绑定个人域名（可选）

<!-- more -->

## 0x01 Hexo安装

> Hexo 是一个快速、简洁且高效的博客框架。
>
> Hexo 使用 [Markdown](http://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

**1.安装Node.js与Git**

*Node*.*js* 是一个基于 Chrome V8 引擎的 JavaScript 运行环境。而Hexo是基于Node.js的，因此在安装Hexo之前要先安装Node.js。直接去官网下载安装程序安装即可。

Git用于进行版本控制。

使用命令`node -v`及其`git version`验证node.js与Git是否安装成功，运行命令出现版本即证明成功安装。

**2.安装Hexo**

安装命令：

`npm install -g hexo-cli`

> npm是 JavaScript 世界的包管理工具,并且是 Node.js 平台的默认包管理工具。

初始化博客：

`hexo init myblog`		 //myblog是可以任意取的。

安装依赖：

`cd myblog`		 //进入这个myblog文件夹
`npm install`		//npm会根据目录下的package.json文件安装所需要的依赖。

完成以上操作后，会创建myblog项目目录，目录结构如下(只列举重要的)：

> - public                            //该文件夹存放的是通过Hexo渲染markdown文章得到的最终静态文件。
> - scaffolds                      //该文件夹下存放文章、页面等的模板。
> - source\posts              //存放Markdown文章，Hexo要渲染的markdown文件就在该目录下。
> - themes                     //存放网页主题的目录。
> - _config.yml             //Hexo配置文件。

运行博客：

`hexo   g` 		     //等价于`hexo generate`，用于生成文章。即将posts目录下的Markdown文件渲染为静态文件并保存到public目录下。

`hexo  s`		   //等价于`hexo server`，运行博客。运行后会监听在localhost:4000，访问即可看到博客页面。

## 0x02 Hexo配置

**修改主题**

Hexo拥有众多主题，本博客采用[Yelee](https://github.com/MOxFIVE/hexo-theme-yelee)。

首先到Github上下载或使用`git clone`将主题下载并放到themes目录下。

然后修改hexo配置文件_config.yml，找到`theme:landscape`修改为`theme:yelee`。

关于主题的一些定制化，例如头像等，找到Yelee主题目录下的_config.yml配置文件进行更改。

## 0x03 Hexo联合Git Pages部署

**1.关于Git Pages**:

GitHub Pages 本用于介绍托管在 GitHub 的项目,他会为每个用户免费分配一个github.io域的域名。使用该域名可以访问同名的代码仓库。

**2.Git Pages配置**

首先，注册并拥有Github账号。

然后，创建一个新的代码仓库。并使用`xxx.github.io`作为仓库名字。

![](build-blog\br.jpg)

然后，配置Hexo的部署方式为Git。找到Hexo配置文件_config.yml ，修改`deploy`:

```
deploy:
  type: git
  repo: https://github.com/sh4douu/sh4douu.github.io.git  #这里是上一步创建的仓库连接。
```

**3.部署hexo到git pages：**

执行部署命令前，先安装用于部署的插件：`npm install hexo-deployer-git --save`

执行部署：`hexo   d`   //等同于hexo deploy

> 所谓的部署，即将hexo用于存放渲染完毕的静态文件的public目录push到刚才在github创建的仓库。
>
> 然后通过Git Pages分配的域名，实现博客网站的部署访问。

**4.Hexo写博客的流程**

首先，编写Markdown文章。使用`hexo new "title"`命令，该命令会以scaffolds目录下的post.md作为文章模板在source\posts目录下生成title.md。编写该markdown文件即可。

编写完成后，要将markdown文件渲染生成静态文件。执行命令：`hexo   g`。执行该命令后，会在public目录下生成渲染完毕的静态文件。

最后，将生成的静态文件push到github上进行部署。使用命令：`hexo    d `。

可能会因为缓存等原因造成某些问题，可以使用命令`hexo  clean`进行清扫。该命令会清除掉所有已渲染的文件及其缓存。清除后再重新生成、部署。

## 0x04 绑定个人域名

Git Pages分配的域名是`xxx.github.io`样式的，若想要使用自己的域名，可通过CNAME解析实现。

首先，注册购买域名，如：sakuxa.com。

然后设置解析，添加一条CNAME记录即可：

![](build-blog\cname.jpg)

然后在source目录下创建一个文件，名称为CNAME，文件无后缀。文件的内容写入域名，如：sakuxa.com

然后再到Github上，找到前面创建的仓库的`settings`，进入并找到自定义域名输入框填入自己的域名即可：

![](build-blog\git.jpg)



**参考连接**

https://blog.csdn.net/sinat_37781304/article/details/82729029

https://github.com/MOxFIVE/hexo-theme-yelee