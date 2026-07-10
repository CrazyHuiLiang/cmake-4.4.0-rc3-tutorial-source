# 第 7 步：自定义命令与生成的文件

代码生成是一种普遍使用的机制，用于将编程语言的能力扩展到其语言模型之外。CMake 对 Qt 的元对象编译器（Meta-Object Compiler）提供了一等支持，但很少有其他代码生成器重要到值得投入这种精力。

相反，代码生成器往往是定制化、针对特定用法的。CMake 提供了用于描述代码生成器用法的设施，使项目能够为其各自的需求添加支持。

在本步骤中，我们将使用 [`add_custom_command()`](../../command/add_custom_command.html#command:add_custom_command) 在教程项目中添加对代码生成器的支持。

## 背景

构建过程中的任何一步通常都可以用其输入和输出来描述。CMake 假定代码生成器和其他自定义过程遵循同样的原理。如此一来，代码生成器的工作方式与编译器、链接器以及工具链的其他元素完全相同：当输入比输出更新（或输出不存在）时，将运行一个由用户指定的命令来更新输出。

> **注：** 此模型假定一个过程的输出在其运行之前就是已知的。CMake 无法描述那些输出的名称和位置取决于输入*内容*的代码生成器。虽然存在各种技巧将这一功能硬塞进 CMake，但它们超出了本教程的范围。

描述一个代码生成器（或任何自定义过程）通常分两步进行。首先，独立于 CMake 目标模型来描述输入和输出，只关注生成过程本身。其次，将输出与某个 CMake 目标关联，以将其插入 CMake 目标模型中。

对于源文件，这只需将生成的文件添加到 `STATIC`、`SHARED` 或 `OBJECT` 库的源文件列表中即可。对于仅生成头文件的生成器，通常需要使用通过 [`add_custom_target()`](../../command/add_custom_target.html#command:add_custom_target) 创建的中间目标，将头文件的生成加入构建阶段（因为 `INTERFACE` 库没有构建步骤）。

## 练习 1 - 使用代码生成器

描述代码生成器的主要机制是 [`add_custom_command()`](../../command/add_custom_command.html#command:add_custom_command) 命令。对于 [`add_custom_command()`](../../command/add_custom_command.html#command:add_custom_command) 而言，“命令”要么是构建环境中可用的可执行程序，要么是一个 CMake 可执行目标名。

```cmake
add_executable(Tool)
# ...
add_custom_command(
  OUTPUT Generated.cxx
  COMMAND Tool -i input.txt -o Generated.cxx
  DEPENDS Tool input.txt
  VERBATIM
)
# ...
add_library(GeneratedObject OBJECT)
target_sources(GeneratedObject
  PRIVATE
    Generated.cxx
)
```

大部分关键字都不言自明，`VERBATIM` 除外。出于一些历史原因（在现代语境下解释起来并无趣味），该参数实际上是必填的。好奇者可查阅 [`add_custom_command()`](../../command/add_custom_command.html#command:add_custom_command) 文档以了解更多细节。

`Tool` 可执行目标同时出现在 `COMMAND` 和 `DEPENDS` 参数中。虽然仅凭 `COMMAND` 就足以让代码正确构建，但将 `Tool` 本身作为自定义命令的依赖项，可以确保当 `Tool` 更新时，自定义命令会被重新运行。

对于仅生成头文件的场景，需要额外的命令，因为库本身没有构建步骤。我们可以使用 [`add_custom_target()`](../../command/add_custom_target.html#command:add_custom_target) 为该库创建一个“人造的”构建步骤。然后，我们用 [`add_dependencies()`](../../command/add_dependencies.html#command:add_dependencies) 命令强制该自定义目标在所有链接此库的目标之前运行。

```cmake
add_custom_target(RunGenerator DEPENDS Generated.h)

add_library(GeneratedLib INTERFACE)
target_sources(GeneratedLib
  INTERFACE
    FILE_SET HEADERS
    BASE_DIRS
      ${CMAKE_CURRENT_BINARY_DIR}
    FILES
      ${CMAKE_CURRENT_BINARY_DIR}/Generated.h
)

add_dependencies(GeneratedLib RunGenerator)
```

> **注：** 我们将 [`CMAKE_CURRENT_BINARY_DIR`](../../variable/CMAKE_CURRENT_BINARY_DIR.html#variable:CMAKE_CURRENT_BINARY_DIR)（一个指明构建树中当前放置产物位置的变量）加入基础目录，因为那正是我们的代码生成器运行时所处的工作目录。列出 `FILES` 对构建并非必要，此处列出仅为清晰起见。

### 目标

向 `MathFunctions` 库添加一个预计算平方根的生成表格。

### 有用资源

- [`add_executable()`](../../command/add_executable.html#command:add_executable)
- [`add_library()`](../../command/add_library.html#command:add_library)
- [`target_sources()`](../../command/target_sources.html#command:target_sources)
- [`add_custom_command()`](../../command/add_custom_command.html#command:add_custom_command)
- [`add_custom_target()`](../../command/add_custom_target.html#command:add_custom_target)
- [`add_dependencies()`](../../command/add_dependencies.html#command:add_dependencies)

### 待编辑文件

- `MathFunctions/CMakeLists.txt`
- `MathFunctions/MakeTable/CMakeLists.txt`
- `MathFunctions/MathFunctions.cxx`

### 入门

`MathFunctions` 库已被修改为在给定小于 10 的数时使用预计算表。然而，这个硬编码的表并不十分精确，只包含最近的截断整数值。

`MakeTable.cxx` 源文件描述了一个会生成更优表格的程序。它接受单个参数作为输入，即要生成的表格的文件名。

完成 `TODO 1` 到 `TODO 10`。

### 构建与运行

无需特殊配置，按平常一样配置并构建即可。注意，`MakeTable` 可执行文件会被安排在 `MathFunctions` 之前执行。

```console
cmake --preset tutorial
cmake --build build
```

验证 `Tutorial` 的输出现在对小于 10 的值使用了预计算表。

### 解决方案

首先，我们添加一个新的可执行目标来生成表格，并将 `MakeTable.cxx` 文件添加为源文件。

TODO 1-2: MathFunctions/MakeTable/CMakeLists.txt

```cmake
add_executable(MakeTable)

target_sources(MakeTable
  PRIVATE
    MakeTable.cxx
)
```

然后，我们添加一个生成该表格的自定义命令，以及一个依赖于该表格的自定义目标。

TODO 3-4: MathFunctions/MakeTable/CMakeLists.txt

```cmake
add_custom_command(
  OUTPUT SqrtTable.h
  COMMAND MakeTable SqrtTable.h
  DEPENDS MakeTable
  VERBATIM
)

add_custom_target(RunMakeTable DEPENDS SqrtTable.h)
```

我们需要添加一个接口库，用以描述将出现在 [`CMAKE_CURRENT_BINARY_DIR`](../../variable/CMAKE_CURRENT_BINARY_DIR.html#variable:CMAKE_CURRENT_BINARY_DIR) 中的输出。`FILES` 参数是可选的。

TODO 5-6: MathFunctions/MakeTable/CMakeLists.txt

```cmake
add_library(SqrtTable INTERFACE)

target_sources(SqrtTable
  INTERFACE
    FILE_SET HEADERS
    BASE_DIRS
      ${CMAKE_CURRENT_BINARY_DIR}
    FILES
      ${CMAKE_CURRENT_BINARY_DIR}/SqrtTable.h
)
```

既然所有目标都已描述完毕，我们可以通过 [`add_dependencies()`](../../command/add_dependencies.html#command:add_dependencies) 将它们关联起来，强制该自定义目标在接口库的任何依赖者之前运行。

TODO 7: MathFunctions/MakeTable/CMakeLists.txt

```cmake
add_dependencies(SqrtTable RunMakeTable)
```

现在我们可以将该接口库添加到 `MathFunctions` 的链接库中，并将整个 `MakeTable` 文件夹加入项目。

TODO 8: MathFunctions/CMakeLists.txt

```cmake
target_link_libraries(MathFunctions
  PRIVATE
    MathLogger
    SqrtTable

  PUBLIC
    OpAdd
    OpMul
    OpSub
)
```

TODO 9: MathFunctions/CMakeLists.txt

```cmake
add_subdirectory(MakeTable)
```

最后，我们更新 `MathFunctions` 库本身，使其利用生成的表格。

TODO 10: MathFunctions/MathFunctions.cxx

```c++
#include <SqrtTable.h>

double table_sqrt(double x)
{
```
