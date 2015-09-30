---
layout: post
title:  "LNCG简明Web服务搭建方案"
date:   2015-09-16 07:42:51
categories: website
---

> 基于 [Go][go] 的 Web 服务解决方案

类似 LAMP ([Linux][linux] + [Apache][apache] + [MySQL][mysql] + [PHP][php])，LNCG 也是一套 Web 服务开发与部署解决方案的尝试：

- <span style="font-size: 120%; color: grey; font-weight: 1000;">&#120027;</span> : 服务器选择 [Linux][linux] 操作系统
- <span style="font-size: 120%; color: grey; font-weight: 1000;">&#120029;</span> : 对外提供的 Http/Https 服务使用 [Nginx][nginx]
- <span style="font-size: 120%; color: grey; font-weight: 1000;">&#120018;</span> : 使用云(e.g. [Qiniu](https://qiniu.com)) + 本地(e.g. [MySQL][mysql]/[Redis](http://redis.io)) 的方式存储数据
- <span style="font-size: 120%; color: grey; font-weight: 1000;">&#120022;</span> : 使用 [Go][go] (e.g. [Beego][beego] 框架) 开发业务逻辑，并负责与数据的交互

本文暂不涉及具体业务细节，重点在记述以 [Beego][beego] 框架搭建了独立的 Http Server 以后，如何配置 [Nginx][nginx] 代理对外提供服务。

### <span style="font-size: 120%; color: green; font-weight: 600;">&#9312;</span> 业务：[Beego][beego] 搭建框架

> [Beego][beego] 是一个典型的 [MVC](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) 架构，其主要设计灵感来源于 [tornado](http://www.tornadoweb.org)、[sinatra](http://www.sinatrarb.com) 和 [flask](http://flask.pocoo.org) 这三个框架，再结合 [Go][go] 本身的一些特性（interface、struct 嵌入等）而设计的一个 [RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer) 框架，可以用来快速开发 API、Web 及后端服务等各种应用。具体开发方式参见其[官方文档](http://beego.me/docs)，这里不做介绍。

首先安装 beego 和 bee 开发工具：

    $ go get -u -v github.com/astaxie/beego
    $ go get -u -v github.com/beego/bee

构建框架代码并启动 http server：

    $ bee new hello-world
    $ cd hello-world
    $ bee run hello-world

浏览器中看一下 <http://127.0.0.1:8080> ，你的 server 已经运行了。

### <span style="font-size: 120%; color: green; font-weight: 600;">&#9313;</span> 存储: 云 + 本地

作为一个网站，存储是必须要考虑的。目前笔者考虑 云 + 本地 的混合存储方案：

> 因为笔者的 vps 硬盘和内存都受限，不得不考虑把数据存储到云端，并通过 API 方式从云端拉取数据。

(1) **对于大规模的数据**

数据存储在云端（需着重考虑安全性问题），通过API Put/Get 数据。考虑到我们想用 [Go][go] 作为主要开发语言，这里选择对 [Go][go] 支持良好的国内云服务厂商 [七牛云存储](http://qiniu.com)：

    $ go get -u -v qiniupkg.com/api.v7

(2) **对于读取速度要求不是很强的中等规模的数据**

采用 [MySQL][mysql] 本地存储：

    $ go get -u -v github.com/go-sql-driver/mysql

(3) **对于读取速度约束很强的小规模数据**

采用 [Redis](http://redis.io) 在内存中存储

    $ go get -u -v gopkg.in/redis.v3


### <span style="font-size: 120%; color: green; font-weight: 600;">&#9314;</span> 服务：[Nginx][nginx] 部署代理

> [Go][go] 本身虽然可以提供独立的 http 服务，但从成熟性与安全性上说，[nginx][nginx] 可以帮我们做很多工作，例如访问日志，cc 攻击，静态服务等等。
>
> 所以我们可以采用这样的部署策略：[Go][go] 只要专注于业务逻辑和功能，通过 [Nginx][nginx] 配置代理实现多应用同时部署。

(1) 获取 nginx 源码并编译

    $ hg clone http://hg.nginx.org/nginx

更新时在 nginx 源码目录执行如下命令即可

    $ hg pull && hg update

(2) 撰写 Buildfile

    all install:
          ./auto/configure --user=www --group=www --prefix=${HOME}/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_realip_module --with-http_image_filter_module
          make
          make install
    clean:
          make clean

> 这里之所以是 Buildfile 而不是 Makefile 是因为 configure 的时候会生成用于真正编译的 Makefile

(3) 编译并安装

    $ cd nginx && make -f Buildfile

> 注意这里 make 出错时按照它的提示安装相应的确实包即可

(4) 配置 nginx 代理

    http {
        ...
        ...
        server {
            listen       80;
            server_name  xzturn;
            charset utf-8;
            access_log  off;
            location /(css|js|fonts|img)/ {
                access_log off;
                expires 1d;
                try_files $uri @backend;
            }
            location / {
                try_files /_not_exists_ @backend;
            }
            location @backend {
                proxy_set_header X-Forwarded-For $remote_addr;
                proxy_set_header Host            $http_host;
                proxy_pass                       http://127.0.0.1:8080;
            }
            ...
        }
        ...

(5) 链接 css/js/font/img 等网页相关配置与资源

    $ cd $HOME/nginx/html
    $ ln -sf path_to_your_beego_project/static/css
    $ ln -sf path_to_your_beego_project/static/js
    $ ln -sf path_to_your_beego_project/static/font
    $ ln -sf path_to_your_beego_project/static/img

> 由于你的网页开发也是在 [go][go] 框架中运行的，这些静态资源都存在 [go][go] 的开发目录下，[nginx][nginx] 代理的时候，需要在当前 html 路径下找到这些资源，所以需要做一下这些链接

(6) 启动 nginx

    $ sudo $HOME/nginx/sbin/nginx -c $HOME/nginx/conf/nginx.conf

再打开浏览器输入 <http://127.0.0.1> , 你将看到通过 [nginx][nginx] 代理提供的 [go][go] 服务了。

***
<br>

至此，整个框架就搭建完成了，下面你就可以专注你自己的业务逻辑了。

[linux]:          http://www.linux.org
[apache]:         http://www.apache.org
[nginx]:          http://nginx.org
[mysql]:          https://www.mysql.com
[php]:            https://secure.php.net
[go]:             https://golang.org
[beego]:          http://beego.me
