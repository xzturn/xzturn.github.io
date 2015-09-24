---
layout: post
title:  "Docker实战之Shadowsocks构建"
date:   2015-09-11 09:22:21
categories: docker
---

本文假设你的系统里面 [docker-engine][docker] 已经装好，具体安装过程请参考这一篇：[Docker安装备忘]({% post_url 2015-08-24-docker-install %})。

下面仍然以 [shadowsocks-go][shadowsocks-go][^1] 为例，看怎么写 [Dockerfile][dockerfile] 来快速构建运行环境。

### <span style="font-size: 100%; color: gray; font-weight: 800;">&#120793;</span> 撰写 [Dockerfile][dockerfile]

{% highlight bash %}
FROM buildpack-deps:jessie-scm

# gcc for cgo
RUN apt-get update && apt-get install -y \
		gcc libc6-dev make \
		--no-install-recommends \
	&& rm -rf /var/lib/apt/lists/*

ENV GOLANG_VERSION 1.5.1
ENV GOLANG_DOWNLOAD_URL https://golang.org/dl/go$GOLANG_VERSION.linux-amd64.tar.gz
ENV GOLANG_DOWNLOAD_SHA1 46eecd290d8803887dec718c691cc243f2175fe0

RUN curl -fsSL "$GOLANG_DOWNLOAD_URL" -o golang.tar.gz \
	&& echo "$GOLANG_DOWNLOAD_SHA1  golang.tar.gz" | sha1sum -c - \
	&& tar -C /usr/local -xzf golang.tar.gz \
	&& rm golang.tar.gz

ENV GOPATH /go
ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH

RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 777 "$GOPATH"
WORKDIR $GOPATH

COPY go-wrapper /usr/local/bin/

RUN go get -u -v github.com/shadowsocks/shadowsocks-go/cmd/shadowsocks-server
{% endhighlight %}

1. [Dockerfile][dockerfile] 必须从 <span style="font-size: 100%; font-weight: 800;">&#70;&#82;&#79;&#77;</span> 开始，表示**基于的镜像名称**，本例中为构建 [golang][go] 的安装依赖。
2. <span style="font-size: 100%; font-weight: 800;">&#82;&#85;&#78;</span> 指令指明了在当前镜像上执行指令，并提交为新的镜像；<span style="font-size: 100%; font-weight: 800;">&#69;&#78;&#86;</span> 指令则用于设置镜像的环境变量。上面做的事情就其实很简单了：安装编译 [go][go] 必须的基础依赖 <span style="font-size: 100%; font-weight: 100;">&#10137;</span> 设置 [go][go] 环境变量<span style="font-size: 100%; font-weight: 100;">&#10137;</span> 下载解压 <span style="font-size: 100%; font-weight: 100;">&#10137;</span> 设定 `GOPATH`。
3. <span style="font-size: 100%; font-weight: 800;">&#87;&#79;&#82;&#75;&#68;&#73;&#82;</span> 指令为其他指令指定工作目录，<span style="font-size: 100%; font-weight: 800;">&#67;&#79;&#80;&#89;</span> 指令会把本机文件（夹）拷贝到镜像指定位置，这里做的事情就是把工作目录切换到 `GOPATH`，并拷贝 `go-wrapper`[^2] 脚本到 `/usr/local/bin` 目录下。
4. 运行 go get 获取 shadowsocks-server。

### <span style="font-size: 100%; color: gray; font-weight: 800;">&#120794;</span> 构建镜像
[Dockerfile][dockerfile] 写好以后，就可以 build 它了，假设 [Dockerfile][dockerfile] 放在目录 ssgo 下：
{% highlight bash %}
$ sudo docker build ssgo
{% endhighlight %}

构建成功以后，看一下：
{% highlight bash %}
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
<none>              <none>              acab4cbd1432        51 seconds ago      663.8 MB
buildpack-deps      jessie-scm          20b348f4d568        2 days ago          291.8 MB
{% endhighlight %}

给你新构建的镜像打一个 tag:
{% highlight bash %}
$ docker tag acab4cbd1432 zturn/ssgo:latest
{% endhighlight %}

下面就可以检查一下镜像里面的 [go][go] 是否正常了：
{% highlight bash %}
$ docker run zturn/ssgo go version
go version go1.5.1 linux/amd64
{% endhighlight %}

[go][go] 正常了，那么通过 go get 安装的 shadowsocks-server 也应该是正常的：
{% highlight bash %}
$ docker run zturn/ssgo shadowsocks-server -h
Usage of shadowsocks-server:
  -c string
    	specify config file (default "config.json")
  -core int
    	maximum number of CPU cores to use, default is determinied by Go runtime
  -d	print debug message
  -k string
    	password
  -m string
    	encryption method, default: aes-256-cfb
  -p int
    	server port
  -t int
    	timeout in seconds (default 300)
  -version
    	print version
{% endhighlight %}

### <span style="font-size: 100%; color: gray; font-weight: 800;">&#120795;</span> 推送到 [DockerHub][dockerhub]
{% highlight bash %}
$ docker login --username={your_name} --password={your_password} --email={your_email}
{% endhighlight %}

正常登录以后，就可以
{% highlight bash %}
$ docker push zturn/ssgo
{% endhighlight %}

等它推送成功就可以了。

### <span style="font-size: 100%; color: gray; font-weight: 800;">&#120796;</span> 在 VPS 上启用基于 [docker][docker] 的 shadowsocks-server

上面三步都可以在本机操作，server 则必然在 vps 上启动的：
{% highlight bash %}
$ docker pull zturn/ssgo
{% endhighlight %}

成功以后，用 docker 启动 shadowsocks-server：
{% highlight bash %}
$ docker run -p 8089:8089 zturn/ssgo shadowsocks-server -d -k {your_ss_server_passwd} -m aes-128-cfb -p 8089
{% endhighlight %}

在这里**一定要注意这一点**: docker run -p 参数指定容器的端口映射。

> 这条命令可以拆开成两部分看：
> 
> 1. shadowsocks-server -d -k {your_ss_server_passwd} -m aes-128-cfb -p 8089
> 2. docker run -p 8089:8089 zturn/ssgo {docker_container_env_cmd}
>
> 命令 1 中是指明了在 [docker][docker] 容器里面使用 shadowsocks-server 在容器内端口 8089 提供服务；
> 命令 2 则是告诉 [docker][docker] 在运行时，将容器的8089端口映射到本机实际的8089端口对外提供服务

这些都做好以后，你的 shadowsocks client 就可以通过连接你的这个 vps_ip:8089 来进行安全上网了。

我的这个镜像已经放到了 [DockerHub][dockerhub] 上（[zturn/ssgo](https://hub.docker.com/r/zturn/ssgo/)），有兴趣的朋友可以自行尝试 :)

****
<br>

[^1]: 之所以选择 [shadowsocks][shadowsocks] 作例子，还是因为“安全”上网，你懂得。。。
[^2]: `go-wrapper` 是一个 [go][go] 的辅助脚本，可以暂时不关注。

[docker]:         https://docker.com
[shadowsocks]:    http://shadowsocks.org
[shadowsocks-go]: https://github.com/shadowsocks/shadowsocks-go
[dockerfile]:     https://docs.docker.com/reference/builder
[dockerhub]:      https://hub.docker.com
[go]:             https://golang.org
