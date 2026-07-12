# 02 目标与属性模型（Targets & Properties）

> 主题：理解"目标即数据"的模型，掌握 `target_*` 命令族、`PUBLIC`/`PRIVATE`/`INTERFACE` 三态语义与传递依赖。

## 1. 从"目录式"到"目标式"

早期 CMake 习惯在目录层级设置编译选项：

```cmake
# 旧式（不推荐）
include_directories(${CMAKE_SOURCE_DIR}/include)
add_definitions(-DFOO)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")
add_library(libdemo src/demo.cpp)
```

这种写法的问题是**作用域是全局/目录级的**：本目录及子目录所有目标都被影响，难以隔离，且无法表达"我这个库需要这些头文件，但使用者不需要"的传递边界。

现代 CMake 以**目标（target）**为组织单位，把选项、依赖、特性都绑定到具体目标上：

```cmake
# 现代写法
add_library(libdemo src/demo.cpp)
target_include_directories(libdemo PUBLIC include/)
target_compile_options(libdemo PRIVATE -Wall)
target_compile_features(libdemo PUBLIC cxx_std_17)
```

核心思想：**目标是一等公民，目标属性是数据**。构建系统的所有信息——头文件路径、链接库、编译选项、语言标准——都以属性形式挂在目标上，下游通过"链接目标"自动获取该公开的属性。这就是"传递依赖"的基础。

## 2. `target_*` 命令族

| 命令 | 设置的属性 | 典型用途 |
|------|------------|----------|
| `target_include_directories` | `INTERFACE_INCLUDE_DIRECTORIES` / `INCLUDE_DIRECTORIES` | 头文件搜索路径 |
| `target_link_libraries` | `LINK_LIBRARIES` / `INTERFACE_LINK_LIBRARIES` | 链接依赖 |
| `target_compile_options` | `COMPILE_OPTIONS` / `INTERFACE_COMPILE_OPTIONS` | 编译选项（警告、宏等） |
| `target_compile_definitions` | `COMPILE_DEFINITIONS` / `INTERFACE_COMPILE_DEFINITIONS` | 预处理宏 |
| `target_compile_features` | `COMPILE_FEATURES` / `INTERFACE_COMPILE_FEATURES` | 语言特性（`cxx_std_17` 等） |
| `target_sources` | `SOURCES` | 追加源文件（CMake 3.13+ 可用于子目录目标） |
| `target_link_directories` | `LINK_DIRECTORIES` | 链接搜索目录（尽量少用，优先 `IMPORTED` 目标） |
| `target_link_options` | `LINK_OPTIONS` / `INTERFACE_LINK_OPTIONS` | 链接器选项 |

每个命令都接受 `PUBLIC`/`PRIVATE`/`INTERFACE` 关键字中的一个或多个来决定属性的传递范围（见下节）。

## 3. `PUBLIC` / `PRIVATE` / `INTERFACE` 三态

这是现代 CMake 最关键、也最容易混淆的概念。它回答的是：**这个属性是给"自己编译"用、给"下游链接我的人"用，还是两者都给？**

- `PRIVATE`：**仅自己**编译本目标的源文件时使用。不向下游传递。
- `INTERFACE`：**仅下游**链接本目标时使用。自己编译源文件不用。
- `PUBLIC`：`PRIVATE` + `INTERFACE`，**自己用，也传给下游**。

判断窍门——对每项依赖问两个问题：

1. **我自己编译 `libdemo.cpp` 时需要它吗？**（比如 `#include` 了它的头文件）
2. **下游链接 `libdemo` 时需要它吗？**（比如我的公开头文件里 `#include` 了它的头文件）

两个"是" → `PUBLIC`；只有第 1 个 → `PRIVATE`；只有第 2 个 → `INTERFACE`。

### 3.1 三类例子

```cmake
add_library(libdemo src/demo.cpp)

# 我的 demo.cpp 里 #include <demo.h>，下游用 libdemo 也 #include <demo.h>
target_include_directories(libdemo PUBLIC include/)

# 我的 demo.cpp 内部用 spdlog，但公开头文件 demo.h 不暴露 spdlog 类型
target_link_libraries(libdemo PRIVATE spdlog::spdlog)

# 我是个纯头文件库(interface header-only)，自己没有源文件要编译，
# 但下游用我时需要 c++17
add_library(libdemo_iface INTERFACE)
target_compile_features(libdemo_iface INTERFACE cxx_std_17)
```

### 3.2 为什么 INTERFACE 库自己也要 `INTERFACE`

`INTERFACE` 库没有源文件，不产生构建产物，它的意义就是**向下游传递一组属性**。因此它身上几乎所有属性都用 `INTERFACE`（`PRIVATE` 对它无意义——它没有自己的源文件可编译）。这也是为什么头文件库、纯配置库适合用 `INTERFACE` 类型。

## 4. 传递依赖如何工作

当目标 A 链接目标 B（`target_link_libraries(A PUBLIC B)`），CMake 会把 B 的 `INTERFACE_*` 属性"并"到 A 的对应属性里。这有两条主要的传递链：

### 4.1 头文件路径的传递（`INTERFACE_INCLUDE_DIRECTORIES`）

下游编译自己源文件时，需要能找到上游公开头文件。因此 `PUBLIC` 的 include 目录会沿链接链传递。

```
app → libdemo(PUBLIC include/) → fmt(PUBLIC include/)
```

编译 `app.cpp` 时，搜索路径里同时有 `libdemo/include` 和 `fmt/include`，因为两者都是 `PUBLIC`。

### 4.2 链接库的传递（`INTERFACE_LINK_LIBRARIES`）

这是 CMake 行为里最需要留意的部分，存在"链接传递性"的细节：

- 默认情况下，`PRIVATE` 链接的库**不**自动传给再下游（它们是 `libdemo` 的实现细节）。
- `PUBLIC`/`INTERFACE` 链接的库会进入 `INTERFACE_LINK_LIBRARIES`，继续向再下游传递。
- CMake 3.13+ 对静态库的 `PRIVATE` 依赖有"间接传递"机制（`LINK_INTERFACE_MULTIPLICITY`、静态库符号解析的连锁需求），但这属于例外，不应作为设计依赖。

工程上的原则：**让传递性反映"接口暴露"**。公开头文件里用到的依赖 → `PUBLIC`；只在 `.cpp` 里用到的 → `PRIVATE`。这样下游不会意外获得它不需要的头文件路径与链接库，构建更快、更隔离。

### 4.3 编译选项/宏/特性的传递

`INTERFACE_COMPILE_OPTIONS`、`INTERFACE_COMPILE_DEFINITIONS`、`INTERFACE_COMPILE_FEATURES` 同样沿 `PUBLIC`/`INTERFACE` 链传递。常见用法：

```cmake
# 库要求下游也用 c++17 编译（因为我的公开头文件用了 c++17 特性）
target_compile_features(libdemo PUBLIC cxx_std_17)

# 库要求下游定义 USE_DEMO_API（因为公开头文件里 #ifdef USE_DEMO_API）
target_compile_definitions(libdemo PUBLIC USE_DEMO_API=1)

# 警告只我自己用，不强迫下游
target_compile_options(libdemo PRIVATE -Wall)
```

## 5. 别名目标（ALIAS）

`add_library(NS::lib ALIAS libdemo)` 创建一个不可修改的别名。用途：

- 让依赖名带命名空间，统一风格（`myproj::core`、`Boost::filesystem`）。
- 区分"本工程构建的目标"与"导入的目标"，下游代码可 `if(TARGET NS::lib)` 判断。
- 写可被 `FetchContent`/`find_package` 双模式消费的库时，别名让上层代码无需关心库来自本地还是导入。

别名不可被 `install(EXPORT)` 导出（导出目标本身），也不可被 `set_target_properties` 修改——它只是一个稳定引用。

## 6. `IMPORTED` 目标（预览）

`IMPORTED` 目标表示"已经存在于系统里、不由本工程构建"的目标，常由 `find_package` 提供的现代配置文件创建（如 `Threads::Threads`、`ZLIB::ZLIB`、`fmt::fmt`）。它们带 `IMPORTED` 属性，不可用 `target_sources` 等修改，但可以用 `target_link_libraries(myapp PRIVATE fmt::fmt)` 正常链接，并享受传递依赖。

这是现代 CMake 依赖管理的推荐形态——比旧的 `include_directories(${ZLIB_INCLUDE_DIRS})` + `link_directories(...)` 健壮得多。详见笔记 [03 依赖管理](03-Dependency-Management.md)。

## 7. 常见反模式

### 7.1 全局污染

```cmake
include_directories(include)       # 影响整个目录树
add_definitions(-DFOO)             # 同上
link_directories(/opt/lib)         # 同上
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")  # 字符串拼接，难追踪
```

这些命令应优先用对应的 `target_*` 替换，把作用域收窄到具体目标。

### 7.2 把 `PRIVATE` 依赖当 `PUBLIC`

把只在 `.cpp` 用的依赖标成 `PUBLIC`，会导致下游被迫拉入无关头文件与链接库，增加编译时间、暴露实现细节，并制造"删掉某个 PRIVATE 依赖就编译不过"的虚假耦合。

### 7.3 用 `LINK_PUBLIC`/`LINK_PRIVATE` 旧关键字

CMake 早期用 `target_link_libraries(A LINK_PUBLIC B)` 表示传递。新代码统一用 `PUBLIC`/`PRIVATE`/`INTERFACE`，旧关键字仅为兼容保留，不要混用。

### 7.4 手写字符串拼接的标志

```cmake
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17")  # 不推荐
```

应改为 `target_compile_features(t PUBLIC cxx_std_17)`，让 CMake 按编译器选择正确标志并传递给下游。

## 8. 小结

- **目标即数据**：所有信息以属性挂在目标上，下游靠链接自动获取，构成传递依赖。
- **三态语义**：`PRIVATE`=自己用、`INTERFACE`=给下游、`PUBLIC`=两者。按"公开头文件是否暴露该依赖"判断。
- **接口库**：无源文件、纯传递属性的 `INTERFACE` 目标，适合头文件库与配置聚合库。
- **别名与 IMPORTED**：`ALIAS` 提供稳定命名，`IMPORTED` 承载外部依赖，二者让消费方代码统一。
- **反模式**：避免目录级全局命令、字符串拼标志、`PRIVATE` 误当 `PUBLIC`。

上一篇：[01 生成器表达式](01-Generator-Expressions.md)　下一篇：[03 依赖管理](03-Dependency-Management.md)
