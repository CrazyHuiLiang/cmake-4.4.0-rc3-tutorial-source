# 第 4 步：深入 CMake 目标命令

CMake 中有若干目标命令可用于描述需求。提醒一下，目标命令是修改其所应用目标属性的命令。这些属性描述了构建软件所需的需求，例如源文件、编译选项和输出名称；或者使用该目标所必需的属性，例如头文件包含路径、库目录以及链接规则。

> **注：** 如 `Step1` 中所述，构建目标所需的属性应使用 `PRIVATE` 作用域关键字描述，使用目标所需的属性应使用 `INTERFACE` 描述，而两者都需要的属性则使用 `PUBLIC` 描述。

在这一步中，我们将逐一介绍 CMake 中所有可用的目标命令。并非所有目标命令都同等重要。我们已经讨论过两个最重要的目标命令：`target_sources()` 和 `target_link_libraries()`。在其余命令中，有些几乎与这两个同样常用，另一些则用于更高级的场景，还有少数几条仅应作为其他选项都不可用时的最后手段。

## 背景

在继续之前，让我们列出所有 CMake 目标命令。我们将它们分为三组：推荐且通常实用的命令、高级且需谨慎使用的命令，以及除非必要否则应避免的"容易自伤（footgun）"命令。

| 常用/推荐 | 高级/谨慎 | 冷门/易自伤 |
| --- | --- | --- |
| `target_compile_definitions()` `target_compile_features()` `target_link_libraries()` `target_sources()` | `get_target_property()` `set_target_properties()` `target_compile_options()` `target_link_options()` `target_precompile_headers()` | `target_include_directories()` `target_link_directories()` |

> **注：** 并不存在所谓"不好"的 CMake 目标命令，它们都有各自的适用场景。此分类只是为了让新手在解决问题时，对应该优先考虑哪些命令有一个简单的直觉。

我们将在下面的练习中演示其中大多数命令。我们不会用到的三个命令是 `get_target_property()`、`set_target_properties()` 和 `target_precompile_headers()`，因此在此先简要讨论它们的用途。

`get_target_property()` 和 `set_target_properties()` 命令通过名称直接访问目标的属性，甚至可以用来给目标附加任意自定义的属性名。

```cmake
add_library(Example)
set_target_properties(Example
  PROPERTIES
    Key Value
    Hello World
)

get_target_property(KeyVar Example Key)
get_target_property(HelloVar Example Hello)

message("Key: ${KeyVar}")
message("Hello: ${HelloVar}")
```

```console
$ cmake -B build
...
Key: Value
Hello: World
```

对 CMake 而言具有语义意义的全部目标属性列表记录在 `cmake-properties(7)` 中，但其中大多数都应通过各自的专用命令来修改。例如，无需直接操作 `LINK_LIBRARIES` 和 `INTERFACE_LINK_LIBRARIES`，它们由 `target_link_libraries()` 处理。

反之，一些较少使用的属性只能通过这些命令访问。`DEPRECATION` 属性（用于给目标附加弃用提示）只能通过 `set_target_properties()` 设置；`ADDITIONAL_CLEAN_FILES`（用于描述 CMake `clean` 目标要额外删除的文件）以及其他此类属性也是如此。

`target_precompile_headers()` 命令接收一个头文件列表（类似于 `target_sources()`），并据此生成预编译头。该预编译头随后会被强制包含到目标中的所有翻译单元里，这对提升构建性能可能很有帮助。

## 练习 1 - 特性与定义

在之前的步骤中，我们曾提醒不要全局设置 `CMAKE_<LANG>_STANDARD`，也不要覆盖打包方对使用哪种语言标准的决定。另一方面，许多库都有构建所需的最低特性集合，此时使用 `target_compile_features()` 命令来传达这些需求是合适的。

```cmake
target_compile_features(MyApp PRIVATE cxx_std_20)
```

`target_compile_features()` 命令将最低语言标准描述为目标属性。如果 `CMAKE_<LANG>_STANDARD` 已高于此版本，或者编译器默认已提供该语言标准，则不做任何处理。如果需要额外标志来启用该标准，CMake 会自动添加。

> **注：** `target_compile_features()` 操作的接口与非接口属性风格与其他目标命令相同。这意味着可以*继承*用 `INTERFACE` 或 `PUBLIC` 作用域关键字指定的语言标准需求。如果某些语言特性仅在实现文件中使用，则相应的编译特性应为 `PRIVATE`。如果该目标的头文件用到了这些特性，则应使用 `PUBLIC` 或 `INTERFACE`。

对于 C++，编译特性形式为 `cxx_std_YY`，其中 `YY` 是标准化年份，例如 `14`、`17`、`20` 等。

`target_compile_definitions()` 命令将编译定义描述为目标属性。它是将构建配置信息传达给源代码本身的最常用机制。与所有属性一样，前面讨论的作用域关键字在此同样适用。

```cmake
target_compile_definitions(MyLibrary
  PRIVATE
    MYLIBRARY_USE_EXPERIMENTAL_IMPLEMENTATION

  PUBLIC
    MYLIBRARY_EXCLUDE_DEPRECATED_FUNCTIONS
)
```

我们既不需要也不应该在通过 `target_compile_definitions()` 描述的编译定义前加上 `-D` 前缀。CMake 会为当前编译器确定正确的标志。

### 目标

使用 `target_compile_features()` 和 `target_compile_definitions()` 来传达语言标准与编译定义需求。

### 参考资源

- `target_compile_features()`
- `target_compile_definitions()`
- `option()`
- `if()`

### 待编辑文件

- `CMakeLists.txt`
- `Tutorial/CMakeLists.txt`
- `MathFunctions/CMakeLists.txt`
- `MathFunctions/MathFunctions.cxx`
- `CMakePresets.json`

### 入门指引

`Help/guide/tutorial/Step4` 目录包含 `Step3` 的完整推荐解决方案以及本步骤相关的 `TODOs`。请完成 `TODO 1` 到 `TODO 8`。

### 构建与运行

我们可以使用 `tutorial` 预设运行 CMake，然后像往常一样构建。

```console
cmake --preset tutorial
cmake --build build
```

验证 `Tutorial` 的输出是否与我们对 `std::sqrt` 的期望一致。

### 解答

首先，我们在顶层 CML 中添加一个新选项。

TODO 1: CMakeLists.txt

```cmake
option(TUTORIAL_BUILD_UTILITIES "Build the Tutorial executable" ON)
option(TUTORIAL_USE_STD_SQRT "Use std::sqrt" OFF)
```

然后，我们为 `MathFunctions` 添加编译特性和定义。

TODO 2-3: MathFunctions/CMakeLists.txt

```cmake
target_compile_features(MathFunctions PRIVATE cxx_std_20)

if(TUTORIAL_USE_STD_SQRT)
  target_compile_definitions(MathFunctions PRIVATE TUTORIAL_USE_STD_SQRT)
endif()
```

以及为 `Tutorial` 添加编译特性。

TODO 4: Tutorial/CMakeLists.txt

```cmake
target_compile_features(Tutorial PRIVATE cxx_std_20)
```

现在我们可以修改 `MathFunctions` 以利用新的定义。

TODO 5: MathFunctions/MathFunctions.cxx

```c++
#include <cmath>
#include <format>
#include <iostream>
```

TODO 6: MathFunctions/MathFunctions.cxx

```c++
double sqrt(double x)
{
#ifdef TUTORIAL_USE_STD_SQRT
  return std::sqrt(x);
#else
  return mysqrt(x);
#endif
}
```

最后，我们可以更新 `CMakePresets.json`。我们不再需要设置 `CMAKE_CXX_STANDARD`，但确实想试用新的编译定义。

TODO 7-8: CMakePresets.json

```json
"cacheVariables": {
  "TUTORIAL_USE_STD_SQRT": "ON"
}
```

## 练习 2 - 编译与链接选项

有时我们需要对传递给编译和链接行的具体选项进行精确控制。这些情况由 `target_compile_options()` 和 `target_link_options()` 来处理。

```cmake
target_compile_options(MyApp PRIVATE -Wall -Werror)
target_link_options(MyApp PRIVATE -T LinksScript.ld)
```

无条件调用 `target_compile_options()` 或 `target_link_options()` 存在若干问题。主要问题在于编译器标志与所使用的编译器前端相关。为了确保我们的项目支持多种编译器前端，我们必须只向编译器传递兼容的标志。

我们可以通过检查 `CMAKE_<LANG>_COMPILER_FRONTEND_VARIANT` 变量来实现这一点，该变量告诉我们编译器前端所支持的标志风格。

> **注：** 在 CMake 3.26 之前，`CMAKE_<LANG>_COMPILER_FRONTEND_VARIANT` 仅对具有多种前端变体的编译器设置。在 CMake 3.26 之后的版本中，单独检查此变量就足够了。然而，本教程面向 CMake 3.23。因此，相关逻辑比我们在此能展开的更复杂。本教程步骤已包含在 CMake 3.23 上检查 MSVC、GCC、Clang 和 AppleClang 编译器变体的正确逻辑。

即使编译器接受了我们传入的标志，编译器标志的语义也会随时间变化，警告相关标志尤其如此。项目不应默认开启"警告即错误"标志，因为这可能因后续版本中原本无害的编译器警告而导致构建失败。

> **注：** 对于错误和警告标志，可考虑将其放入 `CMAKE_<LANG>_FLAGS`，用于本地开发构建和 CI 运行期间（通过预设或 `-D` 标志）。在这些场景下我们确切知道所使用的编译器和工具链，因此可以精确地定制行为，而不会冒着在其他平台上破坏构建的风险。

### 目标

为 `Tutorial` 可执行文件添加适用于 MSVC 风格和 GNU 风格编译器前端的合适警告标志。

### 参考资源

- `target_compile_options()`

### 待编辑文件

- `Tutorial/CMakeLists.txt`

### 入门指引

继续在 `Step4` 目录中编辑文件。用于检查前端变体的条件判断已经写好。请完成 `TODO 9` 和 `TODO 10`，为 `Tutorial` 添加警告标志。

### 构建与运行

由于我们已经为本步骤配置过，可以用通常的命令进行构建。

```cmake
cmake --build build
```

这应该会暴露构建中的一个简单警告，你可以直接修复它。

### 解答

我们需要为 `Tutorial` 添加两个编译选项，一个是 MSVC 风格标志，一个是 GNU 风格标志。

TODO 9-10: Tutorial/CMakeLists.txt

```cmake
if(
  (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC") OR
  (CMAKE_CXX_COMPILER_FRONTEND_VARIANT STREQUAL "MSVC")
)

  target_compile_options(Tutorial PRIVATE /W3)

elseif(
  (CMAKE_CXX_COMPILER_ID STREQUAL "GNU") OR
  (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
)

  target_compile_options(Tutorial PRIVATE -Wall)

endif()
```

## 练习 3 - 包含与链接目录

> **注：** 本练习需要在命令行上直接使用编译器构建一个归档库，且在后续步骤中不会用到。它仅为演示 `target_include_directories()` 和 `target_link_directories()` 的一个用例而设。如果由于任何原因无法完成本练习，可以仅将其作为信息性内容对待，或完全跳过。

通常无需直接描述包含和链接目录，因为在链接由 CMake 生成的目标或导入到 CMake 中的外部依赖（使用后续步骤将介绍的命令）时，这些需求会被继承。

如果我们恰好有一些不由 CMake 目标描述的库或头文件需要纳入构建——也许是厂商提供的预编译二进制文件——我们可以通过 `target_link_directories()` 和 `target_include_directories()` 命令来引入。

```cmake
target_link_directories(MyApp PRIVATE Vendor/lib)
target_include_directories(MyApp PRIVATE Vendor/include)
```

这些命令使用的属性分别映射到 `-L` 和 `-I` 编译器标志（或编译器用于链接和包含目录的任何标志）。

当然，传入一个链接目录并不会告诉编译器将任何内容链接进构建。为此我们需要 `target_link_libraries()`。当 `target_link_libraries()` 收到一个无法映射到目标名称的参数时，它会将该字符串直接作为要链接进构建的库添加到链接行上（并自动添加合适的标志前缀，例如 `-l`）。

### 目标

使用 `target_link_directories()` 和 `target_include_directories()` 在项目中描述一个预编译的、厂商提供的静态库及其头文件。

### 参考资源

- `target_link_directories()`
- `target_include_directories()`
- `target_link_libraries()`

### 待编辑文件

- `Vendor/CMakeLists.txt`
- `Tutorial/CMakeLists.txt`

### 入门指引

你需要将厂商库构建为静态归档以完成本练习。进入 `Help/guide/tutorial/Step4/Vendor/lib` 目录，并按你的平台方式构建代码。

类 Unix 系统上 GCC 工具链的典型命令如下：

```console
g++ -c Vendor.cxx
ar rvs libVendor.a Vendor.o
```

类似地，Windows 上 MSVC 工具链的示例命令如下：

```console
cl -c Vendor.cxx
lib -out:Vendor.lib Vendor.obj
```

此处由于你直接调用 `cl` 和 `lib`，请确保使用与你所用 Visual Studio 版本对应的 Developer Command Prompt，并且目标架构与本 CMake 项目使用的相同。

然后完成 `TODO 11` 到 `TODO 14`。

> **注：** `VendorLib` 是一个 `INTERFACE` 库，意味着它没有构建需求（因为它已经被构建好了）。它的所有属性也都应是接口属性。我们将在下一步更深入地讨论 `INTERFACE` 库。

### 构建与运行

如果你已成功构建 `libVendor`，可以用常规命令重新构建 `Tutorial`。

```console
cmake --build build
```

运行 `Tutorial` 现在应输出一条关于结果对厂商是否可接受的消息。

### 解答

我们需要使用目标链接和包含命令，将该归档库及其头文件描述为 `VendorLib` 的 `INTERFACE` 需求。

TODO 11-13: Vendor/CMakeLists.txt

```cmake
target_include_directories(VendorLib
  INTERFACE
    include
)

target_link_directories(VendorLib
  INTERFACE
    lib
)

target_link_libraries(VendorLib
  INTERFACE
    Vendor
)
```

然后我们可以将 `VendorLib` 添加到 `Tutorial` 的链接库中。

TODO 14: Tutorial/CMakeLists.txt

```cmake
target_link_libraries(Tutorial
  PRIVATE
    MathFunctions
    VendorLib
)
```
