---
title: Clangd工作原理
subtitle:
date: 2024-08-17T23:53:10+08:00
description:
keywords:
license:
weight: 0
tags:
  - 编译
categories:
  - 编译
summary:
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: false
lightgallery: false


# See details front matter: https://fixit.lruihao.cn/documentation/content-management/introduction/#front-matter
---

<!--more-->


## clangd的工作原理

clangd 是一个基于 Clang 的语言服务器，实现了语言服务器协议（LSP），用于提供 C/C++ 项目的智能代码编辑功能，如代码补全、跳转到定义、查找引用等。 clangd 的工作流程如下:

### 启动和初始化
在启动VS code时(VS code内部支持LSP), 如果clangd已经安装, 那么就会自动启动, VS code和clangd通过LSP协议进行通信, clangd会进行初始化, 包括读取配置文件、加载索引和缓存等。

### 读取配置文件
clangd 会读取配置文件，读取顺序为.clangd 文件, compile_commands.json 文件, clang-tidy 配置, clang-format 配置.

	•	clangd 首先加载 .clangd 文件，这个文件配置了 clangd 的基本行为和选项。
	•	compile_commands.json 紧随其后被加载，用于提供项目的具体编译信息。它在 clangd 的配置文件之后加载，但对 clangd 的核心功能影响很大，因为它定义了如何解析项目代码。
	•	clang-tidy 和 clang-format 的配置在此之后加载，并影响代码的静态分析和格式化。

### 索引和缓存
clangd 会索引项目中的源文件，并将索引信息存储在缓存中。索引过程包括解析源文件、提取符号信息、构建符号之间的依赖关系等。索引信息可以加速后续的代码分析，如跳转到定义、查找引用等操作。

在索引时, vscode中可以看到左下角的index, 还会显示进度. 索引完成后, 这个提示会消失.

### 处理编辑器请求
当编辑器发出请求时，如代码补全、跳转到定义、查找引用等，clangd 会根据请求类型和索引信息进行处理。处理结果会以 JSON 格式返回给编辑器，编辑器会根据结果进行相应的操作，如显示补全列表、跳转到定义等。

### 处理动态变化
主要涉及的动作有两个

	•	文件变化检测: clangd 会监视项目中的文件变化。当文件被修改、添加或删除时，它会自动更新索引和缓存，以确保提供的智能编辑功能始终是最新的。
	•	重新加载编译数据库: 如果 compile_commands.json 文件发生变化，clangd 会自动重新加载它，并根据新的编译选项更新编译上下文和索引。

### 关闭
在关闭vscode的时候,clangd 会自动停止运行，并清理内存中加载的缓存和索引数据。不过，clangd 通常会将索引结果持久化到磁盘，以便在下次启动时加快加载速度。
