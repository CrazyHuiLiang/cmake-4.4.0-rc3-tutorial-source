# Step 6: In-Depth System Introspection

In order to discover information about the system environment and the toolchain,
CMake will often compile small test programs to verify the availability of
compiler flags, headers, and builtins or other language constructs.

In this step, we will take advantage of the same test program mechanisms that
CMake uses in our own project code.

## Background

An old trick going back to the oldest days of configuration and build systems
is to verify the availability of some feature by compiling a small program
which uses that feature.

CMake makes this unnecessary for many contexts. As we will address in later
steps, if CMake can find a library dependency, we can rely on it having all
the facilities (headers, code generators, test utilities, etc) we expect it to
have. Conversely, if CMake can't find a dependency, attempting to use the
dependency anyway will almost certainly fail.

However, there are other kinds of information about the toolchain which CMake
doesn't communicate readily. For these advanced cases, we can write our own
test programs and compile commands to check for availability.

CMake provides modules to simplify these checks. These are documented at
`cmake-modules(7)`. Any module that begins with `Check` is a system
introspection module we can use to interrogate the toolchain and system
environment. Some notable ones include:

- `CheckIncludeFiles`: Check one or more C/C++ header files.
- `CheckCompilerFlag`: Check whether the compiler supports a given flag.
- `CheckSourceCompiles`: Checks whether source code can be built for a given language.
- `CheckIPOSupported`: Check whether the compiler supports interprocedural optimization (IPO/LTO).

## Exercise 1 - Check Include File

A fast and easy check to perform is if a given header file is available on
a certain platform, for which CMake provides `CheckIncludeFiles`. This
is most appropriate for system and intrinsic headers, which may not be provided
by a specific package yet are expected to be available in many build environments.

```cmake
include(CheckIncludeFiles)
check_include_files(sys/socket.h HAVE_SYS_SOCKET_H LANGUAGE CXX)
```

> **Note:** These functions are not immediately available in CMake, they must be added via
`include()`'ing their associated module (aka, a CMakeLang file). Many
modules live inside CMake's own `Modules` folder. This built-in `Modules`
folder is one of the places CMake searches when evaluating an `include()`
command. You can think of these modules like standard library headers, they're
expected to be available.

Once a header file is known to exist, we can communicate that to our code using
the same mechanisms of conditionals and target commands already covered.

### Goal

Check if the x86 SSE2 intrinsic header is available, and if so use it to
improve `mathfunctions::sqrt`.

### Helpful Resources

- `CheckIncludeFiles`
- `target_compile_definitions()`

### Files to Edit

- `MathFunctions/CMakeLists.txt`
- `MathFunctions/MathFunctions.cxx`

### Getting Started

The `Help/guide/tutorial/Step6` directory contains the complete, recommended
solution to `Step5` and relevant `TODOs` for this step. It also contains
specialized implementations of the `sqrt` function for various conditions,
which you will find in `MathFunctions/MathFunctions.cxx`.

Complete `TODO 1` through `TODO 3`. Note that some `#ifdef` directives
have already been added to the library, which will change its operation as we
work through the step.

### Build and Run

We can use our usual commands to configure.

```console
cmake --preset tutorial
cmake --build build
```

In the output of the configuration step we should observe CMake checking for
the `emmintrin.h` header.

```console
-- Looking for include file emmintrin.h
-- Looking for include file emmintrin.h - found
```

If the header is available on your system, verify the `Tutorial` output
contains the message about using SSE2. Conversely, if the header is not
available you should see the usual behavior from `Tutorial`.

### Solution

First we include and use the `CheckIncludeFiles` module, verifying the
`emmintrin.h` header is available.

**TODO 1: MathFunctions/CMakeLists.txt**

```cmake
include(CheckIncludeFiles)
check_include_files(emmintrin.h HAS_EMMINTRIN LANGUAGE CXX)
```

Then we use the result of the check to conditionally set a compile definition
on `MathFunctions`.

**TODO 2: MathFunctions/CMakeLists.txt**

```cmake
if(HAS_EMMINTRIN)
  target_compile_definitions(MathFunctions PRIVATE TUTORIAL_USE_SSE2)
endif()
```

Finally we can conditionally include the header in the `MathFunctions` library.

**TODO 3: MathFunctions/MathFunctions.cxx**

```c++
#ifdef TUTORIAL_USE_SSE2
#  include <emmintrin.h>
#endif
```

## Exercise 2 - Check Source Compiles

Sometimes it is insufficient to merely check for a header. This is especially
true when no header is available to check, such is the case with
compiler-builtins. For these scenarios we have `CheckSourceCompiles`.

```cmake
include(CheckSourceCompiles)
check_source_compiles(CXX
  "
    int main() {
      int a, b, c;
      __builtin_add_overflow(a, b, &c);
    }
  "
  HAS_CHECKED_ADDITION
)
```

> **Note:** By default `CheckSourceCompiles` builds and links an executable. The
code to be check must provide a valid `int main()` in order to succeed.

After performing the check, this system introspection can be applied identically
to how we discussed with header files.

### Goal

Check if the GNU SSE2 builtins are available, and if so use them to improve
`mathfunctions::sqrt`.

### Helpful Resources

- `CheckSourceCompiles`
- `target_compile_definitions()`

### Files to Edit

- `MathFunctions/CMakeLists.txt`

### Getting Started

Complete `TODO 4` and `TODO 5`. No code changes to the `MathFunctions`
implementation are necessary, as these have already been provided.

### Build and Run

We need only rebuild the tutorial.

```console
cmake --build build
```

> **Note:** If a check fails and you think it should succeed, you will need to clear the
CMake Cache by deleting the `CMakeCache.txt` file. CMake will not rerun
compile checks on subsequent runs if it has a cached result.

In the output of the configuration step we should observe CMake checking if the
provided source code compiles, which will be reported under the variable name
we provided to `check_source_compiles()`.

```console
-- Performing Test HAS_GNU_BUILTIN
-- Performing Test HAS_GNU_BUILTIN - Success
```

If the builtins are available on your compiler, verify the `Tutorial` output
contains the message about using GNU-builting. Conversely, if the builtins are
not available you should see the previous behavior from `Tutorial`.

### Solution

First we include and use the `CheckSourceCompiles` module, verifying the
provided source code can be built.

**TODO 4: MathFunctions/CMakeLists.txt**

```text
include(CheckSourceCompiles)
check_source_compiles(CXX
  [=[
    typedef double v2df __attribute__((vector_size(16)));
    int main() {
      __builtin_ia32_sqrtsd(v2df{});
    }
  ]=]
  HAS_GNU_BUILTIN
)
```

Then we use the result of the check to conditionally set a compile definition
on `MathFunctions`.

**TODO 5: MathFunctions/CMakeLists.txt**

```cmake
if(HAS_GNU_BUILTIN)
  target_compile_definitions(MathFunctions PRIVATE TUTORIAL_USE_GNU_BUILTIN)
endif()
```

## Exercise 3 - Check Interprocedural Optimization

Interprocedural and link time optimizations can provide significant performance
improvements to some software. CMake has the capacity to check for the
availability of IPO flags via `CheckIPOSupported`.

```cmake
include(CheckIPOSupported)
check_ipo_supported() # fatal error if IPO is not supported
set_target_properties(MyApp
  PROPERTIES
    INTERPROCEDURAL_OPTIMIZATION TRUE
)
```

> **Note:** There a couple important caveats with regard to in-project IPO configuration:
> - CMake does not know about every IPO/LTO flag on every compiler, better
results can often be achieved with individual tuning for a known toolchain.
> - Setting the `INTERPROCEDURAL_OPTIMIZATION` property on a target
does not alter any of the targets it links to, or dependencies from other
projects. IPO can only "see" into other targets which are also compiled
appropriately.
>
> For these reasons, serious consideration should be given to manually setting
up IPO/LTO flags across all projects in the dependency tree via external
mechanisms (presets, `-D` flags, `toolchain files`, etc) instead of in-project
control.

However, especially for extremely large projects, it can be useful to have
an in-project mechanism to use IPO whenever it is available.

### Goal

Enable IPO for the entire tutorial project when it is available from the
toolchain.

### Helpful Resources

- `CheckIPOSupported`
- `CMAKE_INTERPROCEDURAL_OPTIMIZATION`

### Files to Edit

- `CMakeLists.txt`

### Getting Started

Continue editing the files in `Step6`. Complete `TODO 6` and `TODO 7`.

### Build and Run

We need only rebuild the tutorial.

```console
cmake --build build
```

If IPO is unavailable, we will see an error message during configuration.
Otherwise nothing will change.

> **Note:** Regardless of the result of the IPO check, we shouldn't expect any change
in behavior from `Tutorial` or `MathFunctions`.

### Solution

The first `TODO` is easy, we add another option to our project.

**TODO 6: CMakeLists.txt**

```cmake
option(TUTORIAL_ENABLE_IPO "Check for and use IPO support" ON)
```

The next step is involved, however the documentation for `CheckIPOSupported`
has an almost complete example of what we need to do. The only difference is
we are going to enable IPO project-wide instead of for a single target.

**TODO 7: CMakeLists.txt**

```cmake
if(TUTORIAL_ENABLE_IPO)
  include(CheckIPOSupported)
  check_ipo_supported(RESULT result OUTPUT output)
  if(result)
    message("IPO is supported, enabling IPO")
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION ON)
  else()
    message(WARNING "IPO is not supported: ${output}")
  endif()
endif()
```

> **Note:** Normally we have discouraged setting `CMAKE_` variables inside the project.
Here, we are controlling that behavior with an `option()`. This
allows packagers to opt-out of our override. This is an imperfect, but
acceptable solution to situations where we want to provide options to control
project-wide behavior controlled by `CMAKE_` variables.
