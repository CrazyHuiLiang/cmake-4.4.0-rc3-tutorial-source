# 第 8 步：测试与 CTest

从历史上看，测试并不是构建系统的职责。它充其量只会提供一个特定的目标，用于构建并运行项目的测试。

而在 CMake 生态系统中，情况恰恰相反。CMake 的测试生态系统被称为 CTest。这个生态系统看似简单，实则异常强大。事实上，它强大到足以单独成篇，用一整篇教程来阐述我们能用它实现的一切。

本教程并不承担 that tutorial 的任务。在本步骤中，我们仅浅尝 CTest 所提供的一部分能力。

## 背景

就其核心而言，CTest 是一个任务启动器：它运行命令，并报告这些命令返回的是零值还是非零值。我们将在这个层面使用 CTest。

CMake 通过 `enable_testing()` 和 `add_test()` 命令与 CTest 直接集成。这些命令让 CMake 在构建文件夹中搭建必要的基础设施，以便 CTest 发现、运行并报告我们可能关心的各项测试。

在搭建并构建好测试之后，调用 CTest 最简单的方式是直接在构建目录上运行：

```console
ctest --test-dir build
```

这将运行所有可用的测试。可以通过正则表达式来运行特定的测试。

```console
ctest --test-dir build -R SpecificTest
```

CTest 还具备用于脚本、fixtures、sanitizers、job servers、指标上报等更多高级机制。更多信息请参阅 `ctest(1)` 手册。

## 练习 1 - 添加测试

CTest 的惯例规定，测试的构建与运行应基于一个默认为 `ON` 的变量 `BUILD_TESTING`。当通过 `CTest` 模块使用 CTest 的完整能力套件时，这个 `option()` 会为我们自动设置好。当采用更精简的测试方式时，则期望项目自行设置该选项（或至少是一个名称类似的选项）。

当 `BUILD_TESTING` 为真时，应在根 CML 中调用 `enable_testing()` 命令。

```cmake
enable_testing()
```

这会在构建树中生成所有必要的元数据，供 CTest 查找并运行测试。

完成上述操作后，就可以在项目的任何位置使用 `add_test()` 命令来创建测试。该命令的语义与 `add_custom_command()` 类似；我们可以将一个可执行目标指定为"命令"。

```cmake
add_test(
  NAME MyAppWithTestFlag
  COMMAND MyApp --test
)
```

### 目标

为项目中的 MathFunctions 库添加测试，并使用 CTest 运行它们。

### 参考资源

- `BUILD_TESTING`
- `enable_testing()`
- `function()`
- `add_test()`

### 待编辑文件

- `Tests/CMakeLists.txt`
- `CMakeLists.txt`

### 入门指引

测试程序已写入文件 `Tests/TestMathFunctions.cxx` 中。该程序接受一个命令行参数——要测试的数学函数，有效取值为 `add`、`mul`、`sqrt` 和 `sub`。当操作被识别且计算值有效时返回码为零，否则为非零。

完成 `TODO 1` 到 `TODO 7`。

### 构建与运行

无需特殊配置，按常规方式配置并构建即可。

```console
cmake --preset tutorial
cmake --build build
```

使用 CTest 验证所有测试是否通过。

> **注：** 当使用多配置生成器（如 Visual Studio）时，需要通过 `ctest -C` 指定配置，例如 `Debug` 或 `Release`。只要使用多配置生成器就必须这样做，后续命令中将不再特别说明。

```console
ctest --test-dir build
```

你可以使用 `-R` 标志运行单个测试。

```console
ctest --test-dir build -R sqrt
```

### 解答

首先，我们为测试添加一个新的可执行目标。

**TODO 1-2: Tests/CMakeLists.txt**

```cmake
add_executable(TestMathFunctions)

target_sources(TestMathFunctions
  PRIVATE
    TestMathFunctions.cxx
)
```

然后，我们链接正在测试的库。

**TODO 3: Tests/CMakeLists.txt**

```cmake
target_link_libraries(TestMathFunctions
  PRIVATE
    MathFunctions
)
```

我们需要为每个有效操作调用 `add_test()`，但这会变得重复，因此我们编写一个 `function()` 来代劳。

**TODO 4: Tests/CMakeLists.txt**

```cmake
function(MathFunctionTest op)
  add_test(
    NAME ${op}
    COMMAND TestMathFunctions ${op}
  )
endfunction()
```

现在我们可以使用我们的 `function()` 来添加所有测试。

**TODO 5: Tests/CMakeLists.txt**

```cmake
MathFunctionTest(add)
MathFunctionTest(mul)
MathFunctionTest(sqrt)
MathFunctionTest(sub)
```

最后，我们可以在顶层 CML 中添加 `BUILD_TESTING` 选项，并有条件地启用测试的构建与运行。

**TODO 6: CMakeLists.txt**

```cmake
option(BUILD_TESTING "Enable testing and build tests" ON)
```

**TODO 7: CMakeLists.txt**

```cmake
if(BUILD_TESTING)
  enable_testing()
  add_subdirectory(Tests)
endif()
```
