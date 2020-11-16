---
title: CentOS7安装Jenkins
date: 2020-11-16 16:52:34
tags:
---
# CentOS7安装Jenkins
## 首先登录服务器更新系统软件
```
$ yum update
```

## 安装Java和git
```
$ yum install java
$ yum install git
```
注意java的版本来选择不同版本的Jenkins，上面一般会自动安装最行版本的Java-sdk。

## 安装nginx
```
$ yum install nginx //安装
$ service nginx start //启动
```
出现Redirecting to /bin/systemctl start nginx.service
说明nginx已经启动成功了，访问http://你的ip/，如果成功安装会出来nginx默认的欢迎界面

## 安装Jenkins
首先我们进入到`Jenkins`官网，传送门：https://jenkins.io/download/
这时我们会看到两栏，左边的`Stable (LTS)`是12周发布一版，右边的`Weekly`是每周发布一个版本。
这里我们以`Stable`为例进行安装，拖动页面找到你自己的系统并点击
![图片](./stable.png)
选择`Weekly`版本时，有可能受网络、`jenkins.noarch`导入不了等原因安装不成功，同时有可能安装成功后，`插件`无法安装

### 添加Jenkins repo:
```
wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins.io/redhat-stable/jenkins.repo
rpm --import http://pkg.jenkins.io/redhat-stable/jenkins.io.key
```

### 更新Jenkins repo cache：
```
yum clean all
yum makecache
```

### 安装Jenkins：
```
yum install -y jenkins
```

如果yum install安装慢，可以多尝试几次（网络对`jenkins.noarch`的下载很重要，有很大概率会失败），也可以到http://pkg.jenkins-ci.org/redhat-stable/ 中下载指定版本的安装包，再通过rpm来安装。

也可以使用国内Jenkins镜像来安装：
```
// 清华大学镜像
wget https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat-stable/jenkins-2.164.2-1.1.noarch.rpm
rpm -ivh jenkins-2.164.2-1.1.noarch.rpm
```

## 启动Jenkins
### 安装完成后启动Jenkins：
```
# 检查Jenkins服务状态
sudo systemctl status jenkins
# 设置为开机自启动
sudo systemctl enable jenkins
# 启动Jenkins服务
sudo systemctl start jenkins
```

### 为Jenkins开启防火墙8080端口：
```
# 检查防火墙配置
sudo firewall-cmd --list-all
# 开启8080端口
sudo firewall-cmd --zone=public --add-port=8080/tcp 
sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent
# 重新加载防火墙配置
sudo firewall-cmd --reload
```

在浏览器中访问`http://<jenkins_host_ip>:8080`确认是否可以打开Jenkins的`Getting Started`页面。
运行`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`，复制Jenkins初始密码，输入到Jenkins的Getting Started页面来`Unlock Jenkins`。


## 配置Jenkins
这里一般使用推荐插件进行安装，等待Jenkins插件安装完成。

使用`http://pkg.jenkins-ci.org/redhat/jenkins.repo`或`https://pkg.jenkins.io/redhat/jenkins.repo`，安装最新版本的`Jenkins`都有大概率的`jenkins.noarch`或`插件`安装失败，按照网上的各种处理方式都没有办法处理
例如：系统管理 -》 插件管理 -》 高级
将升级站点的URL`http://updates.jenkins-ci.org/update-center.json`更改为`http://mirror.esuni.jp/jenkins/updates/update-center.json`
![图片](./jenkins.png)

创建Jenkins的管理员账号，用该账号来登录Jenkins继续其它配置


## 卸载Jenkins

```
// 卸载rpm方式安装的jenkins
rpm -e jenkins

// 检查是否卸载成功
rpm -ql jenkins

// 彻底删除残留文件
find / -iname jenkins | xargs -n 1000 rm -rf
```
