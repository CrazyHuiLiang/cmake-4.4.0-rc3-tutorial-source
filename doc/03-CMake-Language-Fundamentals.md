# 第 2 步：CMake 语言基础

在上一步中，我们匆忙地一带而过，对 `CMakeLists.txt` 中使用的 CMake 语言的若干方面只是含糊带过，目的是尽快构建出可用的程序。然而在实际场景中，我们会遇到比简单描述源文件与头文件列表复杂得多的情况。

为了应对这种复杂性，CMake 提供了一种图灵完备的领域特定语言，用于描述构建软件的过程。随着我们编写更复杂的 CML 与其他 CMake 文件，理解这门语言的基础将变得不可或缺。该语言的正式名称为 [cmake-language(7)](../../manual/cmake-language.7.html#manual:cmake-language\(7\))，通俗地也称为 CMakeLang。

> **注：** CMake 语言并不适合描述与构建软件无关的事物。尽管它具备一些通用特性，但当开发者用它解决与构建无直接关系的问题时，应当谨慎行事。
>
> 通常正确的做法是：用一门通用编程语言编写一个解决问题的工具，然后教会 CMake 如何在构建过程中调用该工具。人们曾用 CMake 语言编写过代码生成器、加密签名工具，甚至光线追踪器，但这并非推荐的做法。

由于我们希望充分探索这门语言的特性，本步是教程顺序中的一个例外。它既不建立在 `Step1` 之上，也不是 `Step3` 的起点。这里将作为一个沙盒，在不构建任何软件的前提下探索语言特性。我们将在 `Step3` 中重新回到 Tutorial 程序。

> **注：** 本教程致力于展示最佳实践与真实问题的解决方案。然而仅就这一步而言，我们将重新实现一些 CMake 内置函数。在"现实生活"中，请不要自行编写 [list(APPEND)](../../command/list.html#append)。

## 背景

CMakeLang 中仅有的基本类型是字符串与列表。CMake 中的每个对象都是字符串，而列表本身也是字符串，其中以分号作为分隔符。任何看似操作非字符串对象（无论是布尔值、数值、JSON 对象还是其他）的命令，实际上都是在消费一个字符串，执行某些内部转换逻辑（用 CMakeLang 之外的语言完成），然后将任何可能的输出再转换回字符串。

我们可以使用 [set()](../../command/set.html#command:set) 命令创建一个变量，即给一个字符串起一个名字。

```cmake
set(var "World!")
```

变量的值可以通过花括号展开来访问，例如我们想用 [message()](../../command/message.html#command:message) 命令打印出 `var` 所命名的字符串时。

```cmake
set(var "World!")
message("Hello ${var}")
```

```console
$ cmake -P CMakeLists.txt
Hello World!
```

> **注：** [cmake -P](../../manual/cmake.1.html#cmdoption-cmake-P) 被称为"脚本模式"，它会告知 CMake 该文件并不打算包含 [project()](../../command/project.html#command:project) 命令。我们不构建任何软件，而是仅把 CMake 当作命令解释器来使用。

由于 CMakeLang 只有字符串，条件判断完全依据哪些字符串被视为真、哪些被视为假的约定。这些约定*理应*是直观的："True"、"On"、"Yes" 以及（表示）非零数值（的字符串）为真；而 "False"、"Off"、"No"、"0"、"Ignore"、"NotFound" 以及空字符串都被视为假。

不过，其中一些规则要更为复杂，因此花些时间查阅 [if()](../../command/if.html#command:if) 文档中关于表达式的说明是值得的。建议在特定上下文中坚持使用一对一致的取值，例如 "True"/"False" 或 "On"/"Off"。

如前所述，列表是包含分号的字符串。[list()](../../command/list.html#command:list) 命令可用于操作这些列表，CMake 中的许多结构都期望遵循这一约定来运作。例如，我们可以使用 [foreach()](../../command/foreach.html#command:foreach) 命令遍历一个列表。

```cmake
set(stooges "Moe;Larry")
list(APPEND stooges "Curly")

message("Stooges contains: ${stooges}")

foreach(stooge IN LISTS stooges)
  message("Hello, ${stooge}")
endforeach()
```

```console
$ cmake -P CMakeLists.txt
Stooges contains: Moe;Larry;Curly
Hello, Moe
Hello, Larry
Hello, Curly
```

## 练习 1 - 宏、函数与列表

CMake 允许我们编写自己的函数与宏。在构建大量相似目标（例如测试）时，这会非常有用，因为我们会想要一遍又一遍地调用相似的命令集合。我们通过 [function()](../../command/function.html#command:function) 与 [macro()](../../command/macro.html#command:macro) 来实现。

```cmake
macro(MyMacro MacroArgument)
  message("${MacroArgument}\n\t\tFrom Macro")
endmacro()

function(MyFunc FuncArgument)
  MyMacro("${FuncArgument}\n\tFrom Function")
endfunction()

MyFunc("From TopLevel")
```

```console
$ cmake -P CMakeLists.txt
From TopLevel
      From Function
              From Macro
```

与许多语言一样，函数与宏的区别在于作用域。在 CMakeLang 中，[function()](../../command/function.html#command:function) 与 [macro()](../../command/macro.html#command:macro) 都能"看到"在它们之上所有栈帧中创建的变量。然而，[macro()](../../command/macro.html#command:macro) 在语义上类似于文本替换，与 C/C++ 宏相似，因此宏所产生的任何副作用在其调用上下文中都是可见的。如果我们在宏中创建或修改变量，调用者将会看到这一修改。

[function()](../../command/function.html#command:function) 会创建自己的变量作用域，因此副作用对调用者不可见。为了将修改传递回调用该函数的父级，我们必须使用 `set(<var> <value> PARENT_SCOPE)`，其作用与 [set()](../../command/set.html#command:set) 相同，但针对的是属于调用者上下文的变量。

> **注：** 在 CMake 3.25 中新增了 [return(PROPAGATE)](../../command/return.html#command:return) 选项，其作用与 [set(PARENT_SCOPE)](../../command/set.html#command:set) 相同，但提供了稍好一些的人体工学体验。

虽然本练习并不需要，但值得一提的是，[macro()](../../command/macro.html#command:macro) 与 [function()](../../command/function.html#command:function) 都通过 `ARGV` 变量支持可变参数——这是一个包含传递给命令的所有参数的列表，以及 `ARGN` 变量——其中包含最后一个预期参数之后的所有参数。

本练习不会构建任何目标，因此我们将构造自己的 [list(APPEND)](../../command/list.html#append) 版本，用于向列表中添加一个值。

### 目标

实现一个宏和一个函数，用于在不使用 [list(APPEND)](../../command/list.html#append) 命令的前提下向列表追加一个值。

这些命令的期望用法如下：

```cmake
set(Letters "Alpha;Beta")
MacroAppend(Letters "Gamma")
message("Letters contains: ${Letters}")
```

```console
$ cmake -P Exercise1.cmake
Letters contains: Alpha;Beta;Gamma
```

> **注：** 这些练习的扩展名是 `.cmake`，这是不在 `CMakeLists.txt` 中的 CMakeLang 文件所使用的标准扩展名。

### 有用资源

- [macro()](../../command/macro.html#command:macro)
- [function()](../../command/function.html#command:function)
- [set()](../../command/set.html#command:set)
- [if()](../../command/if.html#command:if)

### 待编辑文件

- `Exercise1.cmake`

### 入门

`Exercise1.cmake` 的源代码提供在 `Help/guide/tutorial/Step2` 目录中。其中包含用于验证上述追加行为的测试。

> **注：** 不要求你处理向空列表或未定义列表追加的情形。不过作为附加挑战，该情形已被测试覆盖，如果你想检验自己对 CMakeLang 条件判断的理解，可以一试。

完成 `TODO 1` 与 `TODO 2`。

### 构建与运行

我们将使用脚本模式来运行这些练习。首先进入 `Help/guide/tutorial/Step2` 文件夹，然后可以用以下命令运行代码：

```console
cmake -P Exercise1.cmake
```

脚本会报告这些命令是否被正确实现。

### 解答

本题依赖于对 CMake 变量机制的理解。CMake 变量是字符串的名字；换言之，一个 CMake 变量本身就是一个字符串，它能够通过花括号展开成另一个不同的字符串。

这引出了 CMake 代码中一种常见模式：传给函数与宏的往往不是值本身，而是包含这些值的变量名。因此，`ListVar` 并不包含我们需要追加的列表的*值*，它包含的是一个列表的*名字*，而该列表才包含我们需要追加的值。

当用 `${ListVar}` 展开该变量时，我们将得到列表的名字。如果我们再用 `${${ListVar}}` 展开那个名字，就会得到该列表所包含的值。

要实现 `MacroAppend`，我们只需将对 `ListVar` 的这种理解与对 [set()](../../command/set.html#command:set) 命令的认知结合起来即可。

**TODO 1：点击显示/隐藏答案** — `Exercise1.cmake`

```cmake
macro(MacroAppend ListVar Value)
  set(${ListVar} "${${ListVar}};${Value}")
endmacro()
```

我们在这里无需担心作用域问题，因为宏在其父级的同一作用域中运作。

`FuncAppend` 几乎完全相同，事实上它可以用同样的单行代码实现，只需额外加上 `PARENT_SCOPE`，但题目要求我们基于 `MacroAppend` 来实现它。

**TODO 2：点击显示/隐藏答案** — `Exercise1.cmake`

```cmake
function(FuncAppend ListVar Value)
  MacroAppend(${ListVar} ${Value})
  set(${ListVar} "${${ListVar}}" PARENT_SCOPE)
endfunction()
```

`MacroAppend` 会为我们变换 `ListVar`，但它不会把结果传递到父作用域。由于这是一个函数，我们需要自行用 [set(PARENT_SCOPE)](../../command/set.html#command:set) 来完成传递。

## 练习 2 - 条件与循环

任何结构化编程语言中最常见的两个流程控制元素就是条件判断及其近亲循环。CMakeLang 也不例外。如前所述，给定 CMake 字符串的真假性是由 [if()](../../command/if.html#command:if) 命令确立的约定。

当给定一个字符串时，[if()](../../command/if.html#command:if) 会首先检查它是否为前文讨论过的已知常量值之一。如果该字符串不是这些值之一，命令便假定它是一个变量，并检查该变量经花括号展开后的内容以确定条件的结果。

```cmake
if(True)
  message("Constant Value: True")
else()
  message("Constant Value: False")
endif()

if(ConditionalValue)
  message("Undefined Variable: True")
else()
  message("Undefined Variable: False")
endif()

set(ConditionalValue True)

if(ConditionalValue)
  message("Defined Variable: True")
else()
  message("Defined Variable: False")
endif()
```

```console
$ cmake -P ConditionalValue.cmake
Constant Value: True
Undefined Variable: False
Defined Variable: True
```

> **注：** 这是一个讨论 CMake 中引号用法的好时机。CMake 中的所有对象都是字符串，因此双引号 `"` 往往并非必需。CMake 知道该对象是字符串——一切都是字符串。
>
> 然而，在某些上下文中它是必需的。包含空白的字符串需要双引号，否则会被当作列表处理；CMake 会用分号将各元素拼接在一起。反过来也一样，当对列表进行花括号展开时，如果我们想*保留*分号，就必须在引号内进行。否则 CMake 会把列表项展开为以空格分隔的字符串。
>
> 少数命令（例如 [if()](../../command/if.html#command:if)）能识别带引号与不带引号字符串之间的差别。[if()](../../command/if.html#command:if) 仅在字符串不带引号时才会去检查给定字符串是否表示一个变量。

最后，[if()](../../command/if.html#command:if) 提供了几种有用的比较模式，例如用于字符串匹配的 `STREQUAL`、用于检查变量是否存在的 `DEFINED`，以及用于正则表达式检查的 `MATCHES`。它还支持典型的逻辑运算符 `NOT`、`AND` 与 `OR`。

除条件判断之外，CMake 还提供了两种循环结构：[while()](../../command/while.html#command:while) 遵循与 [if()](../../command/if.html#command:if) 相同的规则来检查循环变量；以及更为实用的 [foreach()](../../command/foreach.html#command:foreach)，它遍历字符串列表，并在[背景](#背景)一节中演示过。

在本练习中，我们将使用循环与条件判断来解决一些简单的问题。我们将使用前文提到的来自 [function()](../../command/function.html#command:function) 的 `ARGN` 变量作为待操作的列表。

### 目标

遍历一个列表，返回所有包含字符串 `Foo` 的字符串。

> **注：** 阅读过命令文档的人会知道这正是 [list(FILTER)](../../command/list.html#filter)，请克制住使用它的诱惑。

### 有用资源

- [function()](../../command/function.html#command:function)
- [foreach()](../../command/foreach.html#command:foreach)
- [if()](../../command/if.html#command:if)
- [list()](../../command/list.html#command:list)

### 待编辑文件

- `Exercise2.cmake`

### 入门

`Exercise2.cmake` 的源代码提供在 `Help/guide/tutorial/Step2` 目录中。其中包含用于验证上述追加行为的测试。

> **注：** 这次你应该使用 [list(APPEND)](../../command/list.html#append) 命令将最终结果收集到一个列表中。输入可以从所提供函数的 `ARGN` 变量中获取。

完成 `TODO 3`。

### 构建与运行

进入 `Help/guide/tutorial/Step2` 文件夹，然后可以用以下命令运行代码：

```console
cmake -P Exercise2.cmake
```

脚本会报告 `FilterFoo` 函数是否被正确实现。

### 解答

我们需要做三件事：遍历 `ARGN` 列表，检查该列表中给定项是否匹配 `"Foo"`，若是则将其追加到 `OutVar` 列表。

虽然调用 [foreach()](../../command/foreach.html#command:foreach) 的方式有几种，但推荐的方式是通过 `IN LISTS` 让命令为我们完成变量展开，从而访问 `ARGN` 列表项。

我们所需的 [if()](../../command/if.html#command:if) 比较是 `MATCHES`，它会检查 `"Foo"` 是否存在于该项中。剩下的就是把该项追加到 `OutVar` 列表。最棘手的部分是要记住 `OutVar` *命名*了一个列表，它本身并不是那个列表，因此我们需要通过 `${OutVar}` 来访问它。

**TODO 3：点击显示/隐藏答案** — `Exercise2.cmake`

```cmake
function(FilterFoo OutVar)

  foreach(item IN LISTS ARGN)
    if(item MATCHES Foo)
      list(APPEND ${OutVar} ${item})
    endif()
  endforeach()

  set(${OutVar} ${${OutVar}} PARENT_SCOPE)
endfunction()
```

## 练习 3 - 使用 include 组织代码

我们已经讨论过如何用 [add_subdirectory()](../../command/add_subdirectory.html#command:add_subdirectory) 引入包含各自 CML 的子目录。在后续步骤中，我们将探索 CMake 代码被打包并在项目间共享的各种方式。

然而对于小型的 CMake 函数与工具，让它们待在项目 CML 之外、与构建系统其余部分相分离的独立 `.cmake` 文件中，往往是有益的。这能够实现关注点分离，把我们用来描述项目的工具与项目中特定的元素剥离开来。

要把这些独立的 `.cmake` 文件引入我们的项目，我们使用 [include()](../../command/include.html#command:include) 命令。该命令会立即在父 CML 的作用域中开始解释被 [include()](../../command/include.html#command:include) 文件的内容。这就像是把整个文件当作一个宏来调用一样。

按照惯例，这类 `.cmake` 文件存放在项目根目录下一个名为 "cmake" 的文件夹中。在本练习中，我们将改用 `Step2` 文件夹。

### 目标

使用练习 1 与练习 2 中的函数来构建并过滤我们自己的条目列表。

### 有用资源

- [include()](../../command/include.html#command:include)

### 待编辑文件

- `Exercise3.cmake`

### 入门

`Exercise3.cmake` 的源代码提供在 `Help/guide/tutorial/Step2` 目录中。其中包含用于验证前两个练习中函数正确用法的测试。

> **注：** 实际上它复用了 Exercise2.cmake 的测试，可复用的代码对大家都有好处。

完成 `TODO 4` 到 `TODO 7`。

### 构建与运行

进入 `Help/guide/tutorial/Step2` 文件夹，然后可以用以下命令运行代码：

```console
cmake -P Exercise3.cmake
```

脚本会报告这些函数是否被正确地调用与组合。

### 解答

[include()](../../command/include.html#command:include) 命令会完整地解释被包含的文件，包括前两个练习中的测试。我们并不想再次运行这些测试。多亏事先有所考虑，这些文件在运行测试前会检查一个名为 `SKIP_TESTS` 的变量，将其设为 `True` 即可得到我们想要的行为。

**TODO 4：点击显示/隐藏答案** — `Exercise3.cmake`

```cmake
set(SKIP_TESTS True)
```

现在我们可以 [include()](../../command/include.html#command:include) 之前的练习以获取它们的函数了。

**TODO 5：点击显示/隐藏答案** — `Exercise3.cmake`

```cmake
include(Exercise1.cmake)
include(Exercise2.cmake)
```

既然 `FuncAppend` 已可供使用，我们就可以用它向 `InList` 追加新元素。

**TODO 6：点击显示/隐藏答案** — `Exercise3.cmake`

```cmake
FuncAppend(InList FooBaz)
FuncAppend(InList QuxBaz)
```

最后，我们可以用 `FilterFoo` 来过滤完整列表。这里需要记住的棘手之处在于，我们的 `FilterFoo` 想要通过 `ARGN` 来操作列表的值，因此在调用 `FilterFoo` 时我们需要展开 `InList`。

**TODO 7：点击显示/隐藏答案** — `Exercise3.cmake`

```cmake
FilterFoo(OutList ${InList})
```
