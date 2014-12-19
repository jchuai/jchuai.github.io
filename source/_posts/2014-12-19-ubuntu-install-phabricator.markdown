---
layout: post
title: "Ubuntu Install Phabricator"
date: 2014-12-19 17:04:39 +0800
comments: true
categories: 
---
#背景
公司马上要部署新的version control system. 计划用Git。同时需要搭一套完整的环境，包括code review， permission control。暂时选定用[Phabricator](http://phabricator.org/). 目前有很多公司在用Phabricator，facebook转战Hg的同时也开始用。而且同时支持git/hg/svn，如果将来公司用git不爽了想用Hg，那么前端可以基本不用更改。做了两个星期的调研到底要用git还是Hg，已经没力气继续了，干脆提供Phabricator，以防止将来高层反悔了，我们也方便移植。
开始用[Bitnami](https://bitnami.com/stack/phabricator)一键安装了一个Phabricator server。结果发现Bitnami在安装时不可避免的更改了Phabricator的一些目录结构，导致后面升级更改都不方便。而且对于一键安装，作为一个系统管理员，最好避免这样偷懒，尤其是针对第三方提供的。一旦出问题就无从查起了。于是决定自己重搭环境。
<!-- more -->
#搭建步骤
##1. Ubuntu机子配置

建议采用Linux机子搭建Phabricator。官网提供了针对Linux机子的脚本，搭建很方便。首先需要更改机子Hostname。Phabricator要求server 域名必须带dot '.',所以如果不想用IP地址，那么需要更改机子的hostname,hosts并重启机器。

```
~$ sudo vi /etc/hostname
~$ sudo vi /etc/hosts
~$ sudo shutdown -r now
```
##2. Install Phabricator
Phabricator 官网提供了针对RedHat和Ubuntu的安装脚本，可以帮助安装好所需要的applications。

```
Ubuntu
~$ wget http://www.phabricator.com/rsrc/install/install_ubuntu.sh
~$ sudo sh install_ubuntu.sh

RedHat
~$ wget http://www.phabricator.com/rsrc/install/install_rhel-derivs.sh
~$ sudo sh install_rhel-derivs.sh
```
脚本会自动去下载相关的app，包括git/Apache/MySql/PHP等。此外，脚本还会git clone 相应的Phabricator的source code到当前folder。

```
$ cd somewhere/ # pick some install directory
somewhere/ $ git clone https://github.com/phacility/libphutil.git
somewhere/ $ git clone https://github.com/phacility/arcanist.git
somewhere/ $ git clone https://github.com/phacility/phabricator.git
```
因为脚本运行过程中没有报任何错误，所以就直接进入[Configure Guide](https://secure.phabricator.com/book/phabricator/article/configuration_guide/) 步骤了。
##3. Configure Apache2
在配置Apache2的时候遇到了些困难。首先我机子上的Apache是通过apt-get安装的(上面脚本运行时已经安装过apache2了)，没有用source code 本地安装。

```
~$ sudo apt-get install apache2
```

其次因为我机子的Apache2已经是最新版本的2.4。所以conf文件位置变掉了。Phabricator 官网上说在httpd.conf中配置VirtualHost，但是Apache2.4已经不用httpd.conf文件了。需要在/etc/apache2/sites-available/000-default.conf中添加VirtualHost。

```
~$ sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/000-default.conf_bak
~$ sudo vi /etc/apache2/sites-available/000-default.conf
/# Update this file following Phabricator official doc
~$ sudo /etc/init.d/apache2 restart


```
##4. Configure MySql
用以下命令确保mysql安装成功并运行

```
~$ mysql -uroot -p
```
在mysql运行情况下开始配置，

```
phabricator/ $ ./bin/storage upgrade --user <user> --password <password>
```
至此Phabricator全部配置完毕，但是还有其他的可以以管理员的身份进行配置。总的来说，因为有了官网提供的脚本，linux上配置Phabricator还是很方便的。
