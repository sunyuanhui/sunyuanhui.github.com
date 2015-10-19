---
layout: post
title: CMake Introduction
category : CMake
tags : [CMake]
---
{% include JB/setup %}

### 一句话介绍CMake
CMake是“cross platform make”的缩写，是个基于BSD授权协议发布的开源跨平台自动化构建系统，由 Kitware 开发与维护。

### CMake的起源
CMake 是kitware公司以及一些开源开发者在开发几个工具套件(VTK)的过程中所产生的衍生品。
后来经过发展，最终形成体系，在2001年成为一个独立的开放源代码项目。
其官方网站是[www.cmake.org](www.cmake.org)。

### CMake与KDE
在 KDE 3 时代，仅有极少数所谓“编译专家”才能通彻地熟悉整个 KDE 构建系统。
甚至有时候，从头开始一个独立的 KDE 项目之前需要先部署 500K 左右的 autotools 配置，最终却只是为了支持一个简单的“hello world”类型程序。
在经历了unsermake，scons以及CMake的选型和尝试之后，KDE4最终决定使用CMake作为自己的构建系统。

> 一名 KDE 开发者所言：“CMake 不会再让你构建项目时烦得只想往自己脑袋上来一枪。”

CMake的流行离不开KDE4的选择。

### 谁在用CMake？
- Netflix
- KDE
- Second Life
- MySQL
- Boost
- Clang/LLVM
- OGRE
- OpenCV

### CMake和Make的关系
虽然名字中含有 “Make”，但是 CMake 和 Unix 上常见的“Make”系统是区别的，CMake 的角色比 Make 更高阶，可以称谓“Meta-Make”。

### CMake的用武之地
CMake不直接构建出最终的软件，或者说它并不是用来编译或链接程序的，而是用来生产标准的构建文件（如 Unix Makefile 或 Visual Studio 的工程文件）。
另外CMake还支持测试（ctest）、打包（cpack）等功能。

### CMake的优势
- 跨平台，在 Linux/Unix 平台生成 makefile，在苹果平台可以生成 xcode，在 Windows 平台可以生成 MSVC 的工程文件。
- 简单的组态文件CMakeLists.txt，简单的组织方式，简单的语法。
- 可扩展，可以为 CMake 编写特定功能的模块，扩充 CMake 功能。

### 结束
所以，看完这些你了解到CMake是用来做什么的了吗？

### 参考资料
[1] [CMake入门--by维基教科书](https://zh.wikibooks.org/w/index.php?title=CMake_%E5%85%A5%E9%96%80&variant=zh-hans)

