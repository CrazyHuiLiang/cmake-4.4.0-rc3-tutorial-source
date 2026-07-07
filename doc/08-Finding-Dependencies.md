# 第 10 步：查找依赖

在 C/C++ 软件开发中，管理构建依赖始终是现代开发者面临的最高级别挑战之一。CMake 提供了丰富的工具集，用于发现和验证不同类型的依赖。

然而，对于打包正确的项目而言，没有必要使用这些高级工具。如今许多流行的库与工具项目都会生成正确的安装树（就像我们在 `Step 9` 中搭建的那种），它们很容易集成到 CMake 中。

在这种最佳情况下，我们只需要 `find_package()` 即可将依赖导入到我们的项目中。

## 背景

CMake 中用于发现依赖的命令主要有五个，前四个是：

- **`find_file()`**：查找并报告某个具名文件的完整路径，这通常是 `find` 系列命令中最灵活的一个。

- **`find_library()`**：查找并报告适合与 `target_link_libraries()` 一起使用的静态库或共享对象的完整路径。

- **`find_path()`**：查找并报告*包含*某个文件的目录的完整路径。它最常与 `target_include_directories()` 配合用于头文件。

- **`find_program()`**：查找并报告某个程序的可调用名称或路径。常与 `execute_process()` 或 `add_custom_command()` 配合使用。

这些命令应被视为“后备”，在主查找命令不适用时使用。主查找命令是 `find_package()`。它使用全面的内置启发式规则以及上游提供的打包文件，为所请求的依赖提供最佳接口。

## 练习 1 - 使用 `find_package()`

`find_package()` 所使用的搜索路径与行为在其文档中有完整描述，但篇幅过长，不便在此复述。简而言之，它会在众所周知的、不太知名的、冷门的以及用户提供的路径中搜索，试图找到满足所给要求的包。

```cmake
find_package(ForeignLibrary)
```

使用 `find_package()` 的最佳方式是：在构建之前确保所有依赖都已安装到同一棵安装树中，然后通过 `CMAKE_PREFIX_PATH` 变量将该安装树的位置告知 `find_package()`。

> **注：** 构建并安装依赖本身可能是一项巨大的工作。本教程会出于演示目的这样做，但**极其**推荐使用包管理器进行项目本地的依赖管理。

除了要查找的包之外，`find_package()` 还接受若干参数。最值得注意的是：

- 一个位置参数 `<version>`，用于描述要与包的配置版本文件进行比对的版本。应谨慎使用；通过包管理器控制所安装依赖的版本，比因原本无害的版本更新而可能破坏构建要好。

  如果已知某个包依赖某个依赖的较旧版本，则使用版本要求可能是合适的。

- `REQUIRED`：用于在未找到时应中止构建的非可选依赖。

- `QUIET`：用于在未找到时不向用户报告任何内容的可选依赖。

`find_package()` 通过 `<PackageName>_FOUND` 变量报告其结果，对于找到和未找到的包，该变量会分别被设置为真或假值。

### 目标

将一个外部安装的测试框架集成到 Tutorial 项目中。

### 参考资源

- `find_package()`
- `target_link_libraries()`

### 待编辑文件

- `TutorialProject/CMakePresets.json`
- `TutorialProject/Tests/CMakeLists.txt`
- `TutorialProject/Tests/TestMathFunctions.cxx`

### 入门指引

`Step10` 文件夹的组织方式与之前的步骤不同。我们需要编辑的教程项目位于 `Step10/TutorialProject` 之下。现在还出现了另一个项目 `SimpleTest`，以及一个部分填充的安装树，我们将在后续练习中使用它。本练习中你无需编辑这些其他目录中的任何内容，所有 `TODOs` 与解答步骤都针对 `TutorialProject`。

`SimpleTest` 包提供了两个有用的构造：可被链接到测试二进制文件中的 `SimpleTest::SimpleTest` 目标，以及用于自动将测试添加到 CTest 的 `simpletest_discover_tests` 函数。

与其他测试框架类似，`simpletest_discover_tests` 只需传入包含测试的可执行目标的名称即可。

```cmake
simpletest_discover_tests(MyTestExe)
```

`TestMathFunctions.cxx` 文件已更新为以类似于 GoogleTest 或 Catch2 的方式使用 `SimpleTest` 框架。请依次完成 `TODO 1` 到 `TODO 5`，以使用新的测试框架。

> **注：** 这可能不言自明，但 `SimpleTest` 是一个非常简陋的测试框架，仅在表面上与一个功能性测试库相似。虽然本教程中的大部分 CMake 代码可以在其他项目中原样使用，但你不应在本教程之外使用 `SimpleTest`，也不应从它所提供的 CMake 代码中学习。

### 构建与运行

首先我们必须安装 `SimpleTest` 框架。进入 `Help/guide/Step10/SimpleTest` 目录并运行以下命令。

```console
cmake --preset tutorial
cmake --install build
```

> **注：** `SimpleTest` 预设设置好了为教程安装 `SimpleTest` 所需的一切。出于超出本教程范围的原因，无需为 `SimpleTest` 构建或提供任何其他配置。

我们可以观察到 `Step10/install` 目录现已被 `SimpleTest` 的头文件和包文件填充。

现在我们可以像往常一样配置并构建 Tutorial 项目，进入 `Help/guide/Step10/TutorialProject` 并运行：

```console
cmake --preset tutorial
cmake --build build
```

通过使用 CTest 运行测试，验证 `SimpleTest` 框架已被正确消费。

### 解答

首先我们调用 `find_package()` 来发现 `SimpleTest` 包。我们使用 `REQUIRED`，因为没有 `SimpleTest` 测试就无法构建。

**TODO 1: TutorialProject/Tests/CMakeLists.txt**

```cmake
find_package(SimpleTest REQUIRED)
```

接下来我们将 `SimpleTest::SimpleTest` 目标添加到 `TestMathFunctions`。

**TODO 2: TutorialProject/Tests/CMakeLists.txt**

```cmake
target_link_libraries(TestMathFunctions
  PRIVATE
    MathFunctions
    SimpleTest::SimpleTest
)
```

现在我们可以用一个对 `simpletest_discover_tests` 的调用来替换测试描述代码。

**TODO 3: TutorialProject/Tests/CMakeLists.txt**

```cmake
simpletest_discover_tests(TestMathFunctions)
```

我们通过将安装树添加到 `CMAKE_PREFIX_PATH` 来确保 `find_package()` 能够发现 `SimpleTest`。

**TODO 4: TutorialProject/CMakePresets.json**

```json
"cacheVariables": {
  "CMAKE_PREFIX_PATH": "${sourceParentDir}/install",
  "TUTORIAL_USE_STD_SQRT": "OFF",
  "TUTORIAL_ENABLE_IPO": "OFF"
}
```

最后，我们通过移除占位符并包含相应的头文件，更新测试以使用 `SimpleTest` 提供的宏。

**TODO 5: TutorialProject/Tests/TestMathFunctions.cxx**

```c++
#include <MathFunctions.h>
#include <SimpleTest.h>

TEST("add")
{
```

## 练习 2 - 传递依赖

库常常相互构建。一个多媒体应用可能依赖一个为各种容器格式提供支持的库，而该库又可能依赖一个或多个其他库来提供压缩算法。

我们需要在我们放入安装树的包配置文件中表达这些传递性需求。我们借助 `CMakeFindDependencyMacro` 模块来完成，它为已安装的包提供了一种相互递归发现的安全机制。

```cmake
include(CMakeFindDependencyMacro)
find_dependency(zlib)
```

`find_dependency()` 还会从顶层 `find_package()` 调用转发参数。如果 `find_package()` 以 `QUIET` 或 `REQUIRED` 调用，`find_dependency()` 也会使用 `QUIET` 和/或 `REQUIRED`。

### 目标

为 `SimpleTest` 添加一个依赖，并确保依赖 `SimpleTest` 的包也能发现这个传递依赖。

### 参考资源

- `CMakeFindDependencyMacro`
- `find_package()`
- `target_link_libraries()`

### 待编辑文件

- `SimpleTest/CMakeLists.txt`
- `SimpleTest/cmake/SimpleTestConfig.cmake`

### 入门指引

本步骤我们只会编辑 `SimpleTest` 项目。传递依赖 `TransitiveDep` 是一个不提供任何行为的虚拟依赖。然而 CMake 并不知情，如果 CMake 找不到所有必需的依赖，`TutorialProject` 测试将无法配置和构建。

`TransitiveDep` 包已经安装到 `Step10/install` 树中。我们不需要像处理 `SimpleTest` 那样再去安装它。

完成 `TODO 6` 到 `TODO 8`。

### 构建与运行

我们需要重新安装 SimpleTest 框架。进入 `Help/guide/Step10/SimpleTest` 目录并运行与之前相同的命令。

```console
cmake --preset tutorial
cmake --install build
```

现在我们可以重新配置并重新构建 `TutorialProject`，进入 `Help/guide/Step10/TutorialProject` 并执行常规步骤。

```console
cmake --preset tutorial
cmake --build build
```

如果构建通过，我们很可能已成功传播了该传递依赖。通过在 `TutorialProject` 的 `CMakeCache.txt` 中搜索名为 `TransitiveDep_DIR` 的条目来验证这一点。这表明 `TutorialProject` 搜索并找到了 `TransitiveDep`，尽管它对此没有直接需求。

### 解答

首先我们调用 `find_package()` 来发现 `TransitiveDep` 包。我们使用 `REQUIRED` 来验证我们已找到 `TransitiveDep`。

**TODO 6: SimpleTest/CMakeLists.txt**

```cmake
find_package(TransitiveDep REQUIRED)
```

接下来我们将 `TransitiveDep::TransitiveDep` 目标添加到 `SimpleTest`。

**TODO 7: SimpleTest/CMakeLists.txt**

```cmake
target_link_libraries(SimpleTest
  INTERFACE
    TransitiveDep::TransitiveDep
)
```

> **注：** 如果此时构建 `TutorialProject`，我们会预期配置失败，因为 `TransitiveDep::TransitiveDep` 目标在该项目中不可用。

最后，我们在 `SimpleTest` 包配置文件中包含 `CMakeFindDependencyMacro` 并调用 `find_dependency()`，以传播该传递依赖。

**TODO 8: SimpleTest/cmake/SimpleTestConfig.cmake**

```cmake
include(CMakeFindDependencyMacro)
find_dependency(TransitiveDep)
```

## 练习 3 - 查找其他类型的文件

在一个完美的世界里，我们关心的每个依赖都会被正确打包，或至少有其他开发者为我们编写发现它的模块。但我们并非生活在一个完美的世界里，有时我们不得不亲自动手，手动发现构建需求。

为此，我们有本步骤前面列举的其他查找命令，例如 `find_path()`。

```cmake
find_path(PackageIncludeFolder Package.h REQUIRED
  PATH_SUFFIXES
    Package
)
target_include_directories(MyApp
  PRIVATE
    ${PackageIncludeFolder}
)
```

### 目标

将一个未打包的头文件添加到 `TutorialProject` 的 `Tutorial` 可执行文件中。

### 参考资源

- `find_path()`
- `target_include_directories()`

### 待编辑文件

- `TutorialProject/Tutorial/CMakeLists.txt`
- `TutorialProject/Tutorial/Tutorial.cxx`

### 入门指引

本步骤我们只会编辑 `TutorialProject` 项目。未打包的头文件 `Unpackaged/Unpackaged.h` 已经安装到 `Step10/install` 树中。

完成 `TODO 9` 到 `TODO 11`。

### 构建与运行

本练习没有特殊的构建步骤，进入 `Help/guide/Step10/TutorialProject` 并执行常规构建即可。

```console
cmake --build build
```

如果构建通过，我们就成功地将 `Unpackaged` 包含目录添加到了项目中。

### 解答

首先我们调用 `find_path()` 来发现 `Unpackaged` 包含目录。我们使用 `REQUIRED`，因为如果找不到 `Unpackaged.h` 头文件，构建 `Tutorial` 将失败。

**TODO 9: TutorialProject/Tutorial/CMakeLists.txt**

```cmake
find_path(UnpackagedIncludeFolder Unpackaged.h REQUIRED
  PATH_SUFFIXES
    Unpackaged
)
```

接下来我们使用 `target_include_directories()` 将发现的路径添加到 `Tutorial`。

**TODO 10: TutorialProject/Tutorial/CMakeLists.txt**

```cmake
target_include_directories(Tutorial
  PRIVATE
    ${UnpackagedIncludeFolder}
)
```

最后，我们编辑 `Tutorial.cxx` 以包含所发现的头文件。

**TODO 11: TutorialProject/Tutorial/Tutorial.cxx**

```c++
#include <MathFunctions.h>
#include <Unpackaged.h>
```
