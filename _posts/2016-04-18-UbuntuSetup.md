---
layout: post
title:  "Ubuntu系统的定制安装"
date:   2016-04-18 19:34:00
categories: ubuntu
---

以下是本人根据自己的需求对ubuntu系统的定制安装。
## 准备
#### 裸机环境
* ubuntu 14.04 32位版本（aliyun经典网络）.
* 能远程连接到Terminal，但无GUI.

#### 远程连接
先通过ssh远程登陆上去：`ssh root@XXX.XXX.XXX`(其中root是远程系统的用户名，后面的是远程系统IP地址).

如遇到以下情况，按提示进入文件（/Users/xiao_dli/.ssh/known_hosts）删除与此IP相关的行即可.
```shell
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
SHA256:S/28qNIldH9WdvFWplW38sew9iMfr+GYEr2KGfDtR4M.
Please contact your system administrator.
Add correct host key in /Users/xiao_dli/.ssh/known_hosts to get rid of this message.
Offending RSA key in /Users/xiao_dli/.ssh/known_hosts:1
RSA host key for XXX.XXX.XXX.XXX has changed and you have requested strict checking.
Host key verification failed.
```

#### 系统更新
执行命令：`apt-get update`

## 安装RVM(rbenv亦可)
主要是本人做RUBY开发需要，操作参考[在ubuntu中安装及使用rvm管理ruby版本](http://blog.csdn.net/abbuggy/article/details/8170899).
* 检查是否安装curl: dpkg -s curl
* 如没安装：sudo apt-get install curl
* curl -L get.rvm.io | bash -s stable

  如出现如下错误，按提示执行：`gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3`
>
```shell
ppg: Can't check signature: public key not found
Warning, RVM 1.26.0 introduces signed releases and automated check of signatures when GPG software found.
Assuming you trust Michal Papis import the mpapis public key (downloading the signatures).

 GPG signature verification failed for '/usr/local/rvm/archives/rvm-1.27.0.tgz' - 'https://github.com/rvm/rvm/releases/download/1.27.0/1.27.0.tar.gz.asc'!
try downloading the signatures:

    gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3

 or if it fails:

    command curl -sSL https://rvm.io/mpapis.asc | gpg --import -
```

* 为便于在开启一个终端会话时加载RVM，在文件（$HOME/.bash_profile)中加入：
>
```shell
 [[ -s "$HOME/.rvm/scripts/rvm" ]] && . "$HOME/.rvm/scripts/rvm" # This loads RVM into a shell session.
```

  记得要重新打开终端才会生效

* （可选）察看安装各版本时候的前提条件,执行：`rvm requirements`
* （可选）更新到当前版本：`rvm get head`

## 安装Ruby
* （可选）为提高安装速度，更新Ruby源：`sed -i -e 's/cache\.ruby-lang\.org\//cache\.ruby-china\.org\//g' ~/.rvm/config/db`, 这里使用的Ruby China源，其实使用[RubyGems 镜像（淘宝)](https://ruby.taobao.org/)源也是可以的.

  其中可能提示文件不存在，执行：
>
```shell
mkdir ~/.rvm
mkdir ~/.rvm/config
touch db
```
* 安装最新版本，执行命令：`rvm install ruby-head`（使用前面设置的Ruby China源，果然速度飞快）

  目前最新的是2.4.0dev.同时安装的还有rubygems(gem).
## 安装Rails
前提：已经安装了gem
* （可选）为提高安装速度，更新gem源，参见[RubyGems 镜像（淘宝)](https://ruby.taobao.org/)：
```shell
gem sources --add https://ruby.taobao.org/ --remove https://rubygems.org/
```

* （可选）不修改自动生成的Gemfile中的source而自动替换gem源,执行命令：
```shell
`bundle config mirror.https://rubygems.org https://ruby.taobao.org`
```

  可能会提示没有bundle命令，执行`gem install bundle`(可参考下一条关闭doc安装).
* （可选）关闭gem doc的安装: 在文件$HOME/.gemrc最后增加一行：
  `gem: --no-ri --no-rdoc`
* 更新gem: `gem update --system`
* 安装最新的rails: `gem install rails --pre`
* (可选)删除历史版本: `gem clean`

## 安装brew
前提：已经安装了ruby， 参考[Homebrew官网](http://brew.sh/index_zh-cn.html)
* 执行：`ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
