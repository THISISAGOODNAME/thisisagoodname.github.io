---
layout: post
title: "Heroku上搭建Snova服务器"
description: "用Heroku架梯子"
category: 开发环境
tags: [开发环境]
---

* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;Heroku是一个支持多种编程语言的云平台即服务。在2010年被Salesforce.com收购。Heroku作为最开始的云平台之一，从2007年6月起开发，当时它仅支持Ruby，但后来增加了对Java、Node.js、Scala、Clojure、Python以及（未记录在正式文件上）PHP和Perl的支持。基础操作系统是Debian，在最新的堆栈则是基于Debian的Ubuntu。

&#160; &#160; &#160; &#160;本次使用的snova，也是一个j2ee serve(snova也有go和nodejs版，但网上评论一般是java版的好用)。我将要在vagrant中进行配置，vagrant的一些相关内容可以参考[我以前的一篇文章](http://aicdg.com/%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/2015/05/13/vagrantbasic.html)
<!-- more -->

##1.创建一个HROEKU账号

&#160; &#160; &#160; &#160;在这个[heroku页面](https://signup.heroku.com/)，输入你的邮箱地址并注册一个帐号。

##2.安装HEROKUTOOLBELT

&#160; &#160; &#160; &#160;注册成功后，进入后台管理界面，新建一个java应用，安装HEROKUTOOLBELT版本选择debian/ubuntu，

出现如下内容:

<pre><code>
# Run this from your terminal:
# Please ensure that you have Ruby installed.
wget -qO- https://toolbelt.heroku.com/install-ubuntu.sh | sh
</code></pre>

在命令行运行`wget -qO- https://toolbelt.heroku.com/install-ubuntu.sh | sh`

安装完成后登陆Heroku

<pre><code>
$ heroku login
Enter your Heroku credentials.
Email: java@example.com
Password:
</code></pre>

##3.下载snova客户端和服务器端软件

[下载链接](https://code.google.com/p/snova/downloads/list)在此,server端程序下载snova-c4-java-server-0.22.0.war

然后把这个程序移动到本机上的vagrant的dev目录(也是vagrant虚拟机的/vagrant目录)

##4.部署snova服务器端到heroku

&#160; &#160; &#160; &#160;安装heroku-deploy插件,只需执行一次，以后不用执行

<pre><code>
$ heroku plugins:install https://github.com/heroku/heroku-deploy
</code></pre>


<pre><code>
$ heroku apps:create APP
</code></pre>

此步会创建一个app，APP是你的 PIDAPP可以是任意字符字母数字组合(字母必须全部小写)，但是不能和别人重名，别人已经使用的PID就不行了

然后会提示 创建成功 app.herokuapp.com ,在浏览器输入你的，看到欢迎页。

上传war文件

<pre><code>
heroku deploy:war --war snova-c4-java-server-0.22.0.war --app APP
</code></pre>

打开[http://abc.herokuapp.com/](http://abc.herokuapp.com/)，如果出现Snova C4 Server 0.21.0.1，证明成功，之后的配置就和goagent一样了，不过客户端记得要用步骤三里下载的那个客户端。

客户端配置

<pre><code>
[LocalServer]
Listen=192.168.1.10:48100（这点很重要...就是路由器给你分配的IP+:48100）
[GAE]
Enable=0
[C4]
Enable=1
WorkerNode[0]=app.herokuapp.com
WorkerNode[1]=app2.herokuapp.com （如果你创建了第二个app，名叫abcdef2。 可以此格式加多个。）
</code></pre>