# 第 1 步：CMake 入门

CMake 教程的第一步旨在帮助读者快速上手，为小型项目编写实用的 CMake 构建。完成本步后，你将能够使用 CMake 描述可执行文件、库、源文件和头文件，以及它们之间的链接关系。

本步中的每个练习都从讨论该练习所需的概念和命令开始，然后给出目标与有用资源列表。`Files to Edit` 部分列出的每个文件都位于 `Step1` 目录中，并包含一个或多个 `TODO` 注释。每个 `TODO` 代表需要修改或添加的一两行代码。`TODOs` 应按编号顺序完成，先完成 `TODO 1`，再完成 `TODO 2`，依此类推。

> **注：** 教程中的每一步都建立在前一步的基础上，但各步之间并非严格连续。与学习 CMake 无关的代码（例如 C++ 函数实现或超出本教程范围的 CMake 代码）有时会在各步之间被添加。

`Getting Started` 部分会给出一些有用的提示并引导你完成练习。随后 `Build and Run` 部分将逐步演示如何构建并测试该练习。最后，在每个练习末尾会回顾预期的解决方案。

## 背景

CMake 的典型用法围绕一个或多个名为 `CMakeLists.txt` 的文件展开。该文件有时被称为 "lists file" 或 "CML"。在一个软件项目中，任何我们希望向 CMake 提供指令以处理该目录或其子目录本地文件与操作的目录中，都会存在一个 `CMakeLists.txt`。每个 CML 由一组命令构成，这些命令描述了与构建该项目相关的信息或动作。

软件项目中并非每个目录都需要 CML，但强烈建议项目根目录包含一个。它将作为 CMake 在配置阶段进行初始设置的入口点。这个*根* CML 应当始终在文件顶部或接近顶部处包含相同的两条命令。

```cmake
cmake_minimum_required(VERSION 3.23)

project(MyProjectName)
```

`cmake_minimum_required()` 是 CMake 向项目开发者提供的兼容性保证。调用时，它确保 CMake 采用所列版本的行为。如果用更高版本的 CMake 来处理包含上述代码的 CML，它会表现得完全如同 CMake 3.23 一样。

`project()` 是一个概念上简单却功能复杂的命令。它告知 CMake 接下来的是一个具名的独立软件项目的描述（而非类似 shell 脚本）。当 CMake 看到 `project()` 命令时，会执行各种检查以确保环境适合构建软件，例如检查编译器与其他构建工具，并探测主机与目标机器的字节序等属性。

> **注：** 虽然为每条命令都提供了指向完整文档的链接，但并不期望读者理解其所用每条 CMake 命令的全部语义。高效地学习 CMake 与学习任何软件一样，是一个渐进的过程。

本教程步骤的余下部分主要关注另外四条命令的用法：用于描述软件项目想要生成的输出产物的 `add_executable()` 和 `add_library()` 命令；用于将输入文件与其各自输出产物关联的 `target_sources()` 命令；以及用于将输出产物彼此关联的 `target_link_libraries()` 命令。

这四条命令是大多数 CMake 用法的骨干。正如我们将要看到的，它们足以描述典型项目的大部分需求。

## 练习 1 - 构建一个可执行文件

最基本的 CMake 项目是从单个源代码文件构建出的可执行文件。对于这样的简单项目，只需一个包含四条命令的 `CMakeLists.txt` 文件即可。

> **注：** 虽然 CMake 支持大写、小写和大小写混合的命令，但推荐使用小写命令，本教程全程使用小写。

前两条命令我们已经介绍过了：`cmake_minimum_required()` 和 `project()`。在根 CML 中，第一条命令无一例外都是 `cmake_minimum_required()`。在某些高级用法中 `project()` 可能不是 CML 的第二条命令，但就我们的目的而言，它始终是第二条。

下一条所需命令是 `add_executable()`。该命令创建一个 *target*（目标）。用 CMake 的术语来说，target 是开发者赋予一组属性的名称。

target 可能需要跟踪的属性示例包括：

- 产物类型（可执行文件、库、头文件集合等）
- 源文件
- 包含目录
- 可执行文件或库的输出名称
- 依赖项
- 编译器与链接器标志

CMake 的机制通常最好被理解为描述并操作 target 及其属性。属性远不止上面列出的这些。CMake 命令的文档通常会以它们所作用的 target 属性来讨论其功能。

target 本身只是名称，是这组属性的句柄。使用 `add_executable()` 命令就像指定我们想用于 target 的名称一样简单。

```cmake
add_executable(MyProgram)
```

既然有了 target 的名称，我们就可以开始为其关联属性，比如想要构建和链接的源文件。用于此操作的主要命令是 `target_sources()`，它接受一个 target 名称作为参数，后跟一个或多个文件集合。

```cmake
target_sources(MyProgram
  PRIVATE
    main.cxx
)
```

> **注：** CMake 中的路径通常是绝对路径，或相对于 `CMAKE_CURRENT_SOURCE_DIR` 的路径。我们还没有讨论过这类变量，因此你可以将其读作"相对于当前 CML 所在位置"。

每个文件集合都以一个 scope keyword（作用域关键字）为前缀。我们将在讨论将 target 链接在一起时讨论这些关键字的完整语义，但简单地说，它们描述了属性应如何被我们 target 的依赖者继承。

通常，没有任何东西依赖于一个可执行文件。其他程序和库不需要链接到可执行文件，也不需要继承头文件或任何类似的东西。因此这里应使用的合适作用域是 `PRIVATE`，它告知 CMake 此属性仅属于 `MyProgram`，不可被继承。

> **注：** 这条规则几乎适用于所有情况。除高级与罕见用法外，可执行文件的作用域关键字应*始终*为 `PRIVATE`。实现文件通常也是如此，无论 target 是可执行文件还是库。唯一需要"看到" `.cxx` 文件的 target 就是构建它的那个 target。

### 目标

理解如何创建一个包含单个可执行文件的简单 CMake 项目。

### 有用资源

- `project()`
- `cmake_minimum_required()`
- `add_executable()`
- `target_sources()`

### 待编辑文件

- `CMakeLists.txt`

### 入门

`Tutorial.cxx` 的源代码位于 `Help/guide/tutorial/Step1/Tutorial` 目录中，可用于计算一个数的平方根。本练习中无需编辑该文件。

在父目录 `Help/guide/tutorial/Step1` 中有一个 `CMakeLists.txt` 文件，你将完成它。从 `TODO 1` 开始，一直做到 `TODO 4`。

### 构建与运行

完成 `TODO 1` 到 `TODO 4` 后，我们就可以构建并运行项目了！首先运行 `cmake` 可执行文件或 `cmake-gui` 来配置项目，然后用所选的构建工具构建它。

例如，从命令行我们可以进入 `Help/guide/tutorial/Step1` 目录，并按如下方式调用 CMake 进行配置：

```console
cmake -B build
```

`-B` 标志告知 CMake 使用给定的相对路径作为生成文件并在构建过程中存放产物的目录位置。如果省略，则使用当前工作目录。一般认为进行 "in-source" 构建（将这些生成文件放在源代码树本身中）是不良做法。

接下来，用 `cmake --build` 让 CMake 构建项目，并传入与 `-B` 标志相同的相对路径。

```console
cmake --build build
```

`Tutorial` 可执行文件将被构建到 `build` 目录中。对于多配置生成器（如 Visual Studio），它可能被放置在 `build/Debug` 等子目录中。

最后，尝试使用新构建的 `Tutorial`：

```console
Tutorial 4294967296
Tutorial 10
Tutorial
```

> **注：** 根据所用 shell 的不同，正确的语法可能是 `Tutorial`、`./Tutorial`、`.\Tutorial`，甚至 `.\Tutorial.exe`。为简便起见，各练习中统一直接使用 `Tutorial`。

### 解决方案

如前所述，一个包含四条命令的 `CMakeLists.txt` 就足以让我们起步。第一行应为 `cmake_minimum_required()`，按如下方式设置 CMake 版本：

TODO 1: CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.23)
```

构建一个基本项目的下一步是使用 `project()` 命令设置项目名称，并告知 CMake 我们打算用这个 `CMakeLists.txt` 构建软件。

TODO 2: CMakeLists.txt

```cmake
project(Tutorial)
```

现在我们可以用 `add_executable()` 为 Tutorial 设置可执行文件 target。

TODO 3: CMakeLists.txt

```cmake
add_executable(Tutorial)
```

最后，我们可以用 `target_sources()` 将源文件与 Tutorial 可执行文件 target 关联起来。

TODO 4: CMakeLists.txt

```cmake
target_sources(Tutorial
  PRIVATE
    Tutorial/Tutorial.cxx
)
```

## 练习 2 - 构建一个库

构建库只需再引入一条命令：`add_library()`。它的工作方式与 `add_executable()` 完全相同，只是用于库。

```cmake
add_library(MyLibrary)
```

不过，现在是引入头文件的好时机。头文件并不直接作为翻译单元被构建，也就是说它们不是 *build*（构建）需求，而是 *usage*（使用）需求。我们需要了解头文件，以便构建给定 target 的其他部分。

因此，头文件的描述方式与 `tutorial.cxx` 等实现文件略有不同。它们还需要与目前所用 `PRIVATE` 关键字不同的 scope keywords（作用域关键字）。

要描述一组头文件，我们将使用所谓的 `FILE_SET`。

```cmake
target_sources(MyLibrary
  PRIVATE
    library_implementation.cxx

  PUBLIC
    FILE_SET myHeaders
    TYPE HEADERS
    BASE_DIRS
      include
    FILES
      include/library_header.h
)
```

这包含很多复杂性，但我们会逐点说明。首先，注意实现文件作为 `PRIVATE` 源文件出现，与之前可执行文件的处理相同。然而，头文件现在使用 `PUBLIC`。这使库的使用者能够"看到"该库的头文件。

> **注：** 我们还没有完全准备好讨论 scope 关键字的全部语义，将在练习 3 中更完整地讲解。

scope 关键字之后是一个 `FILE_SET`，即被描述为单一单元的一组文件。一个 `FILE_SET` 由以下部分组成：

- `FILE_SET <name>` 是 `FILE_SET` 的名称。这是一个句柄，我们可以在其他上下文中用它来描述该集合。
- `TYPE <type>` 是我们所描述的文件类型。最常见的是头文件，但较新版本的 CMake 还支持 C++20 模块等其他类型。
- `BASE_DIRS` 是文件的"基"位置。可以最简单地理解为将通过 `-I` 标志告知编译器用于头文件查找的位置。
- `FILES` 是文件列表，与之前的实现源文件列表相同。

要描述的信息很多，因此我们可以采用一些有用的快捷方式。 notably，如果 `FILE_SET` 的名称与类型相同，则无需提供 `TYPE` 字段。

```cmake
target_sources(MyLibrary
  PRIVATE
    library_implementation.cxx

  PUBLIC
    FILE_SET HEADERS
    BASE_DIRS
      include
    FILES
      include/library_header.h
)
```

还有其他快捷方式可用，但我们会在后续步骤中进一步讨论。

### 目标

构建一个库。

### 有用资源

- `add_library()`
- `target_sources()`

### 待编辑文件

- `CMakeLists.txt`

### 入门

继续编辑 `Step1` 目录中的文件。从 `TODO 5` 开始，完成至 `TODO 6`。

### 构建与运行

让我们再次构建项目。由于练习 1 中已经创建了 build 目录并运行过 CMake 配置，我们可以跳过配置步骤，直接进行构建：

```console
cmake --build build
```

我们应该能看到库与 Tutorial 可执行文件一同被创建。

### 解决方案

我们首先以与 Tutorial 可执行文件相同的方式添加库 target。

TODO 5: CMakeLists.txt

```cmake
add_library(MathFunctions)
```

接下来需要描述源文件。对于实现文件 `MathFunctions.cxx` 很直接；对于头文件 `MathFunctions.h` 则需要使用 `FILE_SET`。

我们可以给这个 `FILE_SET` 起一个独立的名称，也可以使用将其命名为 `HEADERS` 的快捷方式。本教程中使用快捷方式，但两种方案都有效。

对于 `BASE_DIRS`，我们需要确定能让所需的 `#include <MathFunctions.h>` 指令生效的目录。为此，`MathFunctions` 文件夹本身将作为基目录。如果所需的 include 指令是 `#include <MathFunctions/MathFunctions.h>` 之类，我们会做出不同选择。

TODO 6: CMakeLists.txt

```cmake
target_sources(MathFunctions
  PRIVATE
    MathFunctions/MathFunctions.cxx

  PUBLIC
    FILE_SET HEADERS
    BASE_DIRS
      MathFunctions
    FILES
      MathFunctions/MathFunctions.h
)
```

## 练习 3 - 将库与可执行文件链接在一起

我们已经准备好将库与可执行文件结合，为此必须引入一条新命令 `target_link_libraries()`。这条命令的名称可能有些误导，因为它所做的远不止调用链接器。它一般性地描述 target 之间的关系。

```cmake
target_link_libraries(MyProgram
  PRIVATE
    MyLibrary
)
```

我们终于可以讨论 scope keywords（作用域关键字）了。共有三个：`PRIVATE`、`INTERFACE` 和 `PUBLIC`。它们描述了属性如何对 target 可用。

- `PRIVATE` 属性（也称 "non-interface" 属性）仅对拥有它的 target 可用，例如 `PRIVATE` 头文件只对它们所附加的 target 可见。
- `INTERFACE` 属性仅对*链接*该拥有者 target 的 target 可用。拥有者 target 本身无法访问这些属性。header-only 库就是 `INTERFACE` 属性集合的一个例子，因为 header-only 库本身不构建任何东西，也不需要访问自己的文件。
- `PUBLIC` 并非一种独立的属性类型，而是 `PRIVATE` 与 `INTERFACE` 属性的并集。因此用 `PUBLIC` 描述的需求既对拥有者 target 可用，也对消费者 target 可用。

请看以下具体示例：

```cmake
target_sources(MyLibrary
  PRIVATE
    FILE_SET internalOnlyHeaders
    TYPE HEADERS
    FILES
      InternalOnlyHeader.h

  INTERFACE
    FILE_SET consumerOnlyHeaders
    TYPE HEADERS
    FILES
      ConsumerOnlyHeader.h

  PUBLIC
    FILE_SET publicHeaders
    TYPE HEADERS
    FILES
      PublicHeader.h
)
```

> **注：** 我们在此省略了每个 file set 的 `BASE_DIRS`，这是另一种快捷方式。省略时，`BASE_DIRS` 默认为当前源目录。

`MyLibrary` target 有若干属性会被这次 `target_sources()` 调用修改。到目前为止我们一直泛泛地使用 "properties"（属性）一词，但属性本身是我们可以推理的具名值。这里会被修改的两个具体属性是 `HEADER_SETS` 和 `INTERFACE_HEADER_SETS`，二者都包含通过 `target_sources()` 添加的头文件集列表。

值 `internalOnlyHeaders` 会被添加到 `HEADER_SETS`，`consumerOnlyHeaders` 会被添加到 `INTERFACE_HEADER_SETS`，而 `publicHeaders` 会被同时添加到两者。

当某个 target 被构建时，它会使用自身的 *non-interface* 属性（例如 `HEADER_SETS`），再加上它所链接的任何 target 的 *interface* 属性（例如 `INTERFACE_HEADER_SETS`）。

> **注：** **不必在这种细节层面上推理 CMake 属性。** 上述内容仅为完整性而描述。大多数时候你无需关心某条命令正在修改哪些具体属性。
>
> Scope 关键字有一个简单的直觉，当我们从命令所作用 target 的视角来考虑命令时：**PRIVATE** 给自己用，**INTERFACE** 给别人用，**PUBLIC** 给所有人用。

### 目标

在 Tutorial 可执行文件中使用 `MathFunctions` 库提供的 `sqrt()` 函数。

### 有用资源

- `target_link_libraries()`

### 待编辑文件

- `CMakeLists.txt`
- `Tutorial/Tutorial.cxx`

### 入门

继续编辑 `Step1` 中的文件。从 `TODO 7` 开始，完成至 `TODO 9`。在本练习中，我们需要使用 `target_link_libraries()` 将 `MathFunctions` target 添加到 `Tutorial` target 的链接库中。

修改 CML 后，更新 `Tutorial.cxx` 以使用 `mathfunctions::sqrt()` 函数替代 `std::sqrt`。

### 构建与运行

让我们再次构建项目。与之前一样，我们已经创建过 build 目录并运行过 CMake 配置，因此可以跳过配置步骤，直接进行构建：

```console
cmake --build build
```

验证输出与你对 `MathFunctions` 库的预期是否一致。

### 解决方案

在本练习中，我们通过将 `MathFunctions` 添加到 `Tutorial` 的链接库中，把 `Tutorial` 可执行文件描述为 `MathFunctions` target 的消费者。

为此，我们修改 `CMakeLists.txt` 文件以使用 `target_link_libraries()` 命令，以 `Tutorial` 作为要修改的 target，以 `MathFunctions` 作为我们要添加的库。

TODO 7: CMakeLists.txt

```cmake
target_link_libraries(Tutorial
  PRIVATE
    MathFunctions
)
```

> **注：** 此处的顺序仅 loosely 相关。我们在用 `add_library()` 定义 `MathFunctions` 之前调用 `target_link_libraries()` 对 CMake 而言无关紧要。我们只是在记录 `Tutorial` 依赖于某个名为 `MathFunctions` 的东西，而 `MathFunctions` 的含义在此阶段尚未被解析。
>
> 调用 `target_sources()` 或 `target_link_libraries()` 等 CMake 命令时，唯一需要已定义的 target 就是被修改的那个 target。

最后，剩下的工作就是修改 `Tutorial.cxx` 以使用新提供的 `mathfunctions::sqrt` 函数。也就是说，要添加相应的头文件并修改我们的 `sqrt()` 调用。

TODO 8: Tutorial/Tutorial.cxx

```c++
#include <iostream>
#include <string>

#include <MathFunctions.h>
```

TODO 9: Tutorial/Tutorial.cxx

```c++
// calculate square root
double const outputValue = mathfunctions::sqrt(inputValue);
```

## 练习 4 - 子目录

随着教程的推进，我们将添加更多命令来操作 `Tutorial` 可执行文件和 `MathFunctions` 库。我们希望确保将命令保持在与它们所处理文件相关的本地位置。虽然对于这样的小项目而言并非主要问题，但对于拥有众多 target 和数千个文件的大型项目来说，这会非常有用。

`add_subdirectory()` 命令允许我们纳入位于项目子目录中的 CML。

```cmake
add_subdirectory(SubdirectoryName)
```

当 CMake 处理子目录中的 `CMakeLists.txt` 时，子目录 CML 中描述的所有相对路径都是相对于该子目录，而非顶层 CML。

### 目标

使用 `add_subdirectory()` 来组织项目。

### 有用资源

- `add_subdirectory()`

### 待编辑文件

- `CMakeLists.txt`
- `Tutorial/CMakeLists.txt`
- `MathFunctions/CMakeLists.txt`

### 入门

本步的 `TODOs` 分布在三个 `CMakeLists.txt` 文件中。将 `target_sources()` 命令移到子目录中时，请务必注意必要的路径变化。

> **注：** 之前我们说 `BASE_DIRS` 默认为当前源目录。由于 `MathFunctions` 所需的 include 目录现在与调用 `target_sources()` 的 CML 所在目录相同，我们应当完全移除 `BASE_DIRS` 关键字及其参数。

完成 `TODO 10` 至 `TODO 13`。

### 构建与运行

由于做了重新组织，我们需要在重新构建之前清理原来的 build 目录（否则新的 `Target` 构建文件夹会与之前创建的 `Target` 可执行文件冲突）。我们可以用 `--clean-first` 标志来实现。

无需重新配置。由于 CML 的变化，CMake 会自动重新配置自身。

```console
cmake --build build --clean-first
```

> **注：** 我们的可执行文件和库将被输出到构建树中的新位置。该子目录镜像了源代码树中调用 `add_executable()` 和 `add_library()` 的位置。在后续步骤中，你需要进入构建树中的这个子目录来运行教程可执行文件。
>
> 你可以通过删除旧的 `Tutorial` 可执行文件并观察新的可执行文件产生在 `Tutorial/Tutorial`，来验证此行为。

### 解决方案

我们需要把所有与 `Tutorial` 可执行文件相关的命令移到 `Tutorial/CMakeLists.txt` 中，并用 `add_subdirectory()` 命令替换它们。我们还需要更新 `Tutorial.cxx` 的路径。

TODO 10: Tutorial/CMakeLists.txt

```cmake
add_executable(Tutorial)

target_sources(Tutorial
  PRIVATE
    Tutorial.cxx
)

target_link_libraries(Tutorial
  PRIVATE
    MathFunctions
)
```

TODO 11: CMakeLists.txt

```cmake
add_subdirectory(Tutorial)
```

我们需要对 `MathFunctions` 的命令做同样处理，相应地修改相对路径并移除 `BASE_DIRS`，因为它已不再必要，默认值即可工作。

TODO 12: MathFunctions/CMakeLists.txt

```cmake
add_library(MathFunctions)

target_sources(MathFunctions
  PRIVATE
    MathFunctions.cxx

  PUBLIC
    FILE_SET HEADERS
    FILES
      MathFunctions.h
)
```

TODO 13: CMakeLists.txt

```cmake
add_subdirectory(MathFunctions)
```
