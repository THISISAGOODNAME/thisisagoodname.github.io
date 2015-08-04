---
layout: post
title: "Ruby开发环境配置"
description: "在vagrant上配置ruby开发环境"
category: 开发环境
tags: [开发环境]
---

* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;最近想要在heroku上配置一个静态 web 应用，heroku 的开发环境配置需要 ruby，所以把在 vagrant 上配置ruby的过程记录一下，其实和 Ubuntu 下配置差不多，只不过 rvm 的安装稍微有些费劲，因为官网更换了 https ,所以下载的时候需要加上 ssl
<!-- more -->

##以下是详细的开发过程的记录

###1.为rvm安装做准备

&#160; &#160; &#160; &#160;现在开始安装,RVM 脚本需要先安装好 Curl 和 Git 。

<pre><code>
$ sudo apt-get install curl
$ sudo apt-get install git-core
</code></pre>

&#160; &#160; &#160; &#160;配置 git

<pre><code>
$ git config --global user.name "Your Name"
$ git config --global user.email your-email@address.com
</code></pre>

###2.安装RVM

&#160; &#160; &#160; &#160;RVM 的意思是 Ruby 版本管理器，“是一个命令行工具，让你容易的安装、管理和使用多个 Ruby 环境及其相应的 Gem 包。可以下列命令来安装这个脚本。RVM 将安装在你当前登录用户的主目录里。

<pre><code>
$ gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
$ \curl -sSL https://get.rvm.io | bash -s stable
</code></pre>

&#160; &#160; &#160; &#160;切换到主目录，然后添加rvm scripts路径变量到bash：

<pre><code>
 $ echo '[[ -s "$HOME/.rvm/scripts/rvm" ]] && . "$HOME/.rvm/scripts/rvm" # Load RVM function' >> ~/.bash_profile
 $ source ~/.bash_profile
</code></pre>

如果是zsh用户则是

<pre><code>
 $ echo '[[ -s "$HOME/.rvm/scripts/rvm" ]] && . "$HOME/.rvm/scripts/rvm" # Load RVM function' >> ~/.zshrc
 $ source ~/.zshrc
</code></pre>

之后，就可以安装ruby了

###3.安装ruby

&#160; &#160; &#160; &#160;使用RVM命令进行安装ruby

<pre><code>
$ rvm list known        #从结果中选择一个版本进行安装
$ rvm install 2.2.1     #安装成功后通过以下命令查看版本
$ ruby -v
$ gem -v
</code></pre>

&#160; &#160; &#160; &#160;如果有需要可以手动更新下RubyGems 和其他需要更新的 Gem

<pre><code>
$ gem update --system
$ gem update
</code></pre>

###4.将gem的源替换成淘宝的源

&#160; &#160; &#160; &#160;在国内用rubygems官方源各种问题，还是最好换成国内的，推荐淘宝的源，方法如下

<pre><code>
$ gem sources --remove http://rubygems.org/
$ gem sources -a http://ruby.taobao.org/
$ gem sources -l  # 请确保只有 ruby.taobao.org
$ gem install foo
</code></pre>