---
title: Jenkins在Gitlab中实现
date: 2020-11-25 12:41:08
tags:
---

## 安装Jenkins
[https://blog.xuguoqian.com/2020/11/16/CentOS7安装Jenkins/](https://blog.xuguoqian.com/2020/11/16/CentOS7安装Jenkins/)

## 安装Gitlab
[https://blog.xuguoqian.com/2020/11/17/CentOS7安装gitlab/](https://blog.xuguoqian.com/2020/11/17/CentOS7安装gitlab/)

## 配置Jenkins
1. 安装和Git，GitLab插件
系统管理 -> 插件管理 -> 搜索插件并安装
![gitlab-plugins](./gitlab-plugins.png)

2. 配置GitLab插件
安装好GitLab插件后，系统管理 -> 系统配置，下拉找到gitlab配置项：
![gitlab-setting](./gitlab-setting.png)

Gitlab API token的获取：
![seeting](./setting.png)
![seeting_01](./setting_01.png)
![seeting_02](./setting_02.png)
![seeting_03](./setting_03.png)

生成token后，就可以添加Jenkins的Credentials了：
![seeting_04](./setting_04.png)

3. 创建一个Jenkins Job
可参考[Jenkins自动布署你的Vue项目](https://blog.xuguoqian.com/2020/11/24/Jenkins自动布署你的Vue项目/)

4. 配置Job
这里主要是采用webhook来监听，从而实现gitlab的自动发版。想用CI/CD的pipelines模式或其他模式的，自行google。
打开新建的Job：进入`配置`:
![seeting_05](./setting_05.png)
选择`GitLab Connection`：（在第二步配置的配置项）
![seeting_06](./setting_06.png)

然后其他步骤和[Jenkins自动布署你的Vue项目](https://blog.xuguoqian.com/2020/11/24/Jenkins自动布署你的Vue项目/)中一样

## 配置Gitlab
1. 配置Jenkins构建触发器
![seeting_07](./setting_07.png)
2. 配置Gitlab的webhook
![seeting_08](./setting_08.png)

然后你就可以监听到项目的master分支的push事件，实现自动发版了。