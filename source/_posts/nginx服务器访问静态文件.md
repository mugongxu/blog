---
title: nginx服务器访问静态文件
date: 2019-05-18 16:32:34
tags:
---

#### nginx服务器访问静态文件

首先，你需要有静态资源页面（这是废话）。

其次，进行Nginx配置：

> 项目demo配置
> nginx配置
> location自定义代理：但是注意的是，`代理名称路径必须是紧接着root后面的路径`
> 访问服务器/data/server/ui/clock-of-diagrams/dist/下的文件，
> `root为/data/server/ui/，那location代理名称须是：/clock-of-diagrams/dist/`
> root为静态文件在服务器里的具体路径目录
> index默认访问的文件名称

``````js
# 项目demo配置

server {
        listen  80;
        server_name     demo.xuguoqian.com;

        location / {
                root /data/server/ui/ant-design-vue-template/dist/;
                index index.html;
        }

        location ^~ /clock-of-diagrams/dist/ {
                root /data/server/ui/;
        }

        location ^~ /five-in-a-row/ {
                root /data/server/ui/;
        }

        error_page 500 502 503 504  /50x.html;
}

``````

然后就可以通过 http://demo.xuguoqian.com//clock-of-diagrams/dist/index.html 来访问服务器/data/server/ui/clock-of-diagrams/dist/下的文件了，大功告成！！！！！！
