# Step 8: Testing and CTest

Testing is, historically, not the role of the build system. At best it might
have a specific target which maps to building and running the project's tests.

In the CMake ecosystem, the opposite is true. CMake's testing ecosystem is
known as CTest. This ecosystem is both deceivingly simple and incredibly
powerful. In fact it is so powerful it deserves its own full tutorial to
describe everything we could achieve with it.

This is not that tutorial. In this step, we will scratch the surface of some
of the facilities that CTest provides.

## Background

At its core, CTest is a task launcher which runs commands and reports if they
have returned zero or non-zero values. This is the level we will be dealing
with CTest at.

CMake provides direct integration with CTest via the [`enable_testing()`](../../command/enable_testing.html#command:enable_testing)
and [`add_test()`](../../command/add_test.html#command:add_test) commands. These allow CMake to setup the necessary
infrastructure in the build folder for CTest to discover, run, and report
on various tests we might be interested in.

After setting up and building tests, the easiest way to invoke CTest is to run
it directly on the build directory with:

```console
ctest --test-dir build
```

Which will run all available tests. Specific tests can be run with regular
expressions.

```console
ctest --test-dir build -R SpecificTest
```

CTest also has advanced mechanisms for scripting, fixtures, sanitizers,
job servers, metric reportings, and much more. See the [`ctest(1)`](../../manual/ctest.1.html#manual:ctest(1))
manual for more information.

## Exercise 1 - Adding Tests

CTest convention dictates the building and running of tests be based on a
default-`ON` variable named [`BUILD_TESTING`](../../variable/BUILD_TESTING.html#variable:BUILD_TESTING). When using the full
suite of CTest capabilities via the [`CTest`](../../module/CTest.html#module:CTest) module, this
[`option()`](../../command/option.html#command:option) is setup for us. When using a more stripped-down approach to
testing, it's expected the project will setup the option (or at least one of a
similar name) on its own.

When [`BUILD_TESTING`](../../variable/BUILD_TESTING.html#variable:BUILD_TESTING) is true, the [`enable_testing()`](../../command/enable_testing.html#command:enable_testing) command
should be called in the root CML.

```cmake
enable_testing()
```

This will generate all the necessary metadata into the build tree for CTest to
find and run tests.

Once that has been done, the [`add_test()`](../../command/add_test.html#command:add_test) command can be used to create
a test anywhere in the project. The semantics of this command are similar to
[`add_custom_command()`](../../command/add_custom_command.html#command:add_custom_command); we can name an executable target as the "command".

```cmake
add_test(
  NAME MyAppWithTestFlag
  COMMAND MyApp --test
)
```

### Goal

Add tests for the MathFunctions library to the project and run them with CTest.

### Helpful Resources

- [`BUILD_TESTING`](../../variable/BUILD_TESTING.html#variable:BUILD_TESTING)
- [`enable_testing()`](../../command/enable_testing.html#command:enable_testing)
- [`function()`](../../command/function.html#command:function)
- [`add_test()`](../../command/add_test.html#command:add_test)

### Files to Edit

- `Tests/CMakeLists.txt`
- `CMakeLists.txt`

### Getting Started

A testing program has been written in the file `Tests/TestMathFunctions.cxx`.
This program takes a single command line argument, the math function to be
tested, with valid values of `add`, `mul`, `sqrt`, and `sub`. The return
code is zero if the operation is recognized and the calculated value is valid,
otherwise it is non-zero.

Complete `TODO 1` through `TODO 7`.

### Build and Run

No special configuration is needed, configure and build as usual.

```console
cmake --preset tutorial
cmake --build build
```

Verify all the tests pass with CTest.

> **Note:** When using a multi-config generator such as Visual Studio, it
will be necessary to specify a configuration like `Debug` or `Release`
using [`ctest -C`](../../manual/ctest.1.html#cmdoption-ctest-C). This is true whenever using a multi-config
generator, and won't be called out specifically in future commands.

```console
ctest --test-dir build
```

You can run individual tests with the [`-R`](../../manual/ctest.1.html#cmdoption-ctest-R) flag.

```console
ctest --test-dir build -R sqrt
```

### Solution

First we add a new executable for the tests.

**TODO 1-2: Tests/CMakeLists.txt**

```cmake
add_executable(TestMathFunctions)

target_sources(TestMathFunctions
  PRIVATE
    TestMathFunctions.cxx
)
```

Then we link in the library we are testing.

**TODO 3: Tests/CMakeLists.txt**

```cmake
target_link_libraries(TestMathFunctions
  PRIVATE
    MathFunctions
)
```

We need to call [`add_test()`](../../command/add_test.html#command:add_test) for each of the valid operations, but this
would get repetitive, so we write a [`function()`](../../command/function.html#command:function) to do it for us.

**TODO 4: Tests/CMakeLists.txt**

```cmake
function(MathFunctionTest op)
  add_test(
    NAME ${op}
    COMMAND TestMathFunctions ${op}
  )
endfunction()
```

Now we can use our [`function()`](../../command/function.html#command:function) to add all the tests.

**TODO 5: Tests/CMakeLists.txt**

```cmake
MathFunctionTest(add)
MathFunctionTest(mul)
MathFunctionTest(sqrt)
MathFunctionTest(sub)
```

Finally, we can add the [`BUILD_TESTING`](../../variable/BUILD_TESTING.html#variable:BUILD_TESTING) option and conditionally
enable building and running tests in the top-level CML.

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
