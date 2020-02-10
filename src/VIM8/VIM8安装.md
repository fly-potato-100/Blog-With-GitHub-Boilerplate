---
layout: post
title: VIM8安装
slug: vim8_install
date: 2020-02-05 15:20
status: publish
author: 
tags:
  - VIM
excerpt: VIM8的环境准备、编译、安装等
---

## 系统环境

* Windows 10 1809
* [CentWSL 7.6](https://github.com/yuk7/CentWSL)

## 使用yum安装

*此方式有一定局限性，建议直接从源码编译。*

CentOS7自带的VIM版本，依然停留在7.4，可以通过第三方源更新到VIM8版本：

```bash
rpm -Uvh http://mirror.ghettoforge.org/distributions/gf/gf-release-latest.gf.el7.noarch.rpm
rpm --import http://mirror.ghettoforge.org/distributions/gf/RPM-GPG-KEY-gf.el7
yum --enablerepo=gf-plus update vim-*
```

该方法可以不必卸载老版本的vi，**缺点是该源的版本是8.0.3，并且自带feature不支持python3**。

## 从源码安装

*取源码分支最好选取8.1之后的版本，异步功能更健壮，很多插件也推荐。*

1. 卸载自带的vim：

    ```bash
    rpm -e --nodeps vim-minimal # 单独卸载vi而不删除sudo
    yum remove vim-* # 卸载其他vim
    ```

2. 安装编译依赖，包括但不限于：

    ```bash
    yum install ncurses-devel
    yum install python-devel python3-devel  # python插件支持
    yum install perl-devel perl-ExtUtils-Embed perl-ExtUtils-ParseXS perl-ExtUtils-XSpp perl-ExtUtils-CBuilder  # perl插件支持
    yum install lua-devel   # lua插件支持
    yum install ruby-devel  # ruby插件支持
    ```

3. 下载[VIM源码](https://github.com/vim/vim)，本例使用的tag是v8.2.0236。
4. 按下面步骤编译安装：

    ```bash
    # 进入到源码根目录下
    # 注意python和python3最好2选1，不要同时安装，以免出现奇怪的加载问题
    # 2020年之后python2停止维护了，所以这里使用python3
    ./configure --enable-multibyte --enable-cscope \
        --enable-luainterp=yes \
        --enable-perlinterp=yes \
        --enable-python3interp=yes \
        --enable-rubyinterp=yes \
        --prefix=/opt/vim   # 安装到/opt/vim下
    make    # 不要带-j，容易报错
    make install
    ```

5. 增加`/opt/vim/bin`到`$PATH`中。
6. 将`/opt/vim/bin/vim`软连接到`/usr/bin/vi`处，使`visudo`能正常工作。
7. 如要卸载，删掉`/opt/vim`目录，还原步骤5、6的操作即可。

## 制作自己的RPM包

待定
