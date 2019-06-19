---
title: 搭建Hexo博客系统简单流程
date: 2019-05-18 16:33:33
tags:
---

## 什么是Hexo？

Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 [Markdown](https://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。不想看下面的内容，可以查看[官方文档](https://hexo.io/zh-cn/docs/themes.html)

## 安装

安装Hexo其实十分简单，也就几分钟的事。

### 前提条件

环境支持：

- [Node.js](https://nodejs.org/en/)(V6.9+)
- [Git](https://git-scm.com/)

安装好上述环境后（步骤自行百度），接下来就是用npm或者cnpm来完成Hexo的安装吧。

``````
$ npm install -g hexo-cli
``````

### 博客项目来了

1.先创建一个文件夹（用来存放所有blog的东西，随意点就叫blog），然后 `cd` 到该目录下。
2.初始化命令：`hexo init`，初始化后可在blog目录先看到文件。

``````
$ mkdir blog
$ cd blog
$ hexo init
$ npm install
``````

新建完成后，blog文件夹的目录如下：

``````
- node_modules
- scaffolds
- source
   - _posts
- themes
   - landscape
- .gitignore
- _config.yml
- package-lock.json
- package.json
``````

### 配置主题

这里选择了[hexo-theme-laughing](https://github.com/BoizZ/hexo-theme-laughing)主题，好奇的你也可以自行去选择你自己喜欢的[主题](https://hexo.io/themes/)

步骤：
1. clone主题到themes目录下

2. 主题更改

``` yaml
# blog下_config.yml 文件
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
## 更改主题，默认为landscape主题
theme: hexo-theme-laughing

```

3. 自定义主题内容

``` yaml
# hexo-theme-laughing下_config.yml 文件

# Icon
favicon: /favicon.ico
# SEO
keywords: Hexo, Gruntjs, Nodejs, Reactjs, Vuejs
# Header，首页大图，文章大图
page_background: http://callfiles.ueibo.com/hexo-theme-laughing/page_background.jpg
page_menu_button: dark
post_background: http://callfiles.ueibo.com/hexo-theme-laughing/post_background.jpg
post_menu_button: light
title_plancehold: 技术博客
# 用户头像、简介
author:
  head: https://avatars1.githubusercontent.com/u/15698499?s=460&v=4
  signature: 千呼万唤始出来，犹抱琵琶半遮面.
# 右边导航栏配置
navication:
  - name: Github
    link: https://github.com/mugongxu
  - name: V2EX
    link: https://www.v2ex.com/
  - name: 酷猫
    link: http://m.xuguoqian.com/
# content
content_width: 800
# Footer
## Social icon list: [facebook, twitter, weibo, wechat, github, stackoverflow, linkin, email, segmentfault, flickr, zhihu, disqus, douban, bilibili]
# 一些常见技术站路径配置
social:
  - name: Github
    icon: github
    link: https://github.com/BoizZ
  - name: Weibo
    icon: weibo
    link: https://weibo.com/heqibang
  - name: SegmentFault
    icon: segmentfault
    link: https://www.segmentfault.com/u/bon
# Duoshuo
duoshuo:
  enable: false
  siteName: ueibo
# Copyright
copyright:
  record: false
  hexo: false
  laughing: false

```

### 开始写博客了

``````
cd blog
# 创建文章
hexo new 'pagename'
# 启动服务
hexo server

# 然后怎么呢？然后你就可以在http://localhost:4000上看到你自己的blog了嘛（嘻嘻）

``````

#### `到这里是不是有点成就呢，可是后续怎么把自己写的东西打包成静态文件与github.io以及自己的域名关联呢`

## 一句话不要急...

### 打包与github.io关联及发布

``` yaml
# blog下_config.yml 文件，增加下面配置
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/mugongxu/mugongxu.github.io.git
  branch: master
```

然后：

``````
# 生成静态文件
hexo generate
# 发布到github.io上
hexo deploy

``````

再然后，你就可以再自己的 https://xxx.github.io/ 上看到你的博客啦

### 那如果你不想使用github.io来访问你的博客怎么办？很简单

1. 你要有自己的域名，没有的自行去阿里云购买。

2. 域名里面解析到主机

> 打开阿里云域名解析，添加一条解析：
> 记录类型选CNAME。其中主机记录中的page是要解析的域名，我这里解析二级域名，如果要解析 www.xuguoqian.com ，直接填www就行了,这个地方应该没啥好说的。下面的记录值就填github的部署域名xxx.github.com(前提是github的服务已经部署成功，也就是访问xxx.github.com能正常访问)。其他的选项默认就可以了。

3. 主机里面解析至域名

> 从github解析到域名也很简单，就是进入xxx.github.io这个项目，在根目录创建CNAME这个文件（注意这里没有后缀）,然后在这个文件里面写入我们的域名就可以了，我这里写入的是你解析的域名(blog.xuguoqian.com),然后保存就可以了。

上面两步操作完了，等待10分钟后，应该就可以访问了，比如我的blog.xuguoqian.com就可以访问了。


## 没了...
