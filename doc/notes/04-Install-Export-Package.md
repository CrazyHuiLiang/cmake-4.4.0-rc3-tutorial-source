# 04 安装、导出与打包（Install, Export & Package）

> 主题：`install()` 各命令、`install(EXPORT)` 导出目标、生成 `xxxConfig.cmake` 让下游 `find_package`、CPack 打包。

本篇把"把库安装出去并让下游能 `find_package`"的完整链路讲清楚，并在最后用 CPack 把产物打成可分发包。它与笔记 [01](01-Generator-Expressions.md) 的 `BUILD_INTERFACE`/`INSTALL_INTERFACE`、笔记 [02](02-Targets-and-Properties.md) 的目标属性、笔记 [03](03-Dependency-Management.md) 的 `find_package` 产物首尾呼应。

## 1. `install()` 命令族概览

`install()` 在配置阶段声明"安装时做什么"，实际安装发生在 `cmake --install` 阶段。常用形态：

| 命令 | 安装对象 |
|------|----------|
| `install(TARGETS ...)` | 构建产物（库、可执行、对象库等），按产物类型分桶 |
| `install(FILES ...)` | 单个文件（头文件、配置文件）原样安装 |
| `install(DIRECTORY ...)` | 整个目录树（常用于头文件目录） |
| `install(PROGRAMS ...)` | 可执行脚本（设置可执行位） |
| `install(EXPORT ...)` | 导出目标属性集合，供下游导入 |
| `install(SCRIPT/CODE ...)` | 自定义安装脚本/代码 |

安装路径用 GNUInstallDirs 提供的变量，避免硬编码：

```cmake
include(GNUInstallDirs)
# ${CMAKE_INSTALL_BINDIR}      bin
# ${CMAKE_INSTALL_LIBDIR}      lib 或 lib64
# ${CMAKE_INSTALL_INCLUDEDIR}  include
# ${CMAKE_INSTALL_DATADIR}     share
```

## 2. `install(TARGETS)` 与产物分桶

一个目标可能产生多种产物：共享库 `.so`/`.dll`、静态库 `.a`/`.lib`、运行时 `.exe`、导入库 `.lib`（Windows）、调试符号等。`install(TARGETS)` 用权限段把它们分别送到不同目录：

```cmake
install(TARGETS libdemo
    EXPORT libdemoTargets              # 归入导出集 libdemoTargets（见第 4 节）
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}     # 静态库 / Windows 导入库
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}     # 共享库
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}     # 可执行 / DLL
    OBJECTS  DESTINATION ${CMAKE_INSTALL_LIBDIR}    # 对象库
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}  # 公共头文件
)
```

要点：

- `ARCHIVE`/`LIBRARY`/`RUNTIME`/`OBJECTS`/`PUBLIC_HEADER`/`PRIVATE_HEADER`/`FRAMEWORK`/`BUNDLE` 都是可选的分桶关键字，按产物实际类型命中。
- 同一个共享库在 Linux 上走 `LIBRARY`，在 Windows 上 `.dll` 走 `RUNTIME`、导入库 `.lib` 走 `ARCHIVE`——这套分桶让你一份声明跨平台正确。
- `EXPORT` 把该目标登记到导出集，是让下游能 `find_package` 的关键，见第 4 节。

## 3. 安装头文件

两种风格：

```cmake
# 风格 A：整目录安装（推荐，保持目录结构）
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

# 风格 B：用 install(TARGETS ... PUBLIC_HEADER ...) 只装公开头
```

风格 A 更常用，注意路径末尾的 `/`（CMake 约定：源目录带尾 `/` 表示装"目录内容"而非"目录本身"）。

## 4. `install(EXPORT)`：导出目标属性

仅安装产物文件还不够——下游还需要知道"这个库的头文件在哪、依赖什么、编译特性是什么"，才能 `target_link_libraries(myapp PRIVATE libdemo::libdemo)`。这正是 `install(EXPORT)` 做的：它把导出集里目标的 `INTERFACE_*` 属性序列化成一个 `xxxTargets.cmake` 文件，里面定义带命名空间的 `IMPORTED` 目标。

```cmake
install(EXPORT libdemoTargets
    FILE libdemoTargets.cmake
    NAMESPACE libdemo::              # 下游用 libdemo::libdemo 引用
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/libdemo
)
```

安装后，下游安装树里会有：

```
lib/cmake/libdemo/libdemoTargets.cmake
```

但下游是用 `find_package(libdemo)` 来定位的，而 `find_package` 的 Config 模式找的是 `libdemoConfig.cmake`，不是 `libdemoTargets.cmake`。所以还需要写一个 Config 文件并在其中 `include()` 那个 Targets 文件——见第 5 节。

## 5. 生成 `xxxConfig.cmake` 与版本文件

`find_package(libdemo CONFIG)` 会找 `libdemoConfig.cmake`。这个文件通常由模板生成：用 `configure_package_config_file` 把模板里的占位符替换为实际安装路径，并 `include()` 上面导出的 `libdemoTargets.cmake`。

模板 `cmake/libdemoConfig.cmake.in`（节选）：

```cmake
@PACKAGE_INIT@

include("${CMAKE_CURRENT_LIST_DIR}/libdemoTargets.cmake")

# 可选：提供变量形式的便捷查询
set(libdemo_INCLUDE_DIRS "@PACKAGE_CMAKE_INSTALL_INCLUDEDIR@")
check_required_components(libdemo)
```

`@PACKAGE_INIT@` 宏会定义 `PACKAGE_PREFIX_DIR` 等，使 `@PACKAGE_<VAR>@` 解析为相对安装前缀的路径——这正是 `INSTALL_INTERFACE`（笔记 01）里那些相对路径能被正确解析的原因。

在 `CMakeLists.txt` 里生成：

```cmake
include(CMakePackageConfigHelpers)

# 版本文件
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/libdemoConfigVersion.cmake"
    VERSION 1.0.0
    COMPATIBILITY SameMajorVersion)

# Config 文件
configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/libdemoConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/libdemoConfig.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/libdemo)

# 安装这两个文件
install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/libdemoConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/libdemoConfigVersion.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/libdemo)
```

`COMPATIBILITY SameMajorVersion` 表示下游 `find_package(libdemo 1.2)` 时，只要主版本号一致即视为兼容。其他策略有 `AnyNewerVersion`、`ExactVersion`、`SameMinorVersion` 等。

## 6. 让下游 `find_package` 能找到

默认 `find_package` 只在标准路径（如 `/usr/lib/cmake`、`/usr/local/lib/cmake`）找。装到非标准前缀时，下游需要提示路径：

```cmake
# 下游用法
list(APPEND CMAKE_PREFIX_PATH "/opt/libdemo")   # 或用 CMAKE_PREFIX_PATH 环境变量
find_package(libdemo 1.0 CONFIG REQUIRED)
target_link_libraries(myapp PRIVATE libdemo::libdemo)
```

这与笔记 [03](03-Dependency-Management.md) 里 IMPORTED 目标消费完全一致——我们自己造出了一个能被 `find_package` 找到、提供命名空间目标的 Config 包。

## 7. `BUILD_INTERFACE` / `INSTALL_INTERFACE` 在此收尾

回顾笔记 01 的写法：

```cmake
target_include_directories(libdemo PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)
```

- 本工程构建时，展开为源码里的 `include/`，开发期改头文件即时生效。
- 安装导出后，`libdemoTargets.cmake` 里记录的是 `INSTALL_INTERFACE` 的值——相对安装前缀的 `include`，由 `@PACKAGE_INIT@` 解析为绝对路径。
- 因此下游 `find_package` 后 `target_link_libraries` 自动获得正确的安装树头文件路径。

这就是为什么导出能正确工作：**生成器表达式让同一个目标属性在两种场景给出不同值，而 `install(EXPORT)` 只序列化 `INSTALL_INTERFACE` 的结果。**

## 8. CPack 打包

CPake 在 `install()` 声明的基础上，把安装树打成可分发包。最简配置：

```cmake
set(CPACK_PACKAGE_NAME "libdemo")
set(CPACK_PACKAGE_VERSION "1.0.0")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "A demo CMake library")
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
include(CPack)
```

`include(CPack)` 读取这些变量，生成打包目标。生成器由平台与 `CPACK_GENERATOR` 决定：

- Windows：`ZIP`、`NSIS`（生成 `.exe` 安装器）
- Linux：`TGZ`、`DEB`（Debian/Ubuntu）、`RPM`（Fedora/RHEL）
- macOS：`DragNDrop`（`.dmg`）、`ZIP`

指定生成器：

```cmake
set(CPACK_GENERATOR "ZIP;TGZ")   # 或在命令行 -DCPACK_GENERATOR=DEB
```

DEB/RPM 还需额外元数据（维护者、依赖、章节等），例如：

```cmake
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "you@example.com")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libc6 (>= 2.31)")
```

构建产物后执行：

```console
cmake --install .                 # 先安装到临时树
cpack                             # 打包（默认当前目录的 build tree）
cpack -G DEB                      # 指定生成器
```

### 8.1 组件打包

复杂项目可把产物分成多个组件（runtime/development），分别打包：

```cmake
install(TARGETS libdemo
    EXPORT libdemoTargets
    COMPONENT runtime
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(FILES include/libdemo.h
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    COMPONENT development)

set(CPACK_COMPONENTS_ALL runtime development)
set(CPACK_COMPONENT_RUNTIME_DISPLAY_NAME "libdemo runtime")
set(CPACK_COMPONENT_DEVELOPMENT_DISPLAY_NAME "libdemo development files")
include(CPack)
```

下游用户可只装 runtime 组件得到运行所需 `.so`，开发机再装 development 组件获得头文件与 CMake 配置。

## 9. 完整示例

把全篇串起来（库 `libdemo`，公开头文件，导出 Config 包，可用 CPack 打包）：

```cmake
cmake_minimum_required(VERSION 3.20)
project(libdemo VERSION 1.0.0 LANGUAGES CXX)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

add_library(libdemo src/demo.cpp)
add_library(libdemo::libdemo ALIAS libdemo)

target_include_directories(libdemo PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)
target_compile_features(libdemo PUBLIC cxx_std_17)

# 1) 安装产物 + 登记导出集
install(TARGETS libdemo
    EXPORT libdemoTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

# 2) 导出 Targets 文件
install(EXPORT libdemoTargets
    FILE libdemoTargets.cmake
    NAMESPACE libdemo::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/libdemo)

# 3) 生成并安装 Config / ConfigVersion
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/libdemoConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion)

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/libdemoConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/libdemoConfig.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/libdemo)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/libdemoConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/libdemoConfigVersion.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/libdemo)

# 4) CPack
set(CPACK_PACKAGE_NAME "libdemo")
set(CPACK_PACKAGE_VERSION ${PROJECT_VERSION})
include(CPack)
```

验证安装：

```console
cmake -S . -B build -DCMAKE_INSTALL_PREFIX=/tmp/prefix
cmake --build build
cmake --install build

# 确认产物
ls /tmp/prefix/lib/cmake/libdemo/        # libdemoConfig.cmake, libdemoTargets.cmake, ...

# 下游消费
cmake -S /path/to/downstream -B /tmp/dbuild -DCMAKE_PREFIX_PATH=/tmp/prefix
# 下游 CMakeLists.txt 内：
#   find_package(libdemo 1.0 CONFIG REQUIRED)
#   target_link_libraries(app PRIVATE libdemo::libdemo)

# 打包
cd build && cpack -G TGZ
```

## 10. 小结

- **安装**：`install(TARGETS)` 按 `ARCHIVE/LIBRARY/RUNTIME/...` 分桶，配合 `GNUInstallDirs` 跨平台。
- **导出**：`install(EXPORT)` 把目标 `INTERFACE_*` 属性序列化为 `xxxTargets.cmake`，定义带命名空间的 IMPORTED 目标。
- **Config 包**：`configure_package_config_file` + `write_basic_package_version_file` 生成 `xxxConfig.cmake`/`xxxConfigVersion.cmake`，让下游 `find_package` 可用。
- **双轨头文件**：`BUILD_INTERFACE`/`INSTALL_INTERFACE` 让同一目标在构建树与安装树给出不同 include 路径，导出时只保留安装树值。
- **打包**：`include(CPack)` + 生成器选择，把安装树打成 ZIP/DEB/RPM/NSIS 等，组件化可分运行期与开发期。

系列结束。回看：[01 生成器表达式](01-Generator-Expressions.md)｜[02 目标与属性模型](02-Targets-and-Properties.md)｜[03 依赖管理](03-Dependency-Management.md)
