# 第 9 步：安装命令与概念

项目要做的不仅仅是构建和测试代码，还需要将代码提供给使用者。构建树中的文件布局并不适合供其他项目使用：二进制文件位于出人意料的位置，头文件远在源树深处，而且没有明确的方式去发现提供了哪些目标以及如何使用它们。

将产物从源树和构建树迁移到适合使用的最终布局中，这一过程被称为安装（installation）。CMake 将完整的安装工作流作为项目描述的一部分加以支持，既控制安装树中产物的布局，又为希望使用该安装树所提供库的其他 CMake 项目重建目标。

## 背景

所有 CMake 安装都经由单一命令 `install()` 完成，该命令被拆分为许多子命令，各自负责安装过程的某个方面。对于基于目标的 CMake 工作流，通常只需依赖 `install(TARGETS)` 来安装目标本身，而无需借助 `install(FILES)` 或 `install(DIRECTORY)` 手动移动文件。

> **注：** 这正是我们需要为打算安装的头文件集（header sets）添加 `FILES` 的原因。CMake 需要在安装关联目标时能够定位这些文件。

CMake 将基于目标的安装划分为多种产物类型。在 CMake 3.23 中可用的产物类型包括：

- `ARCHIVE`：静态库（`.a` / `.lib`）、DLL 导入库（`.lib`）以及少量其他"归档类"对象。
- `LIBRARY`：共享库（`.so`）、模块及其他动态可加载对象。**不**包括 Windows 的 DLL 文件（`.dll`）或 MacOS framework。
- `RUNTIME`：除 MacOS bundle 外的各类可执行文件，以及 Windows 的 DLL（`.dll`）。
- `OBJECT`：来自 `OBJECT` 库的对象。
- `FRAMEWORK`：静态和共享的 MacOS framework。
- `BUNDLE`：MacOS bundle 可执行文件。
- `PUBLIC_HEADER` / `PRIVATE_HEADER` / `RESOURCE`：由 `PUBLIC_HEADER`、`PRIVATE_HEADER` 和 `RESOURCE` 目标属性描述的文件，通常与 MacOS framework 一起使用。
- `FILE_SET <set-name>`：与目标关联的文件集。这是头文件通常的安装方式。

多数重要的产物类型都有已知的安装目的地，除非另有指示，CMake 会默认使用它们。例如，`RUNTIME` 会被安装到 `CMAKE_INSTALL_BINDIR` 变量所指定的位置（若该变量可用），否则默认为 `bin`。

正如我们使用 `cmake -B` 来控制 CMake 使用哪个构建目录一样，我们有多种方式告知 CMake 把东西安装到哪里。这个位置通常被称为安装前缀（install prefix）。要在配置时设置它——使基于该构建树执行的每次 `cmake --install` 都默认使用给定前缀——我们可以使用以下任意一种：

- `cmake --install-prefix` 选项；
- CMake presets 中的 `installDir` 字段；或
- `CMAKE_INSTALL_PREFIX` 变量。

> **注：** 我们一直不建议在项目内部设置 `CMAKE_` 变量。在没有充分理由的情况下设置 `CMAKE_INSTALL_PREFIX` 是*尤其*糟糕的做法，因为它会阻止用户对其进行覆盖。在提供默认值时，项目应检查 `CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT`。

或者，我们可以使用 `cmake --install --prefix` 选项为单次安装调用设置安装前缀。

下表描述了各产物类型默认目的地的完整清单。

| 目标类型 | 变量 | 内置默认值 |
| --- | --- | --- |
| `RUNTIME` | `${CMAKE_INSTALL_BINDIR}` | `bin` |
| `LIBRARY` | `${CMAKE_INSTALL_LIBDIR}` | `lib` |
| `ARCHIVE` | `${CMAKE_INSTALL_LIBDIR}` | `lib` |
| `PRIVATE_HEADER` | `${CMAKE_INSTALL_INCLUDEDIR}` | `include` |
| `PUBLIC_HEADER` | `${CMAKE_INSTALL_INCLUDEDIR}` | `include` |
| `FILE_SET`（类型 `HEADERS`） | `${CMAKE_INSTALL_INCLUDEDIR}` | `include` |

在大多数情况下，项目应保持默认值不变，除非需要安装到某个默认位置的特定子目录。

CMake 默认并不定义 `CMAKE_INSTALL_<dir>` 变量。如果项目希望指定安装到这些位置之一的子目录，就需要包含 `GNUInstallDirs` 模块，它会为所有尚未定义的 `CMAKE_INSTALL_<dir>` 变量提供取值。

## 练习 1 - 安装产物

对于现代的、基于目标的 CMake 项目而言，产物的安装十分简单，只需一次 `install(TARGETS)` 调用即可。

```cmake
install(
  TARGETS MyApp MyLib

  FILE_SET HEADERS
  FILE_SET anotherHeaderFileSet
)
```

大多数产物类型默认会被安装，无需在 `install()` 命令中列出。但 `FILE_SET` 必须显式命名，以告知 CMake 你希望安装它们。上面的示例中我们安装了两个文件集，一个名为 `HEADERS`，另一个名为 `anotherHeaderFileSet`。

当某个产物类型被命名后，就可以为它指定各种选项，例如目的地。

```cmake
include(GNUInstallDirs)

install(
  TARGETS MyApp MyLib

  RUNTIME
    DESTINATION ${CMAKE_INSTALL_BINDIR}/Subfolder

  FILE_SET HEADERS
)
```

这会将 `MyApp` 目标安装到 `bin/Subfolder`（前提是打包者没有修改 `CMAKE_INSTALL_BINDIR`）。

重要的是，如果 `OBJECT` 产物类型从未被给定目的地，它的行为就像 `INTERFACE` 库一样，只安装其头文件。

### 目标

安装教程项目中描述的库与可执行文件（测试除外）的产物。

### 参考资源

- `install()`
- `cmake --install-prefix`
- `cmake --install --prefix`

### 待编辑文件

- `CMakeLists.txt`

### 入门指引

`Help/guide/tutorial/Step9` 目录包含 `Step8` 的完整推荐解决方案。完成 `TODO 1` 和 `TODO 2`。

### 构建与运行

无需特殊配置，按常规方式配置并构建即可。

```console
cmake --preset tutorial
cmake --build build
```

我们可以用 `cmake --install` 验证安装是否正确。

> **注：** 与 CTest 一样，当使用多配置生成器（如 Visual Studio）时，需要通过 `cmake --install --config` 指定配置，例如 `Debug` 或 `Release`。只要使用多配置生成器就必须这样做，后续命令中将不再特别说明。

```console
cmake --install build --prefix install
```

`install` 文件夹中应正确填充我们的产物。

### 解答

首先，我们为条件构建（从而条件安装）的 `Tutorial` 可执行文件添加 `install(TARGETS)`。

**TODO 1: CMakeLists.txt**

```cmake
if(TUTORIAL_BUILD_UTILITIES)
  add_subdirectory(Tutorial)
  install(
    TARGETS Tutorial
  )
endif()
```

然后我们可以安装其余目标。

**TODO 2: CMakeLists.txt**

```cmake
install(
  TARGETS MathFunctions OpAdd OpMul OpSub MathLogger SqrtTable
  FILE_SET HEADERS
)
```

> **注：** 我们也可以将 `install(TARGETS)` 命令局部地添加到定义目标的各个子文件夹中。这在跟踪所有可安装目标变得困难的大型项目中是常见做法。

安装 `SqrtTable` 和 `MathLogger` 看似没有必要，在当前阶段也确实如此。但由于 CMake 建模目标关系的方式，在下一个练习中重建目标模型时，我们将需要这些目标可用。

## 练习 2 - 导出目标

这堆原始的已安装文件是个好开端，但我们丢失了 CMake 的目标模型。它们实际上并不比我们在 `Step 4` 中讨论过的预编译 vendored 库好多少。我们需要某种方式，让其他项目能根据我们在安装树中提供的内容重建我们的目标。

CMake 为解决此问题提供的机制是一种被称为"目标导出文件（target export file）"的 CMakeLang 文件。它由 `install(EXPORT)` 命令创建。

```cmake
install(
  TARGETS MyApp MyLib
  EXPORT MyProjectTargets
)

include(GNUInstallDirs)

install(
  EXPORT MyProjectTargets
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/MyProject
  NAMESPACE MyProject::
)
```

上面的示例包含几个部分。首先，`install(TARGETS)` 命令接收一个导出名，本质上是一个列表，用于将已安装的目标加入其中。

随后，`install(EXPORT)` 命令消费这一目标列表以生成目标导出文件。该文件名为 `<ExportName>.cmake`，位于所提供的 `DESTINATION` 中。本例中提供的 `DESTINATION` 是惯用的位置，但 `find_package()` 命令搜索的任何位置都是有效的。

最后，目标导出文件所创建的目标会以 `NAMESPACE` 字符串为前缀，即形如 `<NAMESPACE><TargetName>`。按惯例，该前缀通常是项目名后跟两个冒号。

出于在后续步骤中将变得更加明显的原因，我们通常不直接使用此文件。相反，我们让一个名为 `<ProjectName>Config.cmake` 的文件通过 `include()` 来使用它。

```cmake
include(${CMAKE_CURRENT_LIST_DIR}/MyProjectTargets.cmake)
```

> **注：** `CMAKE_CURRENT_LIST_DIR` 变量表示当前正在运行的 CMake 语言文件所在的目录，无论该文件是如何被包含或启动的。

然后通过 `install(FILES)` 将此文件与目标导出一起安装。

```cmake
install(
  FILES
    cmake/MyProjectConfig.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/MyProject
)
```

> **注：** 此文件的名称与位置由 `find_package()` 命令的发现语义所规定，我们将在下一步对此详加讨论。

### 目标

导出 Tutorial 项目的目标，以便其他项目可以使用它们。

### 参考资源

- `install()`
- `GNUInstallDirs`
- `CMAKE_CURRENT_LIST_DIR`

### 待编辑文件

- `CMakeLists.txt`
- `cmake/TutorialConfig.cmake`

### 入门指引

继续编辑 `Help/guide/tutorial/Step9` 目录中的文件。完成 `TODO 3` 到 `TODO 8`。

### 构建与运行

构建命令足以重新配置项目。

```console
cmake --build build
```

我们可以用 `cmake --install` 验证安装是否正确。

```console
cmake --install build --prefix install
```

> **注：** CMake 不会更新未发生变化的文件，只会从构建树和源树中安装新增或更新的文件。

`install` 文件夹中应正确填充我们的产物和导出文件。我们将在下一步演示如何使用这些文件。

### 解答

首先，我们将 `Tutorial` 目标添加到 `TutorialTargets` 导出中。

**TODO 3: CMakeLists.txt**

```cmake
  install(
    TARGETS Tutorial
    EXPORT TutorialTargets
  )
```

我们很快就会需要使用 `CMAKE_INSTALL_<dir>` 变量，因此接下来包含 `GNUInstallDirs` 模块。

**TODO 4: CMakeLists.txt**

```cmake
include(GNUInstallDirs)
```

现在我们将其余目标添加到 `TutorialTargets` 导出中。

**TODO 5: CMakeLists.txt**

```cmake
install(
  TARGETS MathFunctions OpAdd OpMul OpSub MathLogger SqrtTable
  EXPORT TutorialTargets
  FILE_SET HEADERS
)
```

接下来我们安装导出本身，以生成目标导出文件。

**TODO 6: CMakeLists.txt**

```cmake
install(
  EXPORT TutorialTargets
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/Tutorial
  NAMESPACE Tutorial::
)
```

然后我们安装"config"文件，我们将用它来包含目标导出文件。

**TODO 7: CMakeLists.txt**

```cmake
install(
  FILES
    cmake/TutorialConfig.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/Tutorial
)
```

最后，我们可以在配置文件中添加必要的 `include()` 命令。

**TODO 8: cmake/TutorialConfig.cmake**

```cmake
include(${CMAKE_CURRENT_LIST_DIR}/TutorialTargets.cmake)
```

## 练习 3 - 导出版本文件

当从目标导出文件导入 CMake 目标时，没有办法"中途退出"或"撤销"该操作。如果发现某个包对于我们所请求的版本是错误或不兼容的，我们就会在获知版本信息的过程中产生的任何副作用上陷入僵局。

CMake 针对此问题给出的答案是一个轻量级的版本文件，它仅描述版本兼容性信息，可在 CMake 完全提交导入之前进行检查。

CMake 提供了用于生成这些版本文件的辅助模块和脚本，即 `CMakePackageConfigHelpers` 模块。

```cmake
include(CMakePackageConfigHelpers)

write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfigVersion.cmake
  COMPATIBILITY ExactVersion
)
```

可用的兼容性版本包括：

- `AnyNewerVersion`
- `SameMajorVersion`
- `SameMinorVersion`
- `ExactVersion`

此外，包可以将自身标记为 `ARCH_INDEPENDENT`，适用于不附带会将其绑定到特定机器架构的二进制文件的包。

默认情况下，`write_basic_package_version_file()` 所使用的 `VERSION` 是提供给 `project()` 命令的 `VERSION` 号。

### 目标

为 Tutorial 项目导出一个版本文件。

### 参考资源

- `project()`
- `install()`
- `CMakePackageConfigHelpers`
- `PROJECT_VERSION`

### 待编辑文件

- `CMakeLists.txt`

### 入门指引

继续编辑 `Help/guide/tutorial/Step9` 目录中的文件。完成 `TODO 9` 到 `TODO 12`。

### 构建与运行

按之前的方式重新构建并安装。

```console
cmake --build build
cmake --install build --prefix install
```

`install` 文件夹中应正确填充我们新生成并安装的版本文件。

### 解答

首先，我们为 `project()` 命令添加 `VERSION` 参数。

**TODO 9: CMakeLists.txt**

```cmake
project(Tutorial
  VERSION 1.0.0
)
```

接下来，我们包含 `CMakePackageConfigHelpers` 模块，并用它生成配置版本文件。

**TODO 10-11: CMakeLists.txt**

```cmake
include(CMakePackageConfigHelpers)

write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/TutorialConfigVersion.cmake
  COMPATIBILITY ExactVersion
)
```

最后，我们将配置版本文件添加到待安装文件列表中。

**TODO 12: CMakeLists.txt**

```cmake
install(
  FILES
    cmake/TutorialConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/TutorialConfigVersion.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/Tutorial
)
```
