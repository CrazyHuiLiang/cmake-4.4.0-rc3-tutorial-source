# 第 3 步：配置与缓存变量

CMake 项目通常会有一些项目特定的配置变量，用户与打包者会对这些变量感兴趣。CMake 提供了多种方式，让调用它的用户或进程传达这些配置选择，但其中最基本的是 [`-D`](../../manual/cmake.1.html#cmdoption-cmake-D) 标志。

在本步骤中，我们将探讨如何从 CML 内部提供项目配置选项的方方面面，以及如何调用 CMake 来利用 CMake 自身和各个项目所提供的配置选项。

## 背景

假设我们有一个支持多种压缩算法的压缩软件 CMake 项目，我们可能希望让项目的打包者决定在构建我们的软件时启用哪些算法。我们可以通过使用由 [`-D`](../../manual/cmake.1.html#cmdoption-cmake-D) 标志设置的变量来实现这一点。

```cmake
if(COMPRESSION_SOFTWARE_USE_ZLIB)
  message("I will use Zlib!")
  # ...
endif()

if(COMPRESSION_SOFTWARE_USE_ZSTD)
  message("I will use Zstd!")
  # ...
endif()
```

```console
$ cmake -B build \
    -DCOMPRESSION_SOFTWARE_USE_ZLIB=ON \
    -DCOMPRESSION_SOFTWARE_USE_ZSTD=OFF
...
I will use Zlib!
```

当然，我们希望为这些配置选择提供合理的默认值，并提供一种方式来说明某个选项的用途。这一功能由 [`option()`](../../command/option.html#command:option) 命令提供。

```cmake
option(COMPRESSION_SOFTWARE_USE_ZLIB "Support Zlib compression" ON)
option(COMPRESSION_SOFTWARE_USE_ZSTD "Support Zstd compression" ON)

if(COMPRESSION_SOFTWARE_USE_ZLIB)
  # Same as before
# ...
```

```console
$ cmake -B build \
    -DCOMPRESSION_SOFTWARE_USE_ZLIB=OFF
...
I will use Zstd!
```

由 [`-D`](../../manual/cmake.1.html#cmdoption-cmake-D) 标志和 [`option()`](../../command/option.html#command:option) 创建的名称不是普通变量，而是**缓存**变量（cache variables）。缓存变量是全局可见的变量，它们具有*粘性*（sticky）——其值在初次设置后难以更改。事实上它们非常粘，以至于在项目模式下，CMake 会在多次配置之间保存并恢复缓存变量。一旦设置了某个缓存变量，它就会一直保留，直到另一个 [`-D`](../../manual/cmake.1.html#cmdoption-cmake-D) 标志覆盖已保存的变量。

> **注：** CMake 自身有数十个用于配置的普通变量和缓存变量。这些变量记录在 [`cmake-variables(7)`](../../manual/cmake-variables.7.html#manual:cmake-variables\(7\)) 中，其工作方式与项目提供的配置变量相同。

[`set()`](../../command/set.html#command:set) 也可用于操作缓存变量，但不会更改已经创建的变量。

```cmake
set(StickyCacheVariable "I will not change" CACHE STRING "")
set(StickyCacheVariable "Overwrite StickyCache" CACHE STRING "")

message("StickyCacheVariable: ${StickyCacheVariable}")
```

```console
$ cmake -B build
...
StickyCacheVariable: I will not change
```

由于 [`-D`](../../manual/cmake.1.html#cmdoption-cmake-D) 标志在任何项目命令之前被处理，因此在设置缓存变量的值时，它们具有优先权。

```console
$ cmake -B build \
  -DStickyCacheVariable="Commandline always wins"
...
StickyCacheVariable: Commandline always wins
```

虽然缓存变量通常无法更改，但它们可以被普通变量*遮蔽*（shadowed）。我们可以通过 [`set()`](../../command/set.html#command:set) 将一个变量设置为与缓存变量同名，然后用 [`unset()`](../../command/unset.html#command:unset) 移除该普通变量来观察这一现象。

```cmake
set(ShadowVariable "In the shadows" CACHE STRING "")
set(ShadowVariable "Hiding the cache variable")
message("ShadowVariable: ${ShadowVariable}")

unset(ShadowVariable)
message("ShadowVariable: ${ShadowVariable}")
```

```console
$ cmake -B build
...
ShadowVariable: Hiding the cache variable
ShadowVariable: In the shadows
```

> **注：** [Script mode](../../manual/cmake.1.html#script-processing-mode)（脚本模式）的工作方式略有不同，只有命令中 [`-P`](../../manual/cmake.1.html#cmdoption-cmake-P) 标志之前提供的 [`-D`](../../manual/cmake.1.html#cmdoption-cmake-D) 标志才会被求值并在运行的脚本中可用。

## 练习 1 - 使用选项

我们可以想象这样一种场景：使用者真正想要的是我们的 `MathFunctions` 库，而 `Tutorial` 实用程序只是一个“可有可无”的附加物。在这种情况下，我们可能希望添加一个选项，让使用者能够禁用 `Tutorial` 二进制程序的构建，仅构建 `MathFunctions` 库。

凭借我们对选项、条件判断和缓存变量的了解，我们已经具备了实现这一配置所需的全部要素。

### 目标

添加一个名为 `TUTORIAL_BUILD_UTILITIES` 的选项，用于控制是否配置并构建 `Tutorial` 二进制程序。

> **注：** CMake 允许我们在配置完成后确定要构建哪些目标。我们的用户可以只请求 `MathFunctions` 库而不要 `Tutorial`。CMake 还提供了将目标从 `ALL`（构建所有其他可用目标的默认目标）中排除的机制。
>
> 然而，将目标从配置中完全排除的选项既方便又受欢迎，尤其是当配置这些目标涉及可能耗时较长的重量级步骤时。
>
> 如果打包者不感兴趣的目标被完全排除，这也会简化 [`install()`](../../command/install.html#command:install) 的逻辑——我们将在后续步骤中讨论这一点。

### 有用资源

- [`option()`](../../command/option.html#command:option)
- [`if()`](../../command/if.html#command:if)

### 待编辑文件

- `CMakeLists.txt`

### 入门

`Help/guide/tutorial/Step3` 文件夹包含 `Step1` 的完整推荐解决方案，以及本步骤相关的 `TODOs`。花点时间回顾并重新熟悉 `Tutorial` 项目。

当你觉得自己理解了当前代码后，从 `TODO 1` 开始，一直完成到 `TODO 2`。

### 构建与运行

现在我们可以重新配置项目了。不过这一次我们希望通过 [`-D`](../../manual/cmake.1.html#cmdoption-cmake-D) 标志来控制配置。我们同样先导航到 `Help/guide/tutorial/Step3` 并调用 CMake，但这次带上我们的配置选项。

```console
cmake -B build -DTUTORIAL_BUILD_UTILITIES=OFF
```

现在我们可以像往常一样构建了。

```console
cmake --build build
```

构建完成后，我们应观察到没有生成 Tutorial 可执行文件。由于缓存变量具有粘性，即使是重新配置也不应改变这一点——尽管该选项默认为 `ON`。

```console
cmake -B build
cmake --build build
```

上述命令不会生成 Tutorial 可执行文件，缓存变量已被“锁定”。要改变这一点，我们有两个选择。首先，我们可以编辑在多次 CMake 配置运行之间存储缓存变量的文件，即“CMake 缓存”。该文件是 `build/CMakeCache.txt`，在其中我们可以找到该选项缓存变量。

```text
//Build the Tutorial executable
TUTORIAL_BUILD_UTILITIES:BOOL=OFF
```

我们可以将其从 `OFF` 改为 `ON`，重新运行构建，就能得到我们的 `Tutorial` 可执行文件。

> **注：** `CMakeCache.txt` 中的条目形式为 `<Name>:<Type>=<Value>`，但“类型”只是一个提示。CMake 中的所有对象都是字符串，无论缓存中如何声明。

或者，我们可以在命令行上更改缓存变量的值，因为命令行在 `CMakeCache.txt` 加载之前运行，所以它的值优先于缓存文件中的值。

```console
cmake -B build -DTUTORIAL_BUILD_UTILITIES=ON
cmake --build build
```

这样做之后，我们会观察到 `CMakeCache.txt` 中的值已从 `OFF` 翻转为 `ON`，并且 `Tutorial` 可执行文件已被构建。

### 解决方案

首先，我们创建 [`option()`](../../command/option.html#command:option) 来为缓存变量提供一个合理的默认值。

TODO 1: CMakeLists.txt

```cmake
option(TUTORIAL_BUILD_UTILITIES "Build the Tutorial executable" ON)
```

然后我们可以检查该缓存变量，以条件性地启用 `Tutorial` 可执行文件（通过添加其子目录的方式）。

TODO 2: CMakeLists.txt

```cmake
if(TUTORIAL_BUILD_UTILITIES)
  add_subdirectory(Tutorial)
endif()
```

## 练习 2 - `CMAKE` 变量

CMake 提供了若干重要的普通变量和缓存变量，让打包者能够控制构建。诸如编译器、默认标志、软件包搜索路径等决策，都由 CMake 自身的配置变量控制。

其中最重要的是语言标准。因为语言标准会对某个软件包所呈现的 ABI 产生重大影响。例如，库在较新的标准下使用标准 C++ 模板，而在较早的标准下提供 polyfill（兼容填充）是十分常见的做法。如果某个库在不同的标准下被使用，那么标准模板与 polyfill 之间的 ABI 不兼容性可能导致难以理解的错误和运行时崩溃。

通过 [`CMAKE_<LANG>_STANDARD`](../../variable/CMAKE_LANG_STANDARD.html#variable:CMAKE_%3CLANG%3E_STANDARD) 缓存变量可以确保我们的所有目标都在同一语言标准下构建。对于 C++，该变量为 [`CMAKE_CXX_STANDARD`](../../variable/CMAKE_CXX_STANDARD.html#variable:CMAKE_CXX_STANDARD)。

> **注：** 由于这些变量如此重要，开发者同样不应在其 CML 中覆盖或遮蔽它们，这一点也同样重要。如果库因为想要 C++20 而在 CML 中遮蔽 `CMAKE_<LANG>_STANDARD`，而打包者已决定用 C++23 构建其余的库和应用程序，就可能导致上述那种可怕的、难以理解的错误。
>
> 在没有非常充分理由的情况下，不要 [`set()`](../../command/set.html#command:set) `CMAKE_` 全局变量。我们将在后续步骤中讨论目标传达定义和最低标准等需求的更好方法。

在本练习中，我们将向库和可执行文件中引入一些 C++20 代码，并通过设置相应的缓存变量以 C++20 标准构建它们。

### 目标

使用 `std::format` 来格式化输出字符串，而不是使用流运算符。为确保 `std::format` 可用，需将 CMake 配置为对 C++ 目标使用 C++20 标准。

### 有用资源

- [`cmake -D`](../../manual/cmake.1.html#cmdoption-cmake-D)
- [`CMAKE_<LANG>_STANDARD`](../../variable/CMAKE_LANG_STANDARD.html#variable:CMAKE_%3CLANG%3E_STANDARD)
- [`CMAKE_CXX_STANDARD`](../../variable/CMAKE_CXX_STANDARD.html#variable:CMAKE_CXX_STANDARD)
- [`CXX_STANDARD`](../../prop_tgt/CXX_STANDARD.html#prop_tgt:CXX_STANDARD)
- [cppreference `<format>`](https://en.cppreference.com/w/cpp/utility/format/format)

### 待编辑文件

- `Tutorial/Tutorial.cxx`
- `MathFunctions/MathFunctions.cxx`

### 入门

继续编辑 `Step3` 中的文件。完成 `TODO 3` 到 `TODO 7`。我们将修改打印输出，用 `std::format` 代替流运算符。

使用上一个练习中讨论过的任意方法，确保缓存变量已设置为会构建 Tutorial 可执行文件。

### 构建与运行

我们需要用新的标准重新配置项目，可以使用与 `TUTORIAL_BUILD_UTILITIES` 缓存变量相同的方法。

```console
cmake -B build -DCMAKE_CXX_STANDARD=20
```

> **注：** 按照惯例，配置变量以其提供者为前缀。CMake 的配置变量以 `CMAKE_` 为前缀，而项目应将其变量以 `<PROJECT>_` 为前缀。
>
> 本教程的配置变量遵循这一惯例，以 `TUTORIAL_` 为前缀。

既然我们已用 C++20 完成配置，就可以像往常一样构建了。

```console
cmake --build build
```

### 解决方案

我们需要包含 `<format>`，然后使用它。

TODO 3: Tutorial/Tutorial.cxx

```c++
#include <format>
#include <iostream>
#include <string>
```

TODO 4: Tutorial/Tutorial.cxx

```c++
if (argc < 2) {
  std::cout << std::format("Usage: {} number\n", argv[0]);
  return 1;
}
```

TODO 5: Tutorial/Tutorial.cxx

```c++
// calculate square root
double const outputValue = mathfunctions::sqrt(inputValue);
std::cout << std::format("The square root of {} is {}\n", inputValue,
                         outputValue);
```

`MathFunctions` 库也同样处理。

TODO 6: MathFunctions.cxx

```c++
#include <format>
#include <iostream>
```

TODO 7: MathFunctions.cxx

```c++
double delta = x - (result * result);
result = result + 0.5 * delta / result;

std::cout << std::format("Computing sqrt of {} to be {}\n", x, result);
```

## 练习 3 - CMakePresets.json

管理这些配置值可能很快变得难以应付。在 CI 系统中，将它们作为某个 CI 步骤的一部分记录下来是合适的。例如，在 Github Actions 的 CI 步骤中，我们可能会看到类似如下的内容：

```yaml
- name: Configure and Build
  run: |
    cmake \
      -B build \
      -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_CXX_STANDARD=20 \
      -DCMAKE_CXX_EXTENSIONS=ON \
      -DTUTORIAL_BUILD_UTILITIES=OFF \
      # Possibly many more options
      # ...

    cmake --build build
```

在本地开发代码时，即使只输入一次所有这些选项也容易出错。如果出于某种原因需要全新配置，多次这样做会令人筋疲力尽。

针对这一问题有各种各样不同的解决方案，最终如何选择取决于你作为开发者的偏好。偏向命令行的开发者通常使用任务运行器（task runner）来以期望的选项调用 CMake 构建项目。大多数 IDE 也有自定义机制来控制 CMake 配置。

在这里完整列举每一种可能的配置工作流是不现实的。相反，我们将探索 CMake 的内置解决方案，即 [`CMake Presets`](../../manual/cmake-presets.7.html#manual:cmake-presets\(7\))（CMake 预设）。预设为我们提供了一种格式，用以命名并表达一组 CMake 配置选项的集合。

> **注：** 预设能够表达完整的 CMake 工作流，从配置、构建，一直到安装软件包。
>
> 它们远比我们在此篇幅中所能介绍的更为灵活。我们在此仅将其用于配置。

CMake 预设有两个标准文件：`CMakePresets.json`，它作为项目的一部分并被纳入版本控制；以及 `CMakeUserPresets.json`，它用于本地用户配置，不应被纳入版本控制。

对开发者有用的最简预设，不过是配置一些变量而已。

```json
{
  "version": 4,
  "configurePresets": [
    {
      "name": "example-preset",
      "cacheVariables": {
        "EXAMPLE_FOO": "Bar",
        "EXAMPLE_QUX": "Baz"
      }
    }
  ]
}
```

在调用 CMake 时，以前我们会这样做：

```console
cmake -B build -DEXAMPLE_FOO=Bar -DEXAMPLE_QUX=Baz
```

现在我们可以使用预设：

```console
cmake -B build --preset example-preset
```

CMake 会搜索名为 `CMakePresets.json` 和 `CMakeUserPresets.json` 的文件，并在可用时从中加载具名的配置。

> **注：** 命令行标志可以与预设混合使用。命令行标志优先于预设中找到的值。

> **注：** 在 CMake 4.4 及更高版本中，CMake 还可以从通过 [`cmake --presets-file`](../../manual/cmake.1.html#cmdoption-cmake-presets-file) 指定的任意文件中加载预设。这在跨多个项目复用设置时很有用，因为可以避免在每个项目各自的 `CMakePresets.json` 文件中重复这些设置。

预设还支持有限的宏，即可在预设内部进行花括号展开的变量。我们唯一关心的是 `${sourceDir}` 宏，它展开为项目的根目录。我们可以用它来设置构建目录，从而在配置项目时省略 [`-B`](../../manual/cmake.1.html#cmdoption-cmake-B) 标志。

```json
{
  "name": "example-preset",
  "binaryDir": "${sourceDir}/build"
}
```

### 目标

使用 CMake 预设而非命令行标志来配置并构建教程。

### 有用资源

- [`cmake-presets(7)`](../../manual/cmake-presets.7.html#manual:cmake-presets\(7\))

### 待编辑文件

- `CMakePresets.json`

### 入门

继续编辑 `Step3` 中的文件。完成 `TODO 8` 和 `TODO 9`。

> **注：** `CMakePresets.json` 中的 `TODOs` 需要*替换*。完成练习后，文件中不应残留任何 `TODO` 键。

在配置之前删除现有的 build 文件夹即可验证预设是否正常工作，这样可以确保你不会复用现有的 CMake 缓存进行配置。

> **注：** 在 CMake 3.24 及更高版本中，通过 [`cmake --fresh`](../../manual/cmake.1.html#cmdoption-cmake-fresh) 进行配置也能达到相同效果。

此后所有的配置更改都将通过 `CMakePresets.json` 文件进行。

### 构建与运行

现在我们可以使用预设文件来管理配置了。

```console
cmake --preset tutorial
```

预设能够为我们运行构建步骤，但在本教程中，我们将继续自行运行构建。

```console
cmake --build build
```

### 解决方案

我们需要做两处更改：首先，将构建目录（也称为“二进制目录”）设置为项目文件夹的 `build` 子目录；其次，将 `CMAKE_CXX_STANDARD` 设置为 `20`。

TODO 8-9: CMakePresets.json

```json
{
  "version": 4,
  "configurePresets": [
    {
      "name": "tutorial",
      "displayName": "Tutorial Preset",
      "description": "Preset to use with the tutorial",
      "binaryDir": "${sourceDir}/build",
      "cacheVariables": {
        "CMAKE_CXX_STANDARD": "20"
      }
    }
  ]
}
```
