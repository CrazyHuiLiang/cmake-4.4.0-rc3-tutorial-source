# 第 6 步：深入理解系统自省

为了发现关于系统环境和工具链的信息，CMake 通常会编译一些小型测试程序，以验证编译器标志、头文件、内建函数或其他语言特性的可用性。

在这一步中，我们将利用 CMake 自身所使用的相同测试程序机制，在我们自己的项目代码中进行系统自省。

## 背景

一种可追溯到配置与构建系统最早期年代的老技巧是：通过编译一个使用某特性小程序来验证该特性的可用性。

在许多场景下，CMake 使这种做法不再必要。正如我们将在后续步骤中所述，如果 CMake 能够找到某个库依赖项，我们就可以相信它具备我们所期望的全部设施（头文件、代码生成器、测试工具等）。反之，如果 CMake 无法找到某个依赖项，强行使用该依赖几乎注定会失败。

然而，还有一些关于工具链的信息 CMake 并不能轻易获取。对于这些高级场景，我们可以编写自己的测试程序和编译命令来检查可用性。

CMake 提供了模块来简化这些检查，相关文档见 `cmake-modules(7)`。任何以 `Check` 开头的模块都是一个系统自省模块，可用于探查工具链和系统环境。其中一些值得关注的模块包括：

- `CheckIncludeFiles`：检查一个或多个 C/C++ 头文件。
- `CheckCompilerFlag`：检查编译器是否支持给定标志。
- `CheckSourceCompiles`：检查源代码能否针对给定语言进行构建。
- `CheckIPOSupported`：检查编译器是否支持过程间优化（IPO/LTO）。

## 练习 1 - 检查头文件

一种快速且简单的检查是判断某个头文件在特定平台上是否可用，CMake 为此提供了 `CheckIncludeFiles`。这最适用于系统头文件和内建头文件，它们可能不由某个具体包提供，但预期在许多构建环境中都使用。

```cmake
include(CheckIncludeFiles)
check_include_files(sys/socket.h HAVE_SYS_SOCKET_H LANGUAGE CXX)
```

> **注：** 这些函数在 CMake 中并非立即可用，必须通过 `include()` 引入其关联模块（即 CMakeLang 文件）后才能使用。许多模块位于 CMake 自身的 `Modules` 文件夹中。这个内置的 `Modules` 文件夹是 CMake 在执行 `include()` 命令时搜索路径之一。你可以把这些模块想象成标准库头文件，它们预期总是可用的。

一旦确认某个头文件存在，我们就可以使用前面已介绍过的条件判断和目标命令机制将这一信息传达给代码。

### 目标

检查 x86 SSE2 内建头文件是否可用，若可用则用它来改进 `mathfunctions::sqrt`。

### 参考资源

- `CheckIncludeFiles`
- `target_compile_definitions()`

### 需要编辑的文件

- `MathFunctions/CMakeLists.txt`
- `MathFunctions/MathFunctions.cxx`

### 入门指引

`Help/guide/tutorial/Step6` 目录包含 `Step5` 的完整推荐解决方案，以及与本步骤相关的 `TODO`。它还包含针对各种条件专门实现的 `sqrt` 函数，你可以在 `MathFunctions/MathFunctions.cxx` 中找到它们。

完成 `TODO 1` 到 `TODO 3`。注意，库中已经添加了一些 `#ifdef` 指令，随着我们在本步骤中的推进，它们会改变库的行为。

### 构建与运行

我们可以使用通常的命令进行配置。

```console
cmake --preset tutorial
cmake --build build
```

在配置步骤的输出中，我们应当能看到 CMake 在检查 `emmintrin.h` 头文件。

```console
-- Looking for include file emmintrin.h
-- Looking for include file emmintrin.h - found
```

如果你的系统上该头文件可用，请验证 `Tutorial` 的输出包含关于使用 SSE2 的信息。反之，如果该头文件不可用，你应当看到 `Tutorial` 的常规行为。

### 解答

首先，我们引入并使用 `CheckIncludeFiles` 模块，验证 `emmintrin.h` 头文件是否可用。

**TODO 1: MathFunctions/CMakeLists.txt**

```cmake
include(CheckIncludeFiles)
check_include_files(emmintrin.h HAS_EMMINTRIN LANGUAGE CXX)
```

然后，我们使用检查结果在 `MathFunctions` 上条件性地设置一个编译定义。

**TODO 2: MathFunctions/CMakeLists.txt**

```cmake
if(HAS_EMMINTRIN)
  target_compile_definitions(MathFunctions PRIVATE TUTORIAL_USE_SSE2)
endif()
```

最后，我们可以在 `MathFunctions` 库中条件性地包含该头文件。

**TODO 3: MathFunctions/MathFunctions.cxx**

```c++
#ifdef TUTORIAL_USE_SSE2
#  include <emmintrin.h>
#endif
```

## 练习 2 - 检查源代码可编译性

有时仅检查头文件是不够的。当没有头文件可供检查时尤其如此，例如编译器内建函数的情形。对于这些场景，我们有 `CheckSourceCompiles`。

```cmake
include(CheckSourceCompiles)
check_source_compiles(CXX
  "
    int main() {
      int a, b, c;
      __builtin_add_overflow(a, b, &c);
    }
  "
  HAS_CHECKED_ADDITION
)
```

> **注：** 默认情况下，`CheckSourceCompiles` 会构建并链接一个可执行程序。被检查的代码必须提供有效的 `int main()` 才能成功。

执行检查之后，这种系统自省的应用方式与我们在头文件部分所讨论的完全相同。

### 目标

检查 GNU SSE2 内建函数是否可用，若可用则用它来改进 `mathfunctions::sqrt`。

### 参考资源

- `CheckSourceCompiles`
- `target_compile_definitions()`

### 需要编辑的文件

- `MathFunctions/CMakeLists.txt`

### 入门指引

完成 `TODO 4` 和 `TODO 5`。无需修改 `MathFunctions` 的实现代码，因为这些已经提供。

### 构建与运行

我们只需重新构建教程。

```console
cmake --build build
```

> **注：** 如果某项检查失败而你认为它应当成功，你需要通过删除 `CMakeCache.txt` 文件来清除 CMake 缓存。CMake 在后续运行中若已有缓存结果，则不会重新执行编译检查。

在配置步骤的输出中，我们应当能看到 CMake 在检查所提供的源代码是否可编译，结果会以我们传给 `check_source_compiles()` 的变量名报告。

```console
-- Performing Test HAS_GNU_BUILTIN
-- Performing Test HAS_GNU_BUILTIN - Success
```

如果你的编译器上这些内建函数可用，请验证 `Tutorial` 的输出包含关于使用 GNU 内建函数的信息。反之，如果这些内建函数不可用，你应当看到 `Tutorial` 之前的行为。

### 解答

首先，我们引入并使用 `CheckSourceCompiles` 模块，验证所提供的源代码能否构建。

**TODO 4: MathFunctions/CMakeLists.txt**

```text
include(CheckSourceCompiles)
check_source_compiles(CXX
  [=[
    typedef double v2df __attribute__((vector_size(16)));
    int main() {
      __builtin_ia32_sqrtsd(v2df{});
    }
  ]=]
  HAS_GNU_BUILTIN
)
```

然后，我们使用检查结果在 `MathFunctions` 上条件性地设置一个编译定义。

**TODO 5: MathFunctions/CMakeLists.txt**

```cmake
if(HAS_GNU_BUILTIN)
  target_compile_definitions(MathFunctions PRIVATE TUTORIAL_USE_GNU_BUILTIN)
endif()
```

## 练习 3 - 检查过程间优化

过程间优化和链接时优化可以为某些软件带来显著的性能提升。CMake 能够通过 `CheckIPOSupported` 检查 IPO 标志的可用性。

```cmake
include(CheckIPOSupported)
check_ipo_supported() # fatal error if IPO is not supported
set_target_properties(MyApp
  PROPERTIES
    INTERPROCEDURAL_OPTIMIZATION TRUE
)
```

> **注：** 关于项目内 IPO 配置，有几个重要的注意事项：
>
> - CMake 并不了解每种编译器上的所有 IPO/LTO 标志，对于已知的工具链，通过单独调优通常能获得更好的结果。
> - 在目标上设置 `INTERPROCEDURAL_OPTIMIZATION` 属性并不会改变它所链接的任何目标，或其他项目的依赖项。IPO 只能“看到”同样以适当方式编译的其他目标。
>
> 基于这些原因，应当认真考虑通过外部机制（预设、`-D` 标志、工具链文件等）在依赖树中的所有项目间手动设置 IPO/LTO 标志，而不是在项目内部进行控制。

然而，尤其对于极大型项目而言，拥有一种在工具链支持时启用 IPO 的项目内机制会很有用。

### 目标

当工具链支持时，为整个教程项目启用 IPO。

### 参考资源

- `CheckIPOSupported`
- `CMAKE_INTERPROCEDURAL_OPTIMIZATION`

### 需要编辑的文件

- `CMakeLists.txt`

### 入门指引

继续编辑 `Step6` 中的文件。完成 `TODO 6` 和 `TODO 7`。

### 构建与运行

我们只需重新构建教程。

```console
cmake --build build
```

如果 IPO 不可用，我们将在配置期间看到一条错误信息。否则不会有任何变化。

> **注：** 无论 IPO 检查的结果如何，我们都不应期待 `Tutorial` 或 `MathFunctions` 的行为发生任何变化。

### 解答

第一个 `TODO` 很简单，我们向项目添加另一个选项。

**TODO 6: CMakeLists.txt**

```cmake
option(TUTORIAL_ENABLE_IPO "Check for and use IPO support" ON)
```

下一步稍复杂，不过 `CheckIPOSupported` 的文档中有一个几乎完整的示例，正是我们需要做的。唯一区别在于我们要在项目范围内启用 IPO，而不是针对单个目标。

**TODO 7: CMakeLists.txt**

```cmake
if(TUTORIAL_ENABLE_IPO)
  include(CheckIPOSupported)
  check_ipo_supported(RESULT result OUTPUT output)
  if(result)
    message("IPO is supported, enabling IPO")
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION ON)
  else()
    message(WARNING "IPO is not supported: ${output}")
  endif()
endif()
```

> **注：** 通常我们不建议在项目内部设置 `CMAKE_` 变量。这里，我们通过 `option()` 来控制该行为，使打包者可以选择退出我们的覆盖。这是一种不完美但可接受的方案，适用于我们希望提供选项来控制由 `CMAKE_` 变量控制的项目级行为的场景。
