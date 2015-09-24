---
layout: post
title:  "Docker安装备忘"
date:   2015-08-24 11:30:46
categories: docker
---

打算写一系列 [Docker][docker] 的文章，主要是记述自己使用过程中遇到的问题及解决方法。权威的解释还是要参考[官方文档][docker-docs]。

<span style="font-size: 120%; color: black; font-weight: 600;">&#9352;</span> 笔者的环境为 [Ubuntu-14.04.3](http://cdimage.ubuntu.com/ubuntu/releases/14.04.3/release/)[^1]，这也是 [docker][docker] 安装支持最友好的版本之一。

<span style="font-size: 120%; color: black; font-weight: 600;">&#9353;</span> 安装 [curl](http://curl.haxx.se):
{% highlight bash %}
$ sudo apt-get install curl
{% endhighlight %}

<span style="font-size: 120%; color: black; font-weight: 600;">&#9354;</span> 添加 [docker][docker] 的 apt-key （可选）
{% highlight bash %}
$ curl -sSL https://get.docker.com/gpg | sudo apt-key add -
{% endhighlight %}

<span style="font-size: 120%; color: black; font-weight: 600;">&#9355;</span> 获取 [Docker][docker][^2]
{% highlight bash %}
$ curl -sSL https://get.docker.com/ | sh
{% endhighlight %}

<span style="font-size: 120%; color: black; font-weight: 600;">&#9356;</span> 创建 [docker][docker] 用户组并添加到你的用户（可选）
{% highlight bash %}
$ sudo usermod -aG docker your_user_name
{% endhighlight %}

<span style="font-size: 120%; color: black; font-weight: 600;">&#9357;</span> 启动 docker daemon[^3]
{% highlight bash %}
$ sudo start docker
{% endhighlight %}

<span style="font-size: 120%; color: black; font-weight: 600;">&#9358;</span> 验证安装
{% highlight bash %}
$ docker run hello-world
{% endhighlight %}

如果你最终看到如下信息，那么恭喜你，你的 docker engine 已经安装成功了 :)
{% highlight bash %}
Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
  1. The Docker client contacted the Docker daemon.
  2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
  3. The Docker daemon created a new container from that image which runs the executable that produces the output you are currently reading.
  4. The Docker daemon streamed that output to the Docker client, which sent it to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/userguide/
{% endhighlight %}

下面你就可以开始体会 [docker][docker] 的威力了。

****
<br>

[^1]: 这也是[ubuntu官方目前推荐版本](https://wiki.ubuntu.com/TrustyTahr/ReleaseNotes)
[^2]: 如果你系统里面已有的docker是通过这种方式安装的，那么这条命令也是更新docker engine的命令；否则，最好不要执行这条命令（如果你非要执行这条命令，那么需要先把你系统中已有的docker先卸载掉），它可能把你的系统搞挂掉，不可不知，不可不知，不可不知
[^3]: 这步在[docker的官方文档](https://docs.docker.com/installation/ubuntulinux/)里面是没有的，但是不做这一步笔者的系统会报错，特记述之

[docker]:       https://docker.com
[docker-docs]:  https://docs.docker.com
