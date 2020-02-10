---
layout: post
title: LLVM9安装
slug: llvm9_install
date: 2020-02-10 20:20
status: publish
author: 
tags:
  - clang
  - c++17
  - llvm
excerpt: LLVM9的环境准备、编译、安装等
---

## 系统环境

* Windows 10 1809
* [CentWSL 7.6](https://github.com/yuk7/CentWSL)

## 使用yum安装

scl中包含的最新版本是**llvm-toolset-7.0**，其中clang的版本是7.0.1，比较老。如果仅用来进行C++17的开发，已经足够。

若要使用最新的特性，还需从源码编译安装。

## 从源码安装

*llvm的工程迭代比较快，当前10的预览版已经发布，本文选用最新的稳定版9.0.1。*

1. 安装编译依赖，包括但不限于：

    ```bash
    yum install libedit-devel swig3
    ```

2. 下载[LLVM源码](https://github.com/llvm/llvm-project.git)，本例使用的tag是llvmorg-9.0.1。
3. 按下面步骤编译安装：

    ```bash
    # 进入到源码根目录下
    mkdir build
    cd build
    # 使用cmake安装
    # LLVM_ENABLE_PROJECTS包含了需要编译的项目，根据需要增删
    cmake -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX=/opt/llvm-9.0.1 \
        -DLLVM_TARGETS_TO_BUILD=X86 \
        -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra;libcxx;libcxxabi;libunwind;lldb;compiler-rt;lld" \
        ../llvm/
    make -j8
    make install
    ```

4. 增加`/opt/llvm-9.0.1`下的各个目录到环境变量中。
5. 如要卸载，删掉`/opt/llvm-9.0.1`目录，还原步骤5的操作即可。

## 制作自己的RPM包

待定
