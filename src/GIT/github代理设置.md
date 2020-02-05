---
layout: post
title: github代理设置
slug: github_proxy_setup
date: 2020-01-30 22:50
status: publish
author: 
tags:
  - git
  - github
  - 科学上网
excerpt: 为github配置代理以提高访问速度
---

[notice]假定代理地址为`socks5://127.0.0.1:10808`[/notice]

## HTTP/HTTPS方式

*针对`https://github.com/xxxx/xx.git`这样的地址。*

使用git本身的配置即可实现。

```bash
### 设置全局代理
git config --global http.proxy 'socks5://127.0.0.1:10808'
git config --global https.proxy 'socks5://127.0.0.1:10808'

### 只对github.com
git config --global http.https://github.com.proxy 'socks5://127.0.0.1:10808'
git config --global https.https://github.com.proxy 'socks5://127.0.0.1:10808'

### 取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```

## SSH方式

*针对`git@github.com:xxxx/xx.git`这样的地址。*

使用`~/.ssh/config`的`ProxyCommand`配置实现。

如果22端口被墙，可以使用[Using SSH over the HTTPS port](https://help.github.com/en/github/authenticating-to-github/using-ssh-over-the-https-port)的方式实现，设置`Hostname ssh.github.com`及`Port 443`即可。

### Linux/WSL

Linux或WSL下，需要结合`nc`工具使用。增加如下配置：

```text
Host github.com
    User git
    Port 22
    Hostname github.com
    ProxyCommand nc --proxy-type socks5 --proxy 127.0.0.1:10808 %h %p
```

### Windows

Windows下，需要结合`git bash`自带的`connect`工具使用。增加如下配置：

```text
Host github.com
    User git
    Port 22
    Hostname github.com
    ProxyCommand connect -S 127.0.0.1:10808 %h %p
```

## GIT协议方式

*针对`git://github.com/xxxx/xx.git`这样的地址。*

待定
