# 第 5 步：深入理解 CMake 库概念

尽管可执行程序大多是“一刀切”的，但库却有许多不同的形式。仅举几例：静态归档库、共享对象库、模块库、对象库、仅头文件库，以及用于描述可被其他目标继承的高级 CMake 属性的库。

在这一步中，你将学习 CMake 能够描述的一些最常见类型的库。这将涵盖项目内大多数 `add_library()` 的用法。从依赖项中导入（或由项目导出以作为依赖项被使用）的库将在后续步骤中介绍。

## 背景

正如我们在 `Step1` 中所学到的，`add_library()` 命令接受要创建的库目标名称作为第一个参数。第二个参数是可选的 `<type>`，其有效取值如下：

- `STATIC`：静态库（Static Library）——一个由目标文件组成的归档，用于链接到其他目标时使用。
- `SHARED`：共享库（Shared Library）——一种动态库，可被其他目标链接并在运行时加载。
- `MODULE`：模块库（Module Library）——一种插件，不能被其他目标链接，但可在运行时通过类似 dlopen 的机制动态加载。
- `OBJECT`：对象库（Object Library）——一组尚未被归档或链接成库的目标文件集合。
- `INTERFACE`：接口库（Interface Library）——一种库目标，它为依赖者指定使用要求，但自身不编译源代码，也不在磁盘上生成库产物。

此外，还有 `IMPORTED` 库，用于描述来自外部项目或模块并导入到当前项目中的库目标。我们将在后续步骤中简要介绍它们。

`MODULE` 库最常见于插件系统，或作为 Python、Javascript 等运行时加载语言的扩展。它们的行为与普通共享库非常相似，只是不能被其他目标直接链接。由于二者足够相似，我们在此不再深入讨论 `MODULE` 库。

## 练习 1 - 静态库与共享库

虽然 `add_library()` 命令支持显式地设置 `STATIC` 或 `SHARED`（有时这也是必要的），但对于大多数可以两种形式皆可运行的“普通”库来说，最好将第二个参数留空。

当未指定类型时，`add_library()` 会根据 `BUILD_SHARED_LIBS` 的值创建 `STATIC` 或 `SHARED` 库。若 `BUILD_SHARED_LIBS` 为真，则创建 `SHARED` 库，否则创建 `STATIC` 库。

```cmake
add_library(MyLib-static STATIC)
add_library(MyLib-shared SHARED)

# Depends on BUILD_SHARED_LIBS
add_library(MyLib)
```

这是一种理想的行为，因为它允许打包者决定将生成哪种类型的库，并确保依赖者链接到该版本的库，而无需修改其源代码。在某些场景下，全静态构建是合适的；而在另一些场景下，共享库则更受欢迎。

> **注：** CMake 默认不会定义 `BUILD_SHARED_LIBS` 变量，这意味着在没有项目或用户干预的情况下，`add_library()` 将生成 `STATIC` 库。

通过将 `add_library()` 的第二个参数留空，项目为打包者和下游依赖者提供了额外的灵活性。

### 目标

将 `MathFunctions` 构建为共享库。

> **注：** 在 Windows 上，你可能会看到关于空 DLL 的警告，因为 `MathFunctions` 没有导出任何符号。

### 参考资源

- `BUILD_SHARED_LIBS`

### 需要编辑的文件

无需编辑任何文件。

### 入门指引

`Help/guide/tutorial/Step5` 目录包含 `Step4` 的完整推荐解决方案。这一步关注的是构建 `MathFunctions` 库，无需任何 `TODO`。你可以直接进入构建步骤。

### 构建与运行

我们可以使用预设进行配置，并通过 `-D` 标志开启 `BUILD_SHARED_LIBS`。

```console
cmake --preset tutorial -DBUILD_SHARED_LIBS=ON
```

然后，我们可以仅使用 `-t` 构建 `MathFunctions` 库。

```console
cmake --build build -t MathFunctions
```

验证 `MathFunctions` 是否生成了共享库，然后重置 `BUILD_SHARED_LIBS`，可以通过重新配置 `-DBUILD_SHARED_LIBS=OFF` 或删除 `CMakeCache.txt` 来实现。

### 解答

本练习无需对项目做任何修改。

## 练习 2 - 接口库

接口库只用于向其他目标传达使用要求，它们自身不会构建或生成任何产物。因此，接口库的所有属性本身都必须是接口属性，需通过 `INTERFACE` 作用域关键字指定。

```cmake
add_library(MyInterface INTERFACE)
target_compile_definitions(MyInterface INTERFACE MYINTERFACE_COMPILE_DEF)
```

在 C++ 开发中，最常见的接口库类型是仅头文件库。这类库不构建任何内容，仅提供发现其头文件所需的标志。

### 目标

向教程项目添加一个仅头文件库，并在 `Tutorial` 可执行程序中使用它。

### 参考资源

- `add_library()`
- `target_sources()`

### 需要编辑的文件

- `MathFunctions/MathLogger/CMakeLists.txt`
- `MathFunctions/CMakeLists.txt`
- `MathFunctions/MathFunctions.cxx`

### 入门指引

在我们之前对 `target_sources(FILE_SET)` 的讨论中曾提到，如果文件集的名称与文件集的类型相同，可以省略 `TYPE` 参数。我们也说过，如果想将当前源目录作为唯一的基础目录，可以省略 `BASE_DIRS` 参数。

现在我们要介绍第三种简写方式：只有当头文件需要被安装（例如库的公共头文件）时，我们才需要包含 `FILES` 参数。

本练习中的 `MathLogger` 头文件仅供 `MathFunctions` 实现内部使用，不会被安装。因此对 `target_sources(FILE_SET)` 的调用应当非常简洁。

> **注：** 编译器的依赖扫描器会发现这些头文件，以确保正确的增量构建。在这些场景下列出头文件仍然有用，因为该列表可用于生成某些 IDE 所依赖的元数据。

你可以开始编辑 `Step5` 目录。完成 `TODO 1` 到 `TODO 7`。

### 构建与运行

预设已更新为使用 `mathfunctions::sqrt` 而非 `std::sqrt`。我们可以像往常一样进行配置和构建。

```console
cmake --preset tutorial
cmake --build build
```

验证 `Tutorial` 的输出现在是否使用了日志框架。

### 解答

首先，我们添加一个名为 `MathLogger` 的新 `INTERFACE` 库。

**TODO 1: MathFunctions/MathLogger/CMakeLists.txt**

```cmake
add_library(MathLogger INTERFACE)
```

然后，我们添加相应的 `target_sources()` 调用来捕获头文件信息。我们将此文件集命名为 `HEADERS`，从而可以省略 `TYPE`；我们不需要 `BASE_DIRS`，因为会使用当前源目录作为默认值；我们也可以省略 `FILES` 列表，因为不打算安装该库。

**TODO 2: MathFunctions/MathLogger/CMakeLists.txt**

```cmake
target_sources(MathLogger
  INTERFACE
    FILE_SET HEADERS
)
```

现在，我们可以将 `MathLogger` 库添加到 `MathFunctions` 的链接库中，并将 `MathLogger` 文件夹添加到项目中。

**TODO 3: MathFunctions/CMakeLists.txt**

```cmake
target_link_libraries(MathFunctions
  PRIVATE
    MathLogger
)
```

**TODO 4: MathFunctions/CMakeLists.txt**

```cmake
add_subdirectory(MathLogger)
```

最后，我们可以更新 `MathFunctions.cxx` 以利用新的日志器。

**TODO 5: MathFunctions/MathFunctions.cxx**

```c++
#include <cmath>
#include <format>

#include <MathLogger.h>
```

**TODO 6: MathFunctions/MathFunctions.cxx**

```c++
mathlogger::Logger Logger;
```

**TODO 7: MathFunctions/MathFunctions.cxx**

```c++
Logger.Log(std::format("Computing sqrt of {} to be {}\n", x, result));
```

## 练习 3 - 对象库

对象库有若干高级用法，但也存在一些棘手的细微之处，在本教程范围内难以一一详述。

```cmake
add_library(MyObjects OBJECT)
```

对象库最明显的缺点是对象本身不能被传递性地链接。如果对象库出现在某个目标的 `INTERFACE_LINK_LIBRARIES` 中，链接该目标的依赖者将无法“看到”这些对象。在这种情况下，对象库的行为类似 `INTERFACE` 库。一般而言，对象库仅适用于通过 `target_link_libraries()` 进行 `PRIVATE` 或 `PUBLIC` 消费。

对象库的一个常见用例是将多个库目标合并为单个归档或共享库对象。即使在同一项目内，库也可能由于种种原因（例如属于组织内不同团队）作为不同目标维护。然而，将它们作为单一面向消费者的二进制文件进行分发可能是更理想的。对象库使这成为可能。

### 目标

向 `MathFunctions` 库添加若干对象库。

### 参考资源

- `target_link_libraries()`
- `add_subdirectory()`

### 需要编辑的文件

- `MathFunctions/CMakeLists.txt`
- `MathFunctions/MathFunctions.h`
- `Tutorial/Tutorial.cxx`

### 入门指引

我们的 `MathFunctions` 库已提供了若干扩展（我们可以设想它们来自组织内的其他团队）。请花一点时间查看 `MathFunctions/MathExtensions` 中提供的目标。然后完成 `TODO 8` 到 `TODO 11`。

### 构建与运行

无需重新配置，我们可以像往常一样构建。

```console
cmake --build build
```

验证 `Tutorial` 的输出是否包含验证信息。同时花一分钟查看 `build/MathFunctions/MathExtensions` 下的构建目录。你应该会发现，与 `MathFunctions` 不同，任何对象库都不会生成归档文件。

### 解答

首先，我们将所有对象库链接到 `MathFunctions`。这些链接是 `PUBLIC` 的，因为我们希望将这些对象作为 `MathFunctions` 自身构建步骤的一部分加入，并希望这些头文件对库的消费者可用。

然后，我们将 `MathExtensions` 子目录添加到项目中。

**TODO 8: MathFunctions/CMakeLists.txt**

```cmake
target_link_libraries(MathFunctions
  PRIVATE
    MathLogger

  PUBLIC
    OpAdd
    OpMul
    OpSub
)
```

**TODO 9: MathFunctions/CMakeLists.txt**

```cmake
add_subdirectory(MathExtensions)
```

为了让消费者能够使用这些扩展，我们在 `MathFunctions.h` 头文件中包含它们的头文件。

**TODO 10: MathFunctions/MathFunctions.h**

```c++
#include <OpAdd.h>
#include <OpMul.h>
#include <OpSub.h>
```

最后，我们可以在 `Tutorial` 程序中使用这些扩展。

**TODO 11: Tutorial/Tutorial.cxx**

```c++
double const checkValue = mathfunctions::OpMul(outputValue, outputValue);
std::cout << std::format("The square of {} is {}\n", outputValue,
                         checkValue);
```
