# blog
基于Hexo的博客项目系统

## 什么是Hexo？

Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 [Markdown](https://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

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
