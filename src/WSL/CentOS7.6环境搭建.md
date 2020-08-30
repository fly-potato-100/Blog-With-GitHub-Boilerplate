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

* [CentWSL 7.6](https://github.com/yuk7/CentWSL)

## 安装步骤

1. 解压下载下来的zip文件，得到`CentOS7`目录，其中包含了`CentOS7.exe`。
2. 将该目录移到希望安装的目录下，官方建议最好是`C:\`下，我尝试过安装到`D:\`，暂未发现不妥。
    * **tip** 因为办公用电脑使用了域用户管理，禁用了admin。这种情况下，安装目录必须放到`C:\Users\XXX`下，否则会因为权限不足导致安装失败。
    * **tip** 如果同一个distro希望安装多个实例，可以将`CentOS7`的目录名和目录下的`CentOS7.exe`文件名修改为其他（目录名和exe文件名保持一致），比如`MyCentOS7`、`MyCentOS7.exe`。
3. 双击exe等待安装完成。
4. 安装完成后双击exe，或者cmd中输入wsl（如果当前distro是唯一的wsl实例的话），就可以进入到CentOS的shell中了。
    * **tip** 在cmd中，输入`wslconfig /l`可以查看当前安装了哪些distro实例；输入`wslconfig /s CentOS7`可以将wsl默认实例改为CentOS7。
    * **tip** 如果是1903之后的windows版本，直接使用`wsl --help`查看相关命令。

## 一些设置和优化

[notice]如无特别说明，使用root完成下面的操作。[/notice]

### 账户管理

该distro安装后，默认进入的账户是root。建议新建用户自己的账户：
`useradd testuser -d /home/testuser`。

回到cmd中，进入CentOS7的安装目录下，可以通过命令指定进入wsl时的默认用户：
`CentOS7.exe config --default-user testuser`。

* **tip** 以默认用户进入wsl时，会自动继承Windows的PATH变量，即可以直接使用Windows系统自带的exe命令；非默认用户如果也想继承，只能手动export。

如果想以root用户登录，可以在wsl里，授予用户`sudo su -`权限（或者在root下`passwd -d root`删掉root密码，可以直接使用`su -`切换）；也可以直接在cmd中运行以下命令：
`wsl -u root`。

### WSL启动配置

wsl允许配置`/etc/wsl.conf`以在启动时完成自动配置。（详见[Automatically Configuring WSL](https://devblogs.microsoft.com/commandline/automatically-configuring-wsl/)）

为了能以正常权限将Windows下的盘符挂载到wsl中，可以采用以下wsl.conf：

```bash
[automount]
enabled = true
root = /mnt
options = "metadata,uid=1000,gid=1000,dmask=022,fmask=033"
mountFsTab = false
```

该配置会以`uid=1000,gid=1000`的属主（如果按上面的操作执行，这里`uid=1000`的用户应该正是`testuser`）挂载盘符到`/mnt`下，默认的目录权限为0755，默认的文件权限为0744。

可以自定options，但最好保证文件具备x权限，否则windows的exe可能无法在wsl中运行。

### systemctl

CentOS的wsl目前无法运行`systemctl`系列命令（据说是wsl底层问题，可能等wsl2发布后会修复）。幸运的是可以借用[docker-systemctl-replacement](https://github.com/gdraheim/docker-systemctl-replacement)实现同样效果。方法如下：

```bash
cd /usr/bin
mv systemctl systemctl.old # 备份下老的
curl https://raw.githubusercontent.com/gdraheim/docker-systemctl-replacement/master/files/docker/systemctl.py > /tmp/systemctl.py # 只需使用这个python脚本
mv /tmp/systemctl.py /usr/bin/systemctl
chmod +x /usr/bin/systemctl # 增加可执行权限
```

### systemd与开机启动

wsl本身没有开机启动的概念，所以并不会走完整的initd，systemd也不会初始化(这可能会导致ptrace不能正常工作、共享内存参数设置失败等问题)。
可以通过下面的步骤来workaround：

1. 在wsl下，新建文件`/etc/init.wsl`：

    ```bash
    #! /bin/bash
    ### 启动systemd相关初始化
    service systemd-sysctl $1
    ```

2. 保存文件，并赋予可执行权限`chmod +x /etc/init.wsl`
3. 在Windows下，`WIN+R`打开`运行`对话框，输入`shell:startup`打开`启动`目录，新建vbs脚本`start_centos_service.vbs`：

    ```vb
    Set ws = WScript.CreateObject("WScript.Shell")
    ws.run "wsl -d CentOS7 -u root /etc/init.wsl start", vbhide
    ```

4. Windows重启后，将自动执行该vbs脚本，并借助systemd-sysctl进行初始化。

### SSH

如果想用SecureCRT等终端连接到wsl，需要启用ssh。

1. 执行`ssh-keygen -A`以生成默认key，否则sshd会启动失败。
2. 配置`/etc/ssh/sshd_config`。**建议`UseDNS no`**。
3. 根据上一节提供的方法以启用`systemctl`，执行`systemctl start sshd`以启动sshd。
4. `systemctl status sshd`检查是否运行成功。
5. 如果希望开机启动sshd，可以在前述的`/etc/init.wsl`中加入`service sshd $1`

## 卸载

要卸载WSL，进入cmd，执行`wslconfig /u CentOS7`即可。

卸载完毕，如遇到提示`拒绝访问`，猜测是由于Windows无法彻底删除Linux文件系统下创建的文件的缘故。可以使用其他WSL，或者`git bash`，执行`rm -rf <CentOS7解压目录>`即可。

## 备份镜像

有时候我们需要在一台机器上配置好CentOS的环境，然后希望打包成镜像，拷贝到其他机器直接使用，可参考下如下步骤：

* 直接把以下文件和目录打成tar包（wsl自带的backup似乎不好用，不过其实质也是打成tar包）：

    ```bash
    tar -cvpzf /some/path/to/output/rootfs.tar.gz \
    --exclude=/proc \
    --exclude=/tmp/* \
    --exclude=/mnt \
    --exclude=/dev \
    --exclude=/sys \
    --exclude=/run \
    --exclude=/media \
    --exclude=/var/log/* \
    --exclude=/var/cache/* \
    --exclude=/init \
    /*
    ```

* 将生成的`rootfs.tar.gz`和`CentOS7.exe`一起压成包拷走，即可作为备份镜像。
* 使用该备份镜像的方法与使用官方包一致。
