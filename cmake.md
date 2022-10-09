1. ## 介绍

   CMake是一个管理源代码构建的工具。最初，CMake被设计为Makefile的生成器，今天CMake可以生成现代的构建系统，如Ninja，以及ide的项目文件，Visual Studio和Xcode。

   CMake广泛用于C和c++语言，但它也可以用于构建其他语言的源代码。

   第一次接触CMake的人可能有不同的初始目标，要了解如何构建从互联网下载的源代码包 可以从[用户交互指南](#1)开始。这将详细说明运行[cmake(1)](#3)或[cmake-gui(1)](#2)可执行文件所需的步骤，如何选择生成器以及如何完成构建。

   [使用依赖关系指南](#4)针对的是希望开始使用第三方库的开发人员。

   对于使用CMake开始一个项目的开发人员来说，[CMake教程](#5)是一个合适的起点，[CMake -buildsystem(7)](#6)手册旨在帮助开发人员扩展维护构建系统的知识，熟悉可以在CMake中表示的构建目标，[cmake-packages(7)](#7)手册解释了如何创建可以被基于cmake的第三方构建系统轻松使用的包。

2. 命令行工具

   - [cmake(1)](./cmake(1).md)
   - [ctest(1)](https://cmake.org/cmake/help/latest/manual/ctest.1.html)
   - [cpack(1)](https://cmake.org/cmake/help/latest/manual/cpack.1.html)

3. 交互式对话框

   - <span id = "2">cmake-gui(1)</span>
   - <span id = "3">ccmake(1)</span>

4. 参考手册

   - <span id = "6"> [cmake-buildsystem(7)](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html)</span>

   - [cmake-commands(7)](https://cmake.org/cmake/help/latest/manual/cmake-commands.7.html)
   - [cmake-compile-features(7)](https://cmake.org/cmake/help/latest/manual/cmake-compile-features.7.html)
   - [cmake-developer(7)](https://cmake.org/cmake/help/latest/manual/cmake-developer.7.html)
   - [cmake-env-variables(7)](https://cmake.org/cmake/help/latest/manual/cmake-env-variables.7.html)
   - [cmake-file-api(7)](https://cmake.org/cmake/help/latest/manual/cmake-file-api.7.html)
   - [cmake-generator-expressions(7)](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html)
   - [cmake-generators(7)](https://cmake.org/cmake/help/latest/manual/cmake-generators.7.html)
   - [cmake-language(7)](https://cmake.org/cmake/help/latest/manual/cmake-language.7.html)
   - [cmake-modules(7)](https://cmake.org/cmake/help/latest/manual/cmake-modules.7.html)
   - <span id = "7">[cmake-packages(7)](https://cmake.org/cmake/help/latest/manual/cmake-packages.7.html)</span>
   - [cmake-policies(7)](https://cmake.org/cmake/help/latest/manual/cmake-policies.7.html)
   - [cmake-presets(7)](https://cmake.org/cmake/help/latest/manual/cmake-presets.7.html)
   - [cmake-properties(7)](https://cmake.org/cmake/help/latest/manual/cmake-properties.7.html)
   - [cmake-qt(7)](https://cmake.org/cmake/help/latest/manual/cmake-qt.7.html)
   - [cmake-server(7)](https://cmake.org/cmake/help/latest/manual/cmake-server.7.html)
   - [cmake-toolchains(7)](https://cmake.org/cmake/help/latest/manual/cmake-toolchains.7.html)
   - [cmake-variables(7)](https://cmake.org/cmake/help/latest/manual/cmake-variables.7.html)
   - [cpack-generators(7)](https://cmake.org/cmake/help/latest/manual/cpack-generators.7.html)

5. 指导

   - [<span id = "5">CMake教程</span>](./CMake教程.md)
   - <span id = "1">用户交互指南</span>
   - <span id = "4">使用依赖项指南</span>
   - 导出和导入指南
   - IDE集成指南

6. 发行说明

7. 索引和搜索

   https://cmake.org/cmake/help/latest/

8. 啊