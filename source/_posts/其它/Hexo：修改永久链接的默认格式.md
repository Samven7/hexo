---
title: Hexo：修改永久链接的默认格式
top: true
toc: true
mathjax: false
tags:
  - hexo
categories: 其它
abbrlink: 3b228ab0
date: 2019-10-21 13:00:12
summary:
---



### 前言

如果你的文章名称是中文的，那么 Hexo 默认生成的永久链接也会有中文，这样不利于 SEO，且  gitment 评论对中文链接也不支持。

<!-- more -->

在 Hexo 根目录下的 _config.yml 文件采用初始设置。这里因为用“年月日”会让文章链接的层次太深，所以我用"article"代替：

~~~yaml
# permalink: :year/:month/:day/:title/
permalink: article/:title.html
permalink_defaults:
~~~

- 生成的文章链接就是（标题为“我的个人博客”）：

	https://[你的网站域名]/article/我的个人博客.html

链接中出现中文显然不太好，所以下面给出三种替代中的方法。

## 三种解决方案

### 1.1 安装插件方法一（推荐）

在 Hexo 根目录下使用 git bash 执行命令：

```bash
npm install hexo-abbrlink --save
```

在 Hexo 根目录下的 _config.yml 文件，修改为如下配置： 

~~~yml
# permalink: :year/:month/:day/:title/
# permalink: article/:title/
permalink: article/:abbrlink.html
abbrlink:
  alg: crc32  # 算法：crc16(default) and crc32
  rep: hex    # 进制：dec(default) and hex
permalink_defaults:
~~~

然后在 git bash 按顺序运行如下命令：

~~~bash
hexo clean #清除缓存 网页正常情况下可以忽略此条命令
hexo g #生成静态网页
hexo d #开始部署
~~~

再打开网站的文件就可以看到效果。

- 效果（其中60762就是随机生成的）

	https://[你的网站域名]/article/60762.html



### 1.2 安装插件方法二

中文链接转拼音

在 Hexo 根目录下使用 git bash 执行命令：

```bash
npm i hexo-permalink-pinyin --save
```

在 Hexo 根目录下的 _config.yml 文件中，修改以下的配置项：

```yml
permalink: article/:title.html
permalink_pinyin:
  enable: true
  separator: '-' # default: '-'
permalink_defaults:
```

然后在 git bash 按顺序运行如下命令：

~~~bash
hexo clean #清除缓存 网页正常情况下可以忽略此条命令
hexo g #生成静态网页
hexo d #开始部署
~~~

再打开网站的文件就可以看到效果。

- 效果（标题为“我的个人博客”）

	https://[你的网站域名]/article/wo-de-ge-ren-bo-ke.html



### 1.3 采用urlname

在写每篇md文章的时候，在 Front-matter 里加上urlname：

~~~
---
title: typora-vue-theme主题介绍
date: 2018-09-07 09:25:00
urlname: 2019102101
---
~~~

在 Hexo 根目录下的 _config.yml 文件中，修改以下的配置项：

~~~yml
permalink: article/:urlname.html  # urlname值文章里必须填写，格式201905260105
permalink_defaults:
~~~

然后在 git bash 按顺序运行如下命令：

~~~bash
hexo clean #清除缓存 网页正常情况下可以忽略此条命令
hexo g #生成静态网页
hexo d #开始部署
~~~

再打开网站的文件就可以看到效果。

- 效果

	https://[你的网站域名]/article/2019102101.html



### 小结

第一种方法是我试过中最好的；第三次之，因为每次都要手动加上urlname；而第二种，当文章的中文标题名字过长时，效果并不好。