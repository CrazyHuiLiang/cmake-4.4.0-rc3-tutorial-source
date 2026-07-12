# 03 依赖管理（Dependency Management）

> 主题：`find_package` 的两种模式与现代产物、`FetchContent` 源码引入、传递性依赖与避免重复引入。

## 1. 依赖的两类来源

工程依赖通常来自两个地方：

- **系统已安装的包**：用 `find_package` 在标准路径或用户提示路径下查找，得到一组变量或一个 `IMPORTED` 目标。
- **源码**：把依赖源码纳入本工程构建，用 `add_subdirectory`（依赖在本仓库内）或 `FetchContent`（从远程拉取）。

选择原则：能找到稳定系统包且版本可控时用 `find_package`；需要固定版本/打补丁/随源码分发时用源码引入。两者可共存：让用户用选项切换。

## 2. `find_package` 的两种查找模式

`find_package(Pkg)` 内部按顺序尝试两种机制：

### 2.1 Module 模式

查找 `Find<Pkg>.cmake` 文件（通常由 CMake 自带或工程自己提供，放在 `CMAKE_MODULE_PATH`）。这类"Find 模块"是 CMake 帮你写的查找脚本，产出的是**变量**（`<Pkg>_FOUND`、`<Pkg>_INCLUDE_DIRS`、`<Pkg>_LIBRARIES`）。它适用于上游没有提供 CMake 配置文件的旧库。

### 2.2 Config 模式

查找上游安装时随附的 `<Pkg>Config.cmake` / `<Pkg>-config.cmake`。这是上游用 `install(EXPORT)` + `configure_package_config_file` 生成的（见笔记 [04](04-Install-Export-Package.md)）。Config 模式产出的通常是**带命名空间的 IMPORTED 目标**（如 `fmt::fmt`、`Boost::filesystem`），这是现代推荐的形态。

> Module 模式先于 Config 模式，除非用 `find_package(Pkg CONFIG)` 或 `MODULE` 限定。多数情况下让 CMake 自动选择即可，但新库更应依赖 Config 模式。

### 2.3 版本与组件

```cmake
find_package(Boost 1.80 REQUIRED COMPONENTS filesystem system
                                   OPTIONAL_COMPONENTS log)
```

- `1.80`：最低版本要求。
- `REQUIRED`：找不到就报错终止。
- `COMPONENTS`：必需组件；`OPTIONAL_COMPONENTS`：可选组件。
- 找到后用 `<Pkg>_FOUND` 判断，组件用 `Boost_FILESYSTEM_FOUND` 或 `if(TARGET Boost::filesystem)` 判断。

## 3. 现代 `find_package` 产物：IMPORTED 目标 vs 变量

旧式 Find 模块给变量：

```cmake
find_package(ZLIB REQUIRED)
target_include_directories(app PRIVATE ${ZLIB_INCLUDE_DIRS})  # 旧式
target_link_libraries(app PRIVATE ${ZLIB_LIBRARIES})
```

现代 Config 文件给 IMPORTED 目标：

```cmake
find_package(ZLIB REQUIRED)
target_link_libraries(app PRIVATE ZLIB::ZLIB)   # 现代：一行搞定，含 include 与传递依赖
```

IMPORTED 目标的优势：头文件路径、链接库、传递依赖、编译特性都打包在目标里，下游一行 `target_link_libraries` 全部获得；而变量方式要手动拼 `include_directories` + `link_libraries`，且不传递。

> **判断习惯**：`if(TARGET NS::lib)` 优先于 `if(NS_FOUND)`。前者确认目标真的存在且可链接，更可靠。

## 4. `FetchContent`：源码引入

`FetchContent`（CMake 3.11+，3.14+ 体验成熟）在配置阶段从 Git 仓库或 URL 拉取依赖源码，再用 `add_subdirectory` 把它纳入本工程构建。

```cmake
include(FetchContent)

FetchContent_Declare(
    fmt
    GIT_REPOSITORY https://github.com/fmtlib/fmt.git
    GIT_TAG        10.1.1
)
FetchContent_MakeAvailable(fmt)   # 3.14+，自动 add_subdirectory

target_link_libraries(app PRIVATE fmt::fmt)
```

要点：

- `GIT_TAG` 用具体的 tag/commit，不要用 `master`/`main`，避免不可复现构建。
- `FetchContent_MakeAvailable`（3.14+）会自动 `add_subdirectory`，并对已声明但尚未拉取的依赖做拉取。
- 默认下载内容缓存在 `${CMAKE_BINARY_DIR}/_deps`，多次配置不重复下载。
- 网络受限时可用 `GIT_SHALLOW FALSE`、离线镜像、`FETCHCONTENT_SOURCE_DIR_<name>` 指向本地已检出版本来跳过下载。

### 4.1 `FetchContent` vs `add_subdirectory`

- `add_subdirectory` 依赖源码已在工程内（如 submodule）。
- `FetchContent` 自动拉取，适合"只想声明一个远程依赖"的场景。
- 二者纳入的目标都是真实构建目标，享受完整的传递依赖。

## 5. 双模式：让用户选系统包或源码

健壮的工程常让最终构建者二选一：

```cmake
option(DEMO_USE_SYSTEM_FMT "Use system-installed fmt" OFF)

if(DEMO_USE_SYSTEM_FMT)
    find_package(fmt 10.1 CONFIG REQUIRED)
else()
    include(FetchContent)
    FetchContent_Declare(fmt
        GIT_REPOSITORY https://github.com/fmtlib/fmt.git
        GIT_TAG 10.1.1)
    FetchContent_MakeAvailable(fmt)
endif()

# 无论哪条路径，都存在 fmt::fmt 目标，下游代码统一
target_link_libraries(app PRIVATE fmt::fmt)
```

关键在于：两种路径都产出**同名目标**（这里是 `fmt::fmt`），上层代码无需关心来源。这正是别名目标（笔记 02）的价值——让本地构建目标和导入目标共享命名。

## 6. 传递性依赖：下游自动获益

依赖传递依赖 `target_link_libraries` 的 `PUBLIC`/`INTERFACE` 语义（笔记 02）：

```cmake
add_library(libdemo src/demo.cpp)
# libdemo 公开头文件用了 fmt，故 fmt 是 PUBLIC
target_link_libraries(libdemo PUBLIC fmt::fmt)

add_executable(app src/main.cpp)
# app 链接 libdemo 即自动获得 fmt 的头文件路径与链接库
target_link_libraries(app PRIVATE libdemo)
```

设计要点：**把"公开头文件是否暴露某依赖"作为 PUBLIC/PRIVATE 的判据**。这样依赖边界清晰，下游只获得它真正需要的传递依赖。

## 7. 避免重复引入与版本冲突

大型工程里同一依赖可能被多个子项目/多个 FetchContent 声明，处理不当会出现重复 `add_library`、版本打架。

### 7.1 `FetchContent` 的去重

`FetchContent_Declare` 对同名依赖只能声明一次有效配置；重复 `MakeAvailable` 同一名字不会重复 `add_subdirectory`。CMake 3.24+ 的 `OVERRIDE_FIND_PACKAGE` 可让 `FetchContent` 接管对应的 `find_package`：

```cmake
FetchContent_Declare(fmt
    GIT_REPOSITORY https://github.com/fmtlib/fmt.git
    GIT_TAG 10.1.1
    OVERRIDE_FIND_PACKAGE   # 3.24+：让 find_package(fmt) 也走这里
)
FetchContent_MakeAvailable(fmt)
# 此后 find_package(fmt) 会复用上面拉取的 fmt，而非去系统找
```

### 7.2 让 `find_package` 与 `FetchContent` 协同

`FetchContent_MakeAvailable` 在拉取前会先尝试 `find_package(<name>)`（除非禁用）。利用这一点可实现"有系统包就用系统包，否则拉源码"：

```cmake
set(FETCHCONTENT_TRY_FIND_PACKAGE_MODE ALWAYS)  # 3.24+，优先 find_package
FetchContent_Declare(fmt GIT_REPOSITORY ... GIT_TAG ...)
FetchContent_MakeAvailable(fmt)
```

低版本可手写 `if(TARGET fmt::fmt) ... else() FetchContent ... endif()` 实现等价逻辑。

### 7.3 版本冲突的处理思路

当两个依赖各自 FetchContent 拉了不同版本的同一底层库，会冲突。常见解法：

- 升级到统一版本，让一方放弃自带旧版。
- 用 `FetchContent` 的去重特性，在顶层工程只声明一次该底层库，子依赖复用之（前提是子依赖也用 `FetchContent` 且不强制自带）。
- 对无法统一的，考虑把冲突依赖隔离到不同可执行文件，避免链接到同一目标。

## 8. 小结

- **两种来源**：`find_package`（系统/Config 包）与 `FetchContent`（源码），可按选项切换。
- **两种查找模式**：Module 模式产变量、Config 模式产 IMPORTED 目标，优先 Config。
- **判断用 `TARGET`**：`if(TARGET NS::lib)` 比 `<Pkg>_FOUND` 更可靠。
- **双模式**：让系统包与源码都产出同名目标，上层代码统一。
- **传递靠 PUBLIC/INTERFACE**：公开头文件暴露的依赖才标 `PUBLIC`。
- **去重**：`FetchContent` 同名只生效一次；3.24+ 用 `OVERRIDE_FIND_PACKAGE` / `FETCHCONTENT_TRY_FIND_PACKAGE_MODE` 协调 `find_package`。

上一篇：[02 目标与属性模型](02-Targets-and-Properties.md)　下一篇：[04 安装导出与打包](04-Install-Export-Package.md)
