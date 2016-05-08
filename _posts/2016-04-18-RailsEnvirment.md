---
layout: post
title:  "Ubuntu下Rails的环境搭建"
date:   2016-04-18 19:34:00
categories: ubuntu rails
---

以下是本人根据自己的需求对ubuntu/os x系统的定制安装。

## 准备
#### 裸机环境
* ubuntu 14.04 32位版本（aliyun经典网络）.
* 能远程连接到Terminal，但无GUI.

#### 远程连接
* 通过ssh远程登陆上去：`ssh root@XXX.XXX.XXX`(其中root是远程系统的用户名，后面的是远程系统IP地址)，如遇到以下情况

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
按提示进入文件（/Users/xiao_dli/.ssh/known_hosts）删除与此IP相关的行即可.

* 新建用户，不要在root用户下操作:

```shell
adduser denley
echo 'denley ALL=(ALL) ALL' >> /etc/sudoers #解决is not in sudoers file的问题
```
* 新用户登陆：`login denley`

#### 系统更新
执行：

```shell
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
```

#### 其它约定
* 所有操作都是在非root用户下操作.
* 统一使用配置文件: `~/.bash_profile`

```shell
/etc/profile #全局用户的公共部分，只在每个用户登陆时执行一次
/etc/bashrc #全局用户的定制部分, 但每打开bash shell就执行一次
~/.bash_profile #单个用户的公共部分，只在此用户登陆时执行一次
~/.bashrc #单个用户的定制部分，但每打开bash shell就执行一次
```
注：执行： `. $CONFILE_FILE`可以立即生效.

## 安装RVM
类似的还有rbenv, 主要是本人做Ruby开发需要，操作参考[在ubuntu中安装及使用rvm管理ruby版本](http://blog.csdn.net/abbuggy/article/details/8170899).

* 检查是否安装curl: `dpkg -s curl`
* 如没安装：`sudo apt-get install curl`
* 执行: `curl -L get.rvm.io | bash -s stable`

如出现错误：

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

按提示执行：

```shell
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
```

* 为便于在开启一个终端会话时加载RVM，在文件（$HOME/.bash_profile)中加入：

```shell
 [[ -s "$HOME/.rvm/scripts/rvm" ]] && . "$HOME/.rvm/scripts/rvm" # This loads RVM into a shell session.
```

注：好象安装后已加入，但要自己执行下才生效（或者重新登陆）.

* （可选）察看安装各版本时候的前提条件,执行：`rvm requirements`
* （可选）更新到当前版本：`rvm get head`

## 安装Ruby
* （可选）为提高安装速度，更新Ruby源：

  ```shell
  # Ruby China源
  sed -i -e 's/cache\.ruby-lang\.org\//cache\.ruby-china\.org\//g' ~/.rvm/config/db
  ```

  或

  ```shell
  # 淘宝源
  sed -i -E 's!https?://cache.ruby-lang.org/pub/ruby!https://ruby.taobao.org/mirrors/ruby!' ~/.rvm/config/db
  ```

  其中可能提示文件不存在，执行：

  ```shell
  mkdir ~/.rvm
  mkdir ~/.rvm/config
  touch db
  ```

* 安装最新版本，执行：`rvm install ruby-head`,

  使用前面设置的Ruby源，果然速度明显.

  目前最新的是2.3.0, 同时安装的还有rubygems(gem).

* 设置当前Ruby: `rvm use ruby-2.3.0 --default`

## 安装Rails
前提：已经安装了gem

* （可选）为提高安装速度，更新gem源，参见[RubyGems 镜像（淘宝)](https://ruby.taobao.org/)：

```shell
gem sources --add https://ruby.taobao.org/ --remove https://rubygems.org/
```

* （可选）不修改自动生成的Gemfile中的source而自动替换gem源,执行命令：

```shell
bundle config mirror.https://rubygems.org https://ruby.taobao.org
```

可能会提示没有bundle命令，执行: `gem install bundle`.

* （可选）关闭gem doc的安装: 在文件$HOME/.gemrc最后增加一行：
  `gem: --no-ri --no-rdoc`
* 更新gem: `gem update --system`
* 安装最新的rails: `gem install rails --pre`
* (可选)删除历史版本: `gem clean`

## 安装brew
前提：已经安装了ruby

* 执行：`ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Linuxbrew/linuxbrew/go/install)"`
* （可选）安装cask:

```shell
brew install cask
brew cask
```

附：

* [Linux下的Brew安装](http://linuxbrew.sh)
* [Mac OS下的Brew安装](http://brew.sh)

## 安装代理服务器---nginx
两种安装方式，不过方式2目前没有成功过。
#### 方式1---apt-get
* 执行：

```shell
sudo add-apt-repository ppa:nginx/stable
sudo apt-get update
sudo apt-get install nginx
```

* 配置反向代理：修改文件`/etc/nginx/sites-available/default`或 `/etc/nginx/nginx.confg`

```
upstream wwww.domain.com {
  server 0.0.0.0:3000 max_fails=3 fail_timeout=3s
  server 0.0.0.0:3001 max_fails=3 fail_timeout=3s
}
location / {
  proxy_pass http://www.domain.com;
}
```

#### 方式2---brew
前提: 已经安装了brew

* 执行： `brew install nginx`
当出现下列错误时，参考[ubuntu证书配置](http://www.th7.cn/Program/python/201406/217325.shtml)， 其中的*.crt文件从[根证书](https://curl.haxx.se/ca/cacert.pem)下载更名得到:

```shell
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: http://curl.haxx.se/docs/sslcerts.html
```

当出现ld(GNU linker)文件不存在时，是因为优先使用了glibc(brew install glibc)中的ld,修改PATH环境变量.

## 安装Git服务器---gitblit
前提：安装java(`sudo apt-get install openjdk-7-jre`),
可参考：[在Ubuntu/Fedora/CentOS中安装Gitblit](http://www.open-open.com/lib/view/open1427258474981.html)。

* 下载：`wget http://dl.bintray.com/gitblit/releases/gitblit-1.6.2.tar.gz`
* 解压：`tar xvf gitblit-1.6.2.tar.gz $GITBLIT_PATH`(其中$GITBLIT_PATH代表gitblit解压的位置，下同)
* 修改文件`$GITBLIT_PATH/data/gitblit.properties`：

```shell
git.repositoriesFolder = path/to/repo # 仓库所在位置
web.authenticateViewPages = true # 打开权限开关
server.httpPort = XXXX #开启端口号
server.httpBindInterface = XXX.XXX.XXX.XXX  #当前服务器IP地址
```

* 启动：`. $GITBLIT_PATH/gitblit.sh`

如果要以服务启动，则：

* 修改文件：`$GITBLIT_PATH/service-ubuntu.sh`

```shell
GITBLIT_PATH=$GITBLIT_PATH
GITBLIT_BASE_FOLDER=$GITBLIT_PATH/data
GITBLIT_USER="denley"
```

* 安装服务：`sudo ./install-service-ubuntu.sh`
* 启动服务：`sudo service gitblit start`

## 自启动Rails app
[Ruby on Rails 终极部署方案 nginx+mina+puma](https://segmentfault.com/a/1190000002918225)

注：[Ruby Thin服务器](http://koda.iteye.com/blog/1162276)

## RubyMotion
只适用于OS X操作系统下，参考其[官网](http://www.rubymotion.com/cn),以下以Android开发为例。

* 安装Motion: 从网站下载安装，此处下载的是免费版，只支持iOS,Android上的开发（而且仅限于最新版本的SDK)。
* 安装SDK、模拟器（Image后缀的):（可以选择安装其它版本）
`motion android-setup`
* 创建、启动模拟器:（在上面选择安装中，通过菜单）
`Tools > Manage AVDs (Create/Start)``
* 创建项目: `motion create --template=android PROJECT_NAME`
* 运行项目:

```shell
rake         # 模拟器启动的前提下
rake device  # 设备连接的前提下
```

* 更多操作执行: `motion help/motion create --help`
