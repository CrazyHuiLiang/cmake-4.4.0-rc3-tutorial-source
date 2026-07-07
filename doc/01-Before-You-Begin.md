# 第 0 步：开始之前

CMake 教程由一系列动手练习组成，内容是编写并构建一个 C++ 项目；逐步解决库、代码生成器、测试以及外部依赖等日益复杂的构建需求。在踏上这段旅程的第一步之前，我们需要确保手头有正确的工具，并了解如何使用它们。

> **注：** 本教程材料假设用户已具备 C++20 编译器与工具链，并对 C++ 语言至少有入门程度的了解。在此无法一一涵盖获取这些前置条件的所有可能方式。

这一前置步骤提供了关于如何获取并运行 CMake 本身以完成教程其余部分的建议。如果你已经熟悉运行 CMake 的基础知识，可以直接跳到教程的其余部分。

## 获取教程练习

教程源代码示例可从[此压缩包](../../_downloads/0a258b4d00376f5addcda21ec90aeac3/cmake-4.4.0-rc3-tutorial-source.zip)获取。教程的每一步都有一个对应的子文件夹，作为该步练习的起点。

## 获取 CMake

获取 CMake 最直接的方式是从 CMake 官网下载。[网站上的“Download”部分](https://cmake.org/download/)提供了适用于所有常见（以及一些不常见）桌面平台的最新 CMake 构建版本。

不过，更推荐通过你所使用平台上开发者工具的常规分发机制来获取 CMake。CMake 存在于大多数软件包仓库中，可作为 Visual Studio 组件，甚至能从 Python 包索引安装。此外，CMake 通常已包含在大多数面向 C/C++ 的 CI/CD 运行器的基础镜像中。你应查阅自己所用的软件构建环境文档，确认 CMake 是否已经可用。

CMake 也可以按照 CMake 源码树根目录下 `README.rst` 中描述的说明从源码编译。

与任何程序一样，CMake 需要存在于 `PATH` 中才能从 shell 运行。你可以通过运行任意 CMake 命令来验证 CMake 是否可用。

```shell
$ cmake --version
cmake version 3.23.5

CMake suite maintained and supported by Kitware (kitware.com/cmake).
```

> **注：** 如果使用 Visual Studio 提供的开发环境，最好在 Developer Command Prompt 或 Developer Powershell 中运行 CMake。这能确保 CMake 可以访问所有必需的开发者工具与环境变量。

## CMake 生成器

CMake 是一个配置程序，有时被称为“元”构建系统。与其他配置系统一样，CMake 最终并不负责执行产生软件构建的命令，而是根据项目、环境以及用户提供的配置信息生成一个构建系统。

CMake 支持将多种构建系统作为此配置过程的输出。这些输出后端被称为“生成器”，因为它们负责生成构建系统。CMake 支持众多生成器，相关文档可在 `cmake-generators(7)` 中找到。关于你所安装的 CMake 支持哪些生成器的信息，可通过 `cmake --help` 在“Generators”标题下查看。

因此，使用 CMake 还需要有一个能消费此生成器输出的构建程序可用。`Unix Makefiles`、`Ninja` 和 `Visual Studio` 生成器分别需要兼容的 `make`、`ninja` 和 `Visual Studio` 安装。

> **注：** Windows 上的默认生成器通常是运行 CMake 的机器上可用的最新 Visual Studio 版本；在其他平台上默认为 `Unix Makefiles`。

使用哪个生成器可通过 `CMAKE_GENERATOR` 环境变量或 `cmake -G` 选项来控制。

## 单配置与多配置生成器

在许多情况下，可以把底层构建系统当作实现细节，例如在使用 CMake 时不区分 `ninja` 与 `make`。然而，即便是简单的工作流，我们也需要了解某个生成器的一个重要属性：它支持单配置构建，还是支持多配置构建。

软件构建通常有多个我们可能感兴趣的变体。这些变体有诸如 `Debug`、`Release`、`RelWithDebInfo` 和 `MinSizeRel` 这样的名称，其属性与对应变体的名称相符。

单配置构建系统始终以相同方式构建软件，如果它被生成为产生 `Debug` 构建，那么它将始终产生 `Debug` 构建。多配置构建系统则可以根据构建时指定的配置产生不同的输出。

> **注：** 术语 **build configuration**（构建配置）与 **build type**（构建类型）是同义的。在使用只支持单一变体的单配置生成器时，所生成的变体通常被称为“build type”（构建类型）。
>
> 在使用多配置生成器时，可用的变体通常被称为“build configurations”（构建配置）。在构建时选择某个变体通常被称为“selecting a configuration”（选择配置），并在标志与变量中被称为“config”。
>
> 然而这一约定并非通用。技术与通俗文档经常混用这两个术语。在泛指单配置与多配置生成器的语境中，*Configuration* 与 *config* 被认为是更准确的用法。

常用的生成器如下：

| 单配置 | 多配置 |
| --- | --- |
| `Ninja` | `Ninja Multi-Config` |
| `Unix Makefiles` | Visual Studio（所有版本） |
| `FASTBuild` | `Xcode` |

使用单配置生成器时，构建类型依据 `CMAKE_BUILD_TYPE` 环境变量选择，也可在调用 CMake 时通过 `cmake -DCMAKE_BUILD_TYPE=<config>` 直接指定。

> **注：** 就本教程而言，在使用单配置生成器时通常无需指定构建类型。平台特定的默认行为适用于所有练习。

使用多配置生成器时，构建配置在构建时通过构建系统特定的机制或 `cmake --build --config` 选项来指定。

## 其他用法基础

教程其余部分会更深入地讲解剩余的用法基础，但为了确保我们有一个可用的开发环境，这里先列举几个 CMake 选项标志。

- **`cmake -S <dir>`**：指定项目根目录，CMake 将在此查找要构建的项目。该目录包含根 `CMakeLists.txt` 文件，将在教程第 1 步中讨论。

  未指定时，默认为当前工作目录。

- **`cmake -B <dir>`**：指定构建目录，CMake 将在此输出所生成构建系统的文件，并在构建系统运行时输出构建本身的产物。

  未指定时，默认为当前工作目录。

- **`cmake --build <dir>`**：在指定的构建目录中运行构建系统。这是适用于所有生成器的通用命令。对于多配置生成器，可通过以下方式请求所需的配置：

  `cmake --build <dir> --config <cfg>`

## 试一试

`Help/guide/tutorial/Step0` 目录包含一个简单的“Hello World”C++ 项目。CMake 如何配置该项目的细节将在教程第 1 步讨论，我们现在只需关心如何运行 CMake 程序本身。

如上所述，根据我们想要使用的生成器，运行 CMake 有多种可能的方式。如果我们进入 `Help/guide/tutorial/Step0` 目录并运行：

```shell
cmake -B build
```

CMake 将使用平台默认的生成器，为 Step0 项目在 `Help/guide/tutorial/Step0/build` 中生成构建系统。或者，我们也可以指定一个特定的生成器，例如 `Ninja`：

```shell
cmake -G Ninja -B build
```

效果类似，但会改用 `Ninja` 生成器而非平台默认生成器。

> **注：** 我们不能在同一个构建目录中复用不同的生成器。如果想用同一个构建目录切换到不同的生成器，必须在两次 CMake 运行之间删除该构建目录。

生成构建系统后，如何构建并运行该项目取决于我们所使用的生成器类型。如果在非 Windows 平台上使用单配置生成器，我们可以直接这样做：

```shell
cmake --build build
./build/hello
```

> **注：** 在 Windows 上，根据所使用的 shell，我们可能需要指定文件扩展名，即 `./build/hello.exe`。

如果我们使用多配置生成器，就需要指定构建配置。默认配置为 `Debug`、`Release`、`RelWithDebInfo` 和 `MinSizeRel`。构建结果会存放在构建文件夹下特定于配置的子目录中。例如，我们可以运行：

```shell
cmake --build build --config Debug
./build/Debug/hello
```

## 获取帮助与其他资源

如需 CMake 社区的帮助，你可以在 [CMake Discourse 论坛](https://discourse.cmake.org/)上寻求帮助。

如需与 CMake 相关的专业培训，请参见 [CMake 培训落地页](https://www.kitware.com/courses/cmake-training/)。如需其他专业的 CMake 服务，[请通过我们的联系表单与我们联系](https://www.kitware.com/contact/)。

