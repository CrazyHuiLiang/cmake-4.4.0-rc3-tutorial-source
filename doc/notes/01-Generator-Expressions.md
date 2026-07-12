# 01 生成器表达式（Generator Expressions）

> 主题：理解 CMake 生成阶段求值机制，掌握 `$<...>` 语法与高频用法，规避常见陷阱。

## 1. 为什么需要生成器表达式

CMake 的运行分为两个阶段：

- **配置阶段（configure）**：读取 `CMakeLists.txt`，解析命令、设置变量与目标属性。`if()`、`message()`、变量展开 `${VAR}` 都在这一阶段求值。
- **生成阶段（generate）**：根据配置结果，为具体的生成器（Make/Ninja/VS/Xcode）产出构建文件。多配置生成器（如 Visual Studio、Xcode）在这一阶段才区分 `Debug`/`Release`；目标的最终输出路径、链接语言、依赖文件名也只有在这时才完全确定。

问题在于：很多信息在配置阶段**还不知道**。例如：

- 多配置生成器下，一次配置要同时支持 `Debug` 和 `Release`，配置阶段无法 `if(CMAKE_BUILD_TYPE STREQUAL Debug)` 来"二选一"——因为两者都要生成。
- `target_compile_options` 里想引用"本目标的输出文件路径"，但文件名要到生成阶段才能算出。
- 想对"最终会被 MSVC 编译的翻译单元"加 `/utf-8`，但一个目标可能混合 C/CXX，配置阶段说不清。

生成器表达式就是为这些**生成阶段才能确定**的值设计的求值机制。它写成 `$<...>`，在生成阶段被求值并替换。

一句话区分：**`${VAR}` 在配置阶段展开，`$<...>` 在生成阶段求值。**

## 2. 语法基础

生成器表达式以 `$<` 开头、`>` 结尾，常见两种形态：

- `$<NAME:args>`：名为 `NAME` 的表达式，带参数 `args`，求值后替换为结果。
- `$<CONDITION:true>`：当 `CONDITION` 为真时求值为 `true`，否则为空字符串。这是条件表达式的基础。

参数之间用逗号分隔，例如 `$<IF:cond,a,b>`（三目）。嵌套时直接书写：

```cmake
$<$<CONFIG:Debug>:-O0 -g>
#  等价于：若 CONFIG 为 Debug，则展开为 "-O0 -g"，否则为空
```

可读性建议：复杂表达式用空格分隔多个表达式提升可读性，CMake 3.12+ 支持在表达式中换行（`$<` 与 `>` 之间）。

## 3. 常用表达式分类

### 3.1 逻辑与比较

| 表达式 | 含义 |
|--------|------|
| `$<BOOL:x>` | 将 `x` 转为布尔（`0`/`OFF`/`NO`/`FALSE`/`N`/`IGNORE`/`NOTFOUND`/空/`*-NOTFOUND` 为假，其余为真） |
| `$<AND:a,b,...>` | 逻辑与 |
| `$<OR:a,b,...>` | 逻辑或 |
| `$<NOT:a>` | 逻辑非 |
| `$<STREQUAL:a,b>` | 字符串相等 |
| `$<VERSION_LESS:a,b>` 等 | 版本比较 |
| `$<IF:cond,a,b>` | 三目：真则 `a`，假则 `b` |

### 3.2 目标相关（最高频）

| 表达式 | 含义 |
|--------|------|
| `$<TARGET_EXISTS:t>` | 目标 `t` 是否存在 |
| `$<TARGET_PROPERTY:t,prop>` | 目标 `t` 的属性 `prop` 的值 |
| `$<TARGET_PROPERTY:prop>` | 当前目标的属性（仅在目标上下文里有效） |
| `$<TARGET_FILE:t>` | 目标 `t` 主输出文件的**完整路径** |
| `$<TARGET_FILE_NAME:t>` | 仅文件名 |
| `$<TARGET_FILE_DIR:t>` | 文件所在目录 |
| `$<TARGET_OBJECTS:t>` | 对象库 `t` 产生的 `.obj`/`.o` 列表 |
| `$<TARGET_SONAME_FILE:t>` | 共享库的 SONAME 文件路径 |

这些在自定义命令、安装规则、生成器表达式的 include 目录里极其常用。

### 3.3 配置相关

| 表达式 | 含义 |
|--------|------|
| `$<CONFIG>` | 当前配置名（`Debug`/`Release`/...） |
| `$<CONFIG:cfg>` | 当前是否为 `cfg` 配置 |

注意：单配置生成器（Ninja/Make）下配置在配置阶段就由 `CMAKE_BUILD_TYPE` 确定，但表达式仍统一可用，写一份表达式兼容两种生成器。

### 3.4 平台与编译器

| 表达式 | 含义 |
|--------|------|
| `$<PLATFORM_ID:ids>` | 当前平台 ID 是否属于 `ids`（如 `Windows,Linux,Darwin`） |
| `$<C_COMPILER_ID:ids>` / `$<CXX_COMPILER_ID:ids>` | C/C++ 编译器 ID（`GNU`/`Clang`/`MSVC`/`AppleClang`...） |
| `$<COMPILE_LANG:langs>` | 当前正在编译的语言（`C`/`CXX`/`CUDA`...） |
| `$<COMPILE_LANG_AND_ID:lang,ids>` | 语言 + 编译器组合判断（最精确） |
| `$<COMPILE_FEATURES:feature>` | 是否支持某特性（如 `cxx_std_17`） |

`$<COMPILE_LANG_AND_ID>` 是跨编译器加选项的推荐写法，因为它按"每个翻译单元的实际编译器与语言"求值，对混合语言目标尤其安全。

### 3.5 输入/输出接口（安装与导出专用）

| 表达式 | 含义 |
|--------|------|
| `$<BUILD_INTERFACE:x>` | 仅在构建树（本工程内使用）时为 `x`，安装导出后为空 |
| `$<INSTALL_INTERFACE:x>` | 仅在安装导出后被下游消费时为 `x`，构建树为空 |

这一对在笔记 [02 目标与属性模型](02-Targets-and-Properties.md) 与 [04 安装导出与打包](04-Install-Export-Package.md) 中详述，核心作用是让同一个目标的 `INTERFACE_INCLUDE_DIRECTORIES` 在"本工程构建"和"安装后被下游 find_package"两种场景下指向不同路径。

## 4. 典型用法

### 4.1 按配置加编译选项

```cmake
add_library(mylib src.cpp)

# Debug 关闭优化并开调试符号；Release 开 -O3
target_compile_options(mylib PRIVATE
    $<$<CONFIG:Debug>:-O0 -g>
    $<$<CONFIG:Release>:-O3>
)
```

多配置生成器（VS）下，`Debug` 和 `Release` 的 `.vcxproj` 会各自得到正确选项，互不影响。

### 4.2 按编译器加警告（跨平台安全写法）

```cmake
target_compile_options(mylib PRIVATE
    # GCC/Clang：开较多警告
    $<$<COMPILE_LANG_AND_ID:CXX,GNU,Clang,AppleClang>:-Wall -Wextra -Wpedantic>
    # MSVC：用 /W4 并关掉一些吵闹的 C 语言警告
    $<$<COMPILE_LANG_AND_ID:CXX,MSVC>:/W4 /wd4464>
)
```

注意用 `COMPILE_LANG_AND_ID` 而非仅 `CXX_COMPILER_ID`，可保证表达式只作用于真正用该编译器编译的翻译单元。

### 4.3 调试符号按平台放不同位置

```cmake
# Windows 下把 PDB 指到输出目录；Linux/macOS 不适用
target_compile_options(mylib PRIVATE
    $<$<AND:$<CXX_COMPILER_ID:MSVC>,$<CONFIG:Debug>>:/Zi /Fd${CMAKE_CURRENT_BINARY_DIR}/>
)
```

### 4.4 在自定义命令里引用目标输出

```cmake
add_custom_command(TARGET mylib POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy
            "$<TARGET_FILE:mylib>"           # 生成阶段才确定的完整路径
            "${CMAKE_BINARY_DIR}/bin/"
    COMMENT "Copy mylib to bin directory")
```

`$<TARGET_FILE:mylib>` 是连接"目标"与"自定义命令/安装规则"的关键——配置阶段你拿不到精确文件名（多配置生成器下文件名可能含配置名）。

### 4.5 区分构建树与安装树的头文件路径

```cmake
target_include_directories(mylib PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>   # 相对于安装前缀，下游安装后解析
)
```

构建时用源码 `include/`，安装后下游用 `${CMAKE_INSTALL_PREFIX}/include`。详见笔记 04。

## 5. 常见陷阱

### 5.1 不能用于 `if()` 的条件

`if()` 在配置阶段执行，那时生成器表达式尚未求值，`if($<CONFIG:Debug>)` 永远不会被求值成你期望的布尔。需要配置阶段判断配置，用 `CMAKE_BUILD_TYPE`（仅单配置生成器有意义）；需要生成阶段判断配置，用生成器表达式但不能放在 `if()` 里，应放在 `target_compile_options` 等接受生成器表达式的命令里。

### 5.2 假条件产生空字符串，可能留下空项

`$<$<CONFIG:Debug>:-O0>` 在 `Release` 下展开为空字符串。多数命令能容忍空项，但若拼接到字符串中或用于需要确切个数参数的场合，可能出现多余空格或空参数。需要时用 `$<IF:cond,a,b>` 显式给出"假"分支，或把多个互斥选项合并到一个 `$<IF:...>` 里。

### 5.3 分号是列表分隔符

CMake 中 `;` 是列表分隔符。生成器表达式求值结果若含 `;`，会被当作列表展开。在 `COMMAND` 参数、文件路径中尤其要小心。需要字面分号时几乎一定是你用错了结构——通常应拆成多个表达式或多个参数。

### 5.4 `CONFIG` 大小写

配置名是字符串，`$<CONFIG:Debug>` 与 `$<CONFIG:debug>` 不等同。多配置生成器的配置名由 `CMAKE_CONFIGURATION_TYPES` 决定，默认首字母大写（`Debug`/`Release`/`RelWithDebInfo`/`MinSizeRel`）。不要假设小写。

### 5.5 生成器表达式不是万能的求值器

它只能求值**生成阶段已知**的值。你不能用它读取"任意变量在生成阶段的值"——普通变量在配置阶段就已定型。生成器表达式能读的是目标属性、配置、平台、编译器等生成阶段信息。

## 6. 实战：一个跨平台库的选项配置

把上面要点串起来，写一个尽可能健壮的库目标：

```cmake
cmake_minimum_required(VERSION 3.20)
project(libdemo LANGUAGES CXX)

add_library(libdemo src/demo.cpp)

# 头文件路径：构建树用源码目录，安装后用相对 include
target_include_directories(libdemo PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

# 跨编译器警告，按语言+编译器精确匹配
target_compile_options(libdemo PRIVATE
    $<$<COMPILE_LANG_AND_ID:CXX,GNU,Clang,AppleClang>:-Wall -Wextra -Wpedantic>
    $<$<COMPILE_LANG_AND_ID:CXX,MSVC>:/W4 /permissive->
)

# 按配置调整优化与调试信息
target_compile_options(libdemo PRIVATE
    $<$<CONFIG:Debug>:-O0 -g>
    $<$<CONFIG:Release>:-O3>
    $<$<AND:$<CXX_COMPILER_ID:MSVC>,$<CONFIG:Debug>>:/Zi>
)

# 仅 C++17 起才需要的特性依赖（用特性而非硬编码 std=）
target_compile_features(libdemo PUBLIC cxx_std_17)

# 构建后把产物拷到统一 bin 目录（多配置也能正确取路径）
add_custom_command(TARGET libdemo POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:libdemo> ${CMAKE_BINARY_DIR}/staging/
    COMMENT "Stage libdemo artifact")
```

要点回顾：

- 警告用 `COMPILE_LANG_AND_ID` 而非 `CXX_COMPILER_ID`，保证混合语言目标安全。
- include 目录用 `BUILD_INTERFACE`/`INSTALL_INTERFACE` 双轨，为后续安装导出做准备。
- 语言标准用 `target_compile_features(... cxx_std_17)` 而非手写 `-std=c++17`，让 CMake 按编译器选正确标志，且传递给下游。
- 复制产物用 `$<TARGET_FILE:...>`，多配置生成器下也能取到正确路径。

## 7. 小结

- **核心动机**：生成器表达式解决"配置阶段未知、生成阶段才确定"的值的求值，是多配置生成器与跨平台/跨编译器场景的必备工具。
- **最高频三类**：目标路径（`TARGET_FILE` 等）、配置判断（`CONFIG`）、编译器/语言判断（`COMPILE_LANG_AND_ID`）。
- **记住边界**：不能进 `if()`；分号是列表分隔符；配置名大小写敏感；假条件产空串。
- **配合现代写法**：与 `target_*` 命令、`BUILD_INTERFACE`/`INSTALL_INTERFACE`、`compile_features` 结合，是现代 CMake 表达力的关键来源。

下一篇：[02 目标与属性模型](02-Targets-and-Properties.md)。
