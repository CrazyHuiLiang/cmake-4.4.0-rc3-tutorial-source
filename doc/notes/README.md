# CMake 进阶学习笔记（中文原创）

本目录是基于 [CMake 官方文档](https://cmake.org/cmake/help/latest/) 与通用工程实践撰写的**原创**中文学习笔记，聚焦官方教程（见上级目录 [`doc/`](../)）覆盖较浅的进阶主题，用于系统补齐现代 CMake 的核心知识。

> **版权与范围说明**
> 这些笔记是围绕 CMake 公开技术知识撰写的原创讲解，不复制、不逐章翻译任何受版权保护的商业书籍内容。CMake 的命令、属性、变量与最佳实践本身属于公开技术范畴。文中示例代码均可自由使用。

## 与官方教程的关系

- 上级目录 [`doc/`](../) 是 CMake 官方 Tutorial 的逐 Step 中英对照翻译，覆盖"从零搭建一个可执行/库/测试/安装"的主线。
- 本目录 `notes/` 在主线之上，补充官方教程点到为止、但实战中高频且容易踩坑的四个主题。

## 主题目录

| 编号 | 笔记 | 内容概述 |
|------|------|----------|
| 01 | [生成器表达式](01-Generator-Expressions.md) | 为何需要生成阶段求值、`$<...>` 语法、逻辑/目标/配置/平台表达式、典型用法与陷阱 |
| 02 | [目标与属性模型](02-Targets-and-Properties.md) | `target_*` 命令体系、`INTERFACE` 属性、传递依赖、`PUBLIC`/`PRIVATE`/`INTERFACE` 的边界 |
| 03 | [依赖管理](03-Dependency-Management.md) | `find_package` 组件模式、`FetchContent`、传递性依赖、避免重复引入与版本冲突 |
| 04 | [安装导出与打包](04-Install-Export-Package.md) | `install(TARGETS/EXPORT)`、生成 `xxxConfig.cmake` 供下游 `find_package`、CPack 打包 |

## 约定

- 正文、标题、说明为中文；命令、属性、变量名、目标名保留英文。
- 代码块（`cmake`/`console` 等）原样保留，可直接复制使用。
- 示例默认基于 CMake 3.20+ 的现代写法，涉及较新特性会标注所需最低版本。
