---
layout: post
title: VIM8插件整理
slug: vim8_plug
date: 2020-02-14 20:20
status: publish
author: 
tags:
  - VIM
  - C++
  - clangd
  - llvm
  - gtags
  - universal-ctags
excerpt: VIM8的插件安装、配置等，C++开发相关
---

## 前言

已经到了2020年，VIM很多插件都是基于8.0之后的异步特性，迭代出了新的效果。

本文围绕C++开发，挑选了一些知名插件，重做整理。

因为要考虑封闭网络内的开发环境，方便更新拷贝，插件尽量选择简而精、且持续维护的老牌插件，花里胡哨的功能能不要就不要，务求 **简洁耐用** 。

-----

## 系统环境

* Windows 10 1809
* [CentWSL 7.6](https://github.com/yuk7/CentWSL)

## 依赖

### VIM

使用 **VIM8.2** 以上版本，必须确保`vim --version`中的feature里带有 **+python3**。

*因2020-01-01后，python2停止维护，建议开发机上也尽量切换到python3环境，以避免混乱。*

详见[VIM8安装](../vim8_install)。

### 插件管理器

因为要考虑在内网环境下使用，很多常用的如Vundle、[vim-plug](https://github.com/junegunn/vim-plug)等插件管理器的下载和更新功能，都难以发挥作用。

考察过VIM8之后的packages功能后，决定采用之（毕竟是VIM自带新特性，还省掉一个插件）。

再利用`git submodule`按照基于packages的目录布局，上传github。插件目录布局大概如下：

```bash
$ tree -L 4
.
├── pack
│   └── default
│       ├── opt
│       │   ├── vim-cpp-enhanced-highlight
│       │   └── YouCompleteMe
│       └── start
│           ├── gruvbox
│           ├── LeaderF
│           ├── nerdtree
│           ├── tagbar
│           └── vim-airline
...
```

clone或更新到本地后直接使用即可，暂无问题。

packages的特性还在摸索，今后会持续关注。

### ripgrep

LeaderF推荐的grep替代品，真的是快出天际的速度，墙裂推荐！

安装方法参照官方[ripgrep](https://github.com/BurntSushi/ripgrep#installation)。

### universal-ctags

老Exuberant-Ctags的替代品，C++项目必备，很多插件都需要有这个。

*yum源上的老版ctags不要使用。*

安装方法参照官方[universal-ctags](https://github.com/universal-ctags/ctags)。

### GNU Global

gtags + pygments搭配使用用来生成引用tags，C++项目必备，也是老牌知名工具了。

*gtags是Global的一个生成tag的工具，pygments是第三方的一套扩展，用以支持更多语言的tag生成。*

安装后记得安装pygments：`pip3 install pygments`。

*yum源上的版本太老，建议从源码编译安装。*

安装方法参照官方[GNU Global](https://www.gnu.org/software/global/download.html)

### LLVM9

供YouCompleteMe的C++语义补全使用，强烈推荐9.0之后的版本，以避免奇奇怪怪的问题。

*LLVM的完整编译比较费时费力，如果接受不了，YCM自带的符号补全其实已经很不错。*

详见[LLVM9安装](../llvm9_install)。

## 插件安装

得益于vim8自带的packages特性，插件目录只需下载到`$HOME/.vim/pack/default/start/`下即可自动加载，无需额外配置。

如果是可选插件（在vimrc中通过`packadd`命令加载），可以放到`$HOME/.vim/pack/default/opt/`下。

*详见vim8帮助文档，`:help packages`。*

### [gruvbox](https://github.com/morhetz/gruvbox)

备受欢迎的颜色主题，下载即用，不用配置。

喜欢暗色背景的（比如我）可以设置如下：

```bash
set background=dark
```

### [NERDTree](https://github.com/preservim/nerdtree)

老牌文件管理插件了。

设置快捷键映射如下：

```bash
nnoremap <F3> :NERDTreeToggle<CR>
```

进一步可以自行搭配其他衍生插件，比如[nerdtree-git-plugin](https://github.com/Xuyuanp/nerdtree-git-plugin)等

### [tagbar](https://github.com/majutsushi/tagbar)

显示文件大纲的经典插件。

设置快捷键映射如下：

```bash
nnoremap <F4> :TagbarToggle<CR>
```

注意需要先安装`ctags`。

### [AirLine](https://github.com/vim-airline/vim-airline)

状态栏美化，扩展和衍生插件很多，使用方法见仁见智。

### [LeaderF](https://github.com/Yggdroot/LeaderF)

国人设计的模糊查找插件。

查找文件（类似CtrlP）、查找字符串（相当于grep）、甚至最近开始支持的查找tag（基于gtags）。功能越来越全，性能出色，强烈推荐！

注意使用gtags查找tag的时候，最好设置：

```bash
let g:Lf_Gtagslabel = 'pygments'
```

对查定义和查引用都有不错的支持（尤其对C++新标准）。

相比之下，`'native'`对C++新标准支持不好（一些关键字识不准）；`'new-ctags'`虽然使用了`universal-tags`但不支持引用。综合下来建议还是使用`'pygments'`。

记得先安装**ripgrep**，LeaderF使用其`rg`命令来查找，比grep快很多，在巨型项目中也有良好表现。

### [YouCompleteMe](https://github.com/ycm-core/YouCompleteMe)

大名鼎鼎的自动补全插件，最好用的C++补全，没有之一。

缺点是安装LLVM很不省心，尤其是在内网开发机上，只为了一个语义补全，就要编译几个G的LLVM，令人不爽。

我个人的感受是，使用LeaderF查找定义和引用，YCM仅作符号补全放弃语义补全，依然可以接受。

如果要使用C++语义补全，YCM支持两种语义分析引擎（都需要已安装**LLVM9**）：

* libclang。安装时执行：

    ```bash
    LD_LIBRARY_PATH=/path/to/llvm9/lib/:$LD_LIBRARY_PATH python3 install.py --clang-completer --system-libclang
    ```

    实测发现该方法在较大项目中表现不佳。
* clangd（实验中）。安装时直接执行：

    ```bash
    python3 install.py
    ```

    然后在vimrc中设置如下：

    ```bash
    let g:ycm_use_clangd = 1
    let g:ycm_clangd_binary_path = '/path/to/llvm9/bin/clangd'
    ```

    虽然该引擎还处于experiment阶段，不过总体表现好于libclang，并且安装时无需额外依赖，可以安装完随时开关，推荐使用该方法。

总体来说，YCM的C++语义补全，在项目较小时表现尚可，大项目都不甚理想，速度慢、崩溃时有发生，目前仅能算是个锦上添花。

### [cpp enhanced highlight](https://github.com/octol/vim-cpp-enhanced-highlight)

针对C++语言专门做的高亮插件，不完美，但是可以用，另外不要指望在模板方面有出色表现。

## VIM配置

详见[我的自用配置](https://github.com/fly-potato-100/config4vim)中的vimrc。
