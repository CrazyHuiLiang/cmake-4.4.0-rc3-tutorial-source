# CMake 官方教程（中文 + 英文）

本目录收录 CMake 官方教程 [CMake Tutorial](https://cmake.org/cmake/help/latest/guide/tutorial/index.html) 的本地化文档，分为**中文版**与**英文版**两套 Markdown，互不混排：

- 中文版：`00-Introduction.md` ~ `12-In-Depth-CMake-Library-Concepts.md`
- 英文版：`00-Introduction_en.md` ~ `12-In-Depth-CMake-Library-Concepts_en.md`
- 英文原始网页（保留原样式）：[`_original/`](_original/)

## 文件命名约定

| 后缀 | 语言 | 说明 |
|------|------|------|
| `XX-Name.md` | 中文 | 译文，正文为中文，代码块原样保留英文 |
| `XX-Name_en.md` | 英文 | 由官方 HTML 转换的纯英文 Markdown，未作翻译 |
| `_original/*.html` | 英文 | 官方原始网页，保留 Sphinx 样式，便于核对 |

## 翻译规则（中文版）

- 正文、标题、说明文字译为中文；命令、文件名、变量名、CMake 目标名与函数名保留英文。
- 代码块（`cmake`/`c++`/`console`/`text` 等）原样保留，不翻译、不增删字符，便于直接复制使用。
- "Exercise N" 译为"练习 N"；note 告示框译为 `> **注：** …`。
- 内部交叉链接改指本地 Markdown 文件；外部链接保留原 URL。

## 教程目录

| 编号 | 中文版 | 英文版 | 内容概述 |
|------|--------|--------|----------|
| 00 | [引言](00-Introduction.md) | [Introduction](00-Introduction_en.md) | 教程总体介绍与各步骤概览 |
| 01 | [开始之前](01-Before-You-Begin.md) | [Before You Begin](01-Before-You-Begin_en.md) | 获取 CMake、生成器、单/多配置生成器、获取练习源码 |
| 02 | [CMake 入门](02-Getting-Started-with-CMake.md) | [Getting Started with CMake](02-Getting-Started-with-CMake_en.md) | 构建可执行文件、库、链接、子目录组织（练习 1–4） |
| 03 | [CMake 语言基础](03-CMake-Language-Fundamentals.md) | [CMake Language Fundamentals](03-CMake-Language-Fundamentals_en.md) | 宏/函数与列表、条件与循环、include 组织（练习 1–3） |
| 04 | [配置与缓存变量](04-Configuration-and-Cache-Variables.md) | [Configuration and Cache Variables](04-Configuration-and-Cache-Variables_en.md) | option、CMake 变量、CMakePresets.json（练习 1–3） |
| 05 | [自定义命令与生成文件](05-Custom-Commands-and-Generated-Files.md) | [Custom Commands and Generated Files](05-Custom-Commands-and-Generated-Files_en.md) | 使用代码生成器（练习 1） |
| 06 | [测试与 CTest](06-Testing-and-CTest.md) | [Testing and CTest](06-Testing-and-CTest_en.md) | 用 CTest 组织与运行测试 |
| 07 | [安装命令与概念](07-Installation-Commands-and-Concepts.md) | [Installation Commands and Concepts](07-Installation-Commands-and-Concepts_en.md) | 安装规则、导出与导入目标 |
| 08 | [查找依赖](08-Finding-Dependencies.md) | [Finding Dependencies](08-Finding-Dependencies_en.md) | find_package、传递性依赖、查找其他文件（练习 1–3） |
| 09 | [杂项功能](09-Miscellaneous-Features.md) | [Miscellaneous Features](09-Miscellaneous-Features_en.md) | 生成器表达式等附加特性 |
| 10 | [系统内省深入](10-In-Depth-System-Introspection.md) | [In-Depth System Introspection](10-In-Depth-System-Introspection_en.md) | 检测系统与平台特性 |
| 11 | [CMake 目标命令深入](11-In-Depth-CMake-Target-Commands.md) | [In-Depth CMake Target Commands](11-In-Depth-CMake-Target-Commands_en.md) | 目标属性与命令的高级用法 |
| 12 | [CMake 库概念深入](12-In-Depth-CMake-Library-Concepts.md) | [In-Depth CMake Library Concepts](12-In-Depth-CMake-Library-Concepts_en.md) | 库的进阶概念与最佳实践 |

## 配套源码

本仓库根目录提供与教程各步骤���应的源码（`Step1`–`Step11`、`Complete`），可与上述文档对照练习。

## 说明

- 中文译文由 AI 翻译，如遇与英文不一致处，以 [`_original/`](_original/) 中的官方英文页面或对应 `_en.md` 为准。
- 翻译时间：2026-07-07；对应 CMake 文档版本为 4.4.0-rc3 的最新在线版。
