---
layout: post
title: CentOS7.6环境搭建
slug: centos76_env_prepare
date: 2020-01-09 12:50
status: publish
author: 
tags:
  - WSL
  - CentOS
  - ssh
excerpt: 介绍WSL下，CentOS7.6的环境搭建步骤
---

## 系统环境

* Windows 10 **1809或更高**
* 启用WSL

## Distro下载

* [CentWSL](https://github.com/yuk7/CentWSL)

## 安装步骤

1. 解压下载下来的zip文件，得到`CentOS7`目录，其中包含了`CentOS7.exe`。
2. 将该目录移到希望安装的目录下，官方建议最好是`C:\`下，我尝试过安装到`D:\`，暂未发现不妥。
    * **tip1** 因为办公用电脑使用了域用户管理，禁用了admin。这种情况下，安装目录必须放到`C:\Users\XXX`下，否则会因为权限不足导致安装失败。
    * **tip2** 如果同一个Distro希望安装多个实例，可以将`CentOS7`的目录名和目录下的`CentOS7.exe`文件名修改为其他（目录名和exe文件名保持一致），比如`MyCentOS7`、`MyCentOS7.exe`。
3. 双击exe等待安装完成。
4. 安装完成后双击exe，或者cmd中输入wsl（如果当前distro是唯一的wsl实例的话），就可以进入到CentOS的shell中了。
