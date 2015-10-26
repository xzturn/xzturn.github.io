---
layout: post
title:  "程序员建站极简指南"
date:   2015-09-01 08:21:29
categories: website
---

## 1. 购买域名

个人推荐 [GoDaddy][godaddy] 老品牌、稳定、活动力度还不小（非店大欺客），且支持支付宝付款；缺点是不支持中文。
- 很久之前用过 [Bluehost][bluehost]，体验很差（主要是慢），这次没试，刚看了一下，已经支持[中文](http://cn.bluehost.com)了，技术支持应该比之前会好一些。
- 这次还看了糖果主机 [Sugarhosts][sugarhosts] ，支持[中文](http://www.sugarhosts.com/zh-cn/)和支付宝付款，不过笔者订购的时候比 [GoDaddy][godaddy] 贵一些，而且生效时间比 [GoDaddy][godaddy] 长（交完钱之后挺长时间都是“待定”状态，而 [GoDaddy][godaddy] 申请完就生效了）；最坑爹的是域名服务不支持退款，跟它客服沟通了半天，才把钱给退了（买服务的时候一定要把服务条款看清，慎之慎之）。

> P.S. 多说一句，目前最火（贵）的 “高大上” 域名顺序是（.com域名性价比较高）：
> 
> .io > .me > .in > .xyz > .cc > .com > .org > .net > .info

## 2. 购买 [VPS][vps]

一般的虚拟主机限制太多（当然最便宜），而独立服务器又太贵（个人试水的时候完全没必要），这时候 [VPS][vps] 就提供了一个个人建站性价比最高的解决方案：你可以随便折腾你的系统（**这点对程序员来说非常重要**），又花费不高。

这里强烈推荐 [Linode][linode]，高度可定制、速度还可以（国内用户建议选择日本机房），最基础(便宜)的版本 [Linode][linode] 1GB 每月10美元（不过目前好像这个版本没有日本机房了，囧）；你还可以同时选择 [Vuler](https://www.vultr.com/) 和 [DigitalOcean](https://www.digitalocean.com/) 对 [Linode][linode] 进行容灾备份，它们的最基础价格都是每月5美元。

> 购买好 [vps][vps] 以后，你就可以把你的 IP 绑定到你的域名上，准备提供对外网站访问了。

当然，还有一些安全设置、网络高级配置等等，可以按照[Linode的文档库](https://www.linode.com/docs/)中所讲的一项一项设置。

## 3. [VPS][vps] 环境设置

笔者的目的是试水个人网站开发，不采用传统的LAMP架构；数据存储目前计划是使用免费的云存储（如[七牛云](http://www.qiniu.com/)），开发语言选择 [Go][go]（Full Stack 解决方案），服务器代理用 [Nginx][nginx]；还可以尝试 [Docker][docker]，看看是否能建立一套不一样的生态体系。

### [Go][go] 环境搭建

#### 先安装 go1.4

(1) 下载源码
{% highlight bash %}
$ wget https://storage.googleapis.com/golang/go1.4.2.src.tar.gz
{% endhighlight %}
(2) 设置环境
{% highlight bash %}
$ vim ~.bashrc
{% endhighlight %}
增加如下设置
{% highlight bash %}
export GOROOT=$HOME/go
export GOOS=linux
export GOARCH=amd64
export GOBIN=$GOROOT/bin
export GOPATH=$HOME/workspace/go
export PATH=$PATH:$GOBIN
{% endhighlight %}
(3) 安装
{% highlight bash %}
$ tar -xzvf go1.4.2.src.tar.gz
$ cd go/src && ./all.bash
{% endhighlight %}
(4) 验证安装
{% highlight bash %}
$ go version
{% endhighlight %}
此时你应该能看到如下信息
{% highlight bash %}
go version go1.4.2 linux/amd64
{% endhighlight %}
(5) 为了下面能够顺畅安装 go1.5，把源码拷到 `$GOROOT` 下，并改名
{% highlight bash %}
$ cd .. && cp -rf * $GOROOT
$ cd && mv go go1.4
{% endhighlight %}

#### 再安装高大上的 go1.5

(1) 获取源码
{% highlight bash %}
$ git clone https://go.googlesource.com/go
$ cd go
$ git checkout go1.5.1
{% endhighlight %}
(2) 编译
{% highlight bash %}
$ cd src && ./all.bash
{% endhighlight %}
(3) 验证安装
{% highlight bash %}
$ go version
{% endhighlight %}
此时你应该能看到如下信息
{% highlight bash %}
go version go1.5.1 linux/amd64
{% endhighlight %}
(4) (Optional) 源码到 GOROOT 下并删除 go1.4
{% highlight bash %}
$ cd .. && cp -rf * $GOROOT
$ cd && rm -rf go1.4
{% endhighlight %}
(5) 设置 [go][go] 工作环境
{% highlight bash %}
$ cd && mkdir -p worksapce/go
$ cd workspace/go && mkdir bin pkg src
{% endhighlight %}
(6) 安装有用的 go tools
{% highlight bash %}
$ go get golang.org/x/tools/cmd/...
{% endhighlight %}
(7) (Optional) 搭建 web 网站，推荐一个非常好用的 Go 框架 [beego](http://beego.me)
{% highlight bash %}
$ go get github.com/astaxie/beego
$ go get github.com/beego/bee
{% endhighlight %}

至此，你的 [Go][go] 环境就完全设置好了，以上是在 Ubuntu14.04 LTS 上亲测可行的源码安装方式，更详细的信息，可以参考其[官方文档](https://golang.org/doc/install/source)。

### [Nginx][nginx] 环境

(1) 获取源码
{% highlight bash %}
$ hg clone http://hg.nginx.org/nginx
{% endhighlight %}
(2) 编写 Buildfile
{% highlight bash %}
$ cd nginx && vim Buildfile
{% endhighlight %}
内容如下:
{% highlight bash %}
all install:
	  ./auto/configure --user=www --group=www --prefix=${HOME}/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_realip_module --with-http_image_filter_module
	  make
	  make install
clean:
	  make clean
{% endhighlight %}
(3) 安装依赖
{% highlight bash %}
$ sudo apt-get install libpcre3-dev libssl-dev libgd-dev
{% endhighlight %}
(4) 源码安装
{% highlight bash %}
$ make -f Buildfile
{% endhighlight %}
(5) 验证安装
{% highlight bash %}
$ $HOME/nginx/sbin/nginx -v
{% endhighlight %}
如果看到类似如下信息，证明nginx安装成功
{% highlight bash %}
nginx version: nginx/1.9.5
{% endhighlight %}

### [Docker][docker] 环境

关于Docker环境的安装，可以参考我[这篇文章]({% post_url 2015-08-24-docker-install %})，这里再简单列一下：

(1) 获取最新的 [docker][docker] package
{% highlight bash %}
$ curl -sSL https://get.docker.com/gpg | sudo apt-key add -
$ curl -sSL https://get.docker.com/ | sh
{% endhighlight %}
(2) 创建 [docker][docker] 用户组并添加到你的用户
{% highlight bash %}
$ sudo usermod -aG docker your_user_name
{% endhighlight %}
(3) 更新最新的 [docker][docker] engine
{% highlight bash %}
$ curl -sSL https://get.docker.com/ | sh
{% endhighlight %}
(4) 验证安装
{% highlight bash %}
$ docker run hello-world
{% endhighlight %}
(5) (Optional) 获取 [docker][docker] 源码
{% highlight bash %}
$ git clone https://github.com/docker/docker.git
{% endhighlight %}

## 4. <a name="ssproxy">SS上网配置</a>

天朝的上网大家都懂的，为了自由地获取信息，笔者曾经使用过多种不同的方法，早期以免费为主，包括

* [洋葱路由器 Tor](https://www.torproject.org/)[^1]
* 基于 [GAE](https://appengine.google.com/) 的 [goagent](https://github.com/goagent/goagent)[^2]

它们最大的缺点是不稳定，下面介绍一种基于 [ShadowSockes](http://shadowsocks.org) 的“安全”上网方法，不多说，步骤如下[^3]:

(1) 在你的 [VPS][vps] 上运行 shadowsocks-server:
{% highlight bash %}
$ go get github.com/shadowsocks/shadowsocks-go/cmd/shadowsocks-server
{% endhighlight %}
并配置 config.json, E.g. 你的 server 开启 8088 端口提供服务：
{% highlight bash %}
{
    "server":"127.0.0.1",
    "server_port":8088,
    "password":"MySSServerPasswd@VPS",
    "timeout":600,
    "method":"aes-128-cfb"
}
{% endhighlight %}
最后启动
{% highlight bash %}
$ nohup shadowsocks-server -c config.json > sss.log &
{% endhighlight %}
(2) 在你的本机运行 client:
{% highlight bash %}
$ go get github.com/shadowsocks/shadowsocks-go/cmd/shadowsocks-local
{% endhighlight %}
并配置 config.json，E.g. 本机开启 7070 端口进行服务：
{% highlight bash %}
{
    "server":"your_vps_ip_address",
    "server_port":8088,
    "local_address":"127.0.0.1",
    "local_port":7070,
    "password":"MySSServerPasswd@VPS",
    "timeout":600,
    "method":"aes-128-cfb"
}
{% endhighlight %}
最后启动
{% highlight bash %}
$ nohup shadowsocks-local -c config.json > ssl.log &
{% endhighlight %}
(3) 这时，你就可以用如下配置在本机安全上网了:
{% highlight bash %}
SOCKS5 127.0.0.1:7070
{% endhighlight %}
比如，你可以配置 [chrome](http://www.google.com/chrome) 的 [SwitchyOmega](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif) 插件，新建 profile 把上述 proxy 信息填进去，保存后使用此 profile，chrome 访问 [https://www.google.com/ncr](https://www.google.com/ncr)，bingo :) 享受你的自由吧！

至此，环境就基本搭好了，可以继续折腾你的个人网站了。

***
<br>

[^1]: 被墙，需自备翻墙工具访问（这又是类似一个鸡生蛋蛋生鸡的问题）；第三方提供的下载需谨慎，一般都有后门
[^2]: goagent 作者迫于压力，在 [github](https://github.com) 的源码已经删除
[^3]: shadowsocks 的源码在 [github](https://github.com) 也已经删除了，目前用的是其 [go][go] 实现的一个版本 [shadowsocks-go](https://github.com/shadowsocks/shadowsocks-go)


[godaddy]:        https://godaddy.com
[bluehost]:       http://bluehost.com
[sugarhosts]:     http://www.sugarhosts.com
[linode]:         https://linode.com
[vps]:            https://en.wikipedia.org/wiki/Virtual_private_server
[docker]:         https://docker.com
[go]:             https://golang.org
[nginx]:          http://nginx.org
