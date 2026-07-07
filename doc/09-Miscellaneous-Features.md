# 第 11 步：杂项特性

有些特性不太适合放入主线教程，或重要性不足以在主线教程中专门介绍，但仍值得一提。这些练习收集了其中一部分特性，可视为"附加内容"。

CMake 中有许多教程未涵盖的特性，其中一些对使用它们的项目而言至关重要；另一些则被打包方广泛使用，但在产出本地构建的软件开发者中鲜有讨论。

本列表并非对 CMake 剩余能力的详尽讨论，它可能随时间与相关性变化而增减。

## 练习 1：目标别名

本教程重点关注安装依赖并从安装树中使用它们，也推荐使用包管理器来简化此过程。然而，出于历史和现实的各种原因，CMake 项目的使用方式并非总是如此。

可以将依赖的源代码完全放入父项目中，并通过 `add_subdirectory()` 来使用它。这样做时，暴露出来的目标名称是该项目内部使用的名称，而非通过 `install(EXPORT)` 导出的名称，目标名称也不会带有该命令为目标添加的命名空间前缀字符串。

有些项目希望以与 `find_package()` 使用者所见一致的接口来支持这种工作流。CMake 通过 `add_library(ALIAS)` 和 `add_executable(ALIAS)` 提供了这种支持。

```cmake
add_library(MyLib INTERFACE)
add_library(MyProject::MyLib ALIAS MyLib)
```

### 目标

为 `MathFunctions` 库添加一个库别名。

### 参考资源

- `add_library()`

### 待编辑文件

- `TutorialProject/MathFunctions/CMakeLists.txt`

### 入门指引

本步骤中我们只会编辑 `Step11` 文件夹中的 `TutorialProject` 项目。请完成 `TODO 1`。

### 构建与运行

要构建该项目，我们首先需要配置并安装 `SimpleTest`。进入 `Help/guide/Step11/SimpleTest` 并运行相应命令。

```console
cmake --preset tutorial
cmake --install build
```

然后进入 `Help/guide/Step11/TutorialProject`，执行通常的构建。

```console
cmake --preset tutorial
cmake --build build
```

添加别名后行为上不应有可观察的变化。

### 解答

我们在 `MathFunctions` 的 CML 中添加一行。

TODO 1: TutorialProject/MathFunctions/CMakeLists.txt

```cmake
add_library(Tutorial::MathFunctions ALIAS MathFunctions)
```

## 练习 2：生成器表达式

`Generator expressions` 是 CMake 在某些上下文中支持的一种复杂的领域特定语言。最简单的理解方式是把它们看作延迟求值的条件表达式，用于表达在 CMake 配置阶段尚无法确定正确行为的需求。

> **注：** 生成器表达式之名正是由此而来——它们在底层构建系统被生成时才求值。

生成器表达式曾常与 `target_include_directories()` 结合使用，以表达跨构建树和安装树的包含目录需求，但 file sets 已取代这一用例。如今它们最常见的应用在于多配置生成器以及复杂的依赖注入系统。

```cmake
target_compile_definitions(MyApp PRIVATE "MYAPP_BUILD_CONFIG=$<CONFIG>")
```

### 目标

为 `SimpleTest` 添加一个生成器表达式，在编译定义中检查构建配置。

### 参考资源

- `target_compile_definitions()`
- `cmake-generator-expressions(7)`

### 待编辑文件

- `SimpleTest/CMakeLists.txt`

### 入门指引

本步骤中我们只会编辑 `Step11` 文件夹中的 `SimpleTest` 项目。请完成 `TODO 2`。

### 构建与运行

要构建该项目，我们首先需要配置并安装 `SimpleTest`。进入 `Help/guide/Step11/SimpleTest` 并运行相应命令。

```console
cmake --preset tutorial
cmake --install build
```

然后进入 `Help/guide/Step11/TutorialProject`，执行通常的构建。

```console
cmake --preset tutorial
cmake --build build
```

直接运行 `TestMathFunctions` 二进制时，我们应看到一条消息，指明用于构建该可执行文件的构建配置（不一定与配置 `SimpleTest` 时使用的配置相同）。在单配置生成器上，可以通过设置 `CMAKE_BUILD_TYPE` 来更改构建配置。

### 解答

我们在 `SimpleTest` 的 CML 中添加一行。

TODO 2: SimpleTest/CMakeLists.txt

```cmake
target_compile_definitions(SimpleTest INTERFACE "SIMPLETEST_CONFIG=$<CONFIG>")
```
