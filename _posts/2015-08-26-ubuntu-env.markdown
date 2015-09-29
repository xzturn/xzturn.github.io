---
layout: post
title:  "构建Ubuntu工作环境"
date:   2015-08-26 07:22:22
categories: ubuntu
---

作为一个小有“情怀”的程序员，长期以来在 [Ubuntu][ubuntu] 环境工作，也一直比较嗨皮。不过前两天折腾 [docker][docker] 把系统搞挂了，又快速重新恢复了一遍系统环境，觉得有必要记录一下环境的构建过程。

## 系统快速安装

用U盘启动盘安装最为简洁方便，过程如下：

<span style="font-size: 100%; color: black; font-weight: 100;">&#120783;</span>. 从 [cdimage](http://cdimage.ubuntu.com/ubuntu/releases/14.04.3/release/) 上下载 [ubuntu][ubuntu] 映像

    wget http://cdimage.ubuntu.com/ubuntu/releases/14.04.3/release/ubuntu-14.04-desktop-amd64+mac.iso

<span style="font-size: 100%; color: black; font-weight: 100;">&#120784;</span>. 在 windows 环境下推荐使用 [Universal USB Installer](http://www.pendrivelinux.com/universal-usb-installer-easy-as-1-2-3/) 制作U盘启动盘

<span style="font-size: 100%; color: black; font-weight: 100;">&#120785;</span>. 重启机器，在 BIOS 里面设置U盘优先启动

<span style="font-size: 100%; color: black; font-weight: 100;">&#120786;</span>. 进入 [Ubuntu][ubuntu] 安装界面

<span style="font-size: 100%; color: black; font-weight: 100;">&#120787;</span>. 分区安装

> 这里一定要养成的一个好习惯是 `/home` 目录与根目录 `/` 分开在不同的逻辑分区上，这样如果重装，`/home` 中的所有东西包括设置都可以原样保留

安装过程中最好有网络，这样可以把比较新的更新包含进来。大概十分钟之后，你的系统就安装好了。

## 系统必备

<span style="font-size: 100%; color: black; font-weight: 100;">&#120783;</span>. [搜狗拼音输入法Linux版](http://pinyin.sogou.com/linux/?r=pinyin)

> 下载后直接 deb 安装，重启系统后就OK了

<span style="font-size: 100%; color: black; font-weight: 100;">&#120784;</span>. [google chrome](https://chrome.google.com)
    
> 下载 deb 安装，由于官网被墙，可以在国内的非官方镜像下载（一搜一大把），然后再更新

    wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | sudo apt-key add -

<span style="font-size: 100%; color: black; font-weight: 100;">&#120785;</span>. 更新源设置

> 一般增加 **网易源** 和 **阿里云** 的源

## 开发环境构建

<span style="font-size: 100%; color: black; font-weight: 100;">&#120783;</span>. 基础开发工具

    sudo apt-get install build-essential libstdc++6

<span style="font-size: 100%; color: black; font-weight: 100;">&#120784;</span>. 升级 gcc 到 4.9.2（支持 C++11）

    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get install gcc-4.9 g++-4.9

<span style="font-size: 100%; color: black; font-weight: 100;">&#120785;</span>. 安装代码管理工具

    sudo atp-get install git subversion mercurial

<span style="font-size: 100%; color: black; font-weight: 100;">&#120786;</span>. 翻墙还是必不可少的，推荐免费的 [goagent](https://github.com/goagent)

    git clone https://github.com/goagent/goagent.git


## 常用工具

1. 文档写作推荐 Markdown
    - 离线编辑器推荐 [haroopad](http://haroopad.userecho.com/)
    - 在线编辑器推荐 [Github-Flavored Markdonw Editor](http://jbt.github.io/markdown-editor)
2. 画图工具
    - 流程图: [yEd](http://www.yworks.com/en/products/yfiles/yed/)
    - 图像编辑: [GIMP](http://www.gimp.org/)
3. 编辑器
    - 神之编辑器: [Emacs](http://www.gnu.org/software/emacs/)
    - 编辑器之神: [Vim](http://www.vim.org/)
4. 最好的笔记与任务管理工具(没有之一): [org-mode](http://orgmode.org/)
5. 数据分析: [R](http://cran.r-project.org/)

{% highlight bash %}
svn co https://svn.r-project.org/R/trunk R
cd R
./configure --prefix=/usr
make
sudo make install
{% endhighlight %}

<hr><br>
以上基本上是构建 [Ubuntu][ubuntu] 工作环境最基本最基本的步骤（当然我们是根据需求随时 apt-get 的），整个时间不会超过1个小时，是不是比某软的系统重装一次要爽得多呢，哈哈～

[docker]:         https://docker.com
[ubuntu]:         http://ubuntu.com
