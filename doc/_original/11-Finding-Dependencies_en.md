# Step 10: Finding Dependencies

In C/C++ software development, managing build dependencies is consistently
one of the highest ranked challenges facing modern developers. CMake provides
an extensive toolset for discovering and validating dependencies of different
kinds.

However, for correctly packaged projects there is no need to use these advanced
tools. Many popular library and utility projects today produce correct install
trees, like the one we set up in `Step 9`, which are easy is to integrate
into CMake.

In this best-case scenario, we only need the `find_package()` to
import dependencies into our project.

## Background

There are five principle commands used for discovering dependencies with
CMake, the first four are:

**`find_file()`**

  Finds and reports the full path to a named file, this tends to be the
  most flexible of the `find` commands.

**`find_library()`**

  Finds and reports the full path to a static archive or shared object
  suitable for use with `target_link_libraries()`.

**`find_path()`**

  Finds and reports the full path to a directory *containing* a file. This
  is most commonly used for headers in combination with
  `target_include_directories()`.

**`find_program()`**

  Finds and reports and invocable name or path for a program. Often used in
  combination with `execute_process()` or `add_custom_command()`.

These commands should be considered "backup", used when the primary find command
is unsuitable. The primary find command is `find_package()`. It uses
comprehensive built-in heuristics and upstream-provided packaging files to
provide the best interface to the requested dependency.

## Exercise 1 - Using `find_package()`

The search paths and behaviors used by `find_package()` are fully
described in its documentation, but much too verbose to replicate here. Suffice
to say it searches well known, lesser known, obscure, and user-provided
locations attempting to find a package which meets the requirements given to it.

```cmake
find_package(ForeignLibrary)
```

The best way to use `find_package()` is to ensure all dependencies have
been installed to a single install tree prior to the build, and then make the
location of that install tree known to `find_package()` via the
`CMAKE_PREFIX_PATH` variable.

> **Note:** Building and installing dependencies can itself be an immense amount of labor.
> While this tutorial will do so for illustration purposes, it is **extremely**
> recommended that a package manager be used for project-local dependency
> management.

`find_package()` accepts several parameters besides the package to be
found. The most notable are:

- A positional `<version>` argument, for describing a version to be checked
against the package's config version file. This should be used sparingly,
it is better to control the version of the dependency being installed via
a package manager than possibly break the build on otherwise innocuous
version updates.
  If the package is known to rely on an older version of a dependency, it
may be appropriate to use a version requirement.

- `REQUIRED` for non-optional dependencies which should abort the build
if not found.

- `QUIET` for optional dependencies which should not report anything to
users when not found.

`find_package()` reports its results via `<PackageName>_FOUND`
variables, which will be set to a true or false value for found and not found
packages respectively.

### Goal

Integrate an externally installed test framework into the Tutorial project.

### Helpful Resources

- `find_package()`

- `target_link_libraries()`

### Files to Edit

- `TutorialProject/CMakePresets.json`

- `TutorialProject/Tests/CMakeLists.txt`

- `TutorialProject/Tests/TestMathFunctions.cxx`

### Getting Started

The `Step10` folder is organized differently than previous steps. The tutorial
project we need to edit is under `Step10/TutorialProject`. Another project
is now present, `SimpleTest`, as well as a partially populated install tree
which we will use in later exercises. You do not need to edit anything in these
other directories for this exercise, all `TODOs` and solution steps are for
`TutorialProject`.

The `SimpleTest` package provides two useful constructs, the
`SimpleTest::SimpleTest` target to be linked into a test binary, and the
`simpletest_discover_tests` function for automatically adding tests to
CTest.

Similar to other test frameworks, `simpletest_discover_tests` only needs
to be passed the name of the executable target containing the tests.

```cmake
simpletest_discover_tests(MyTestExe)
```

The `TestMathFunctions.cxx` file has been updated to use the `SimpleTest`
framework in the vein of GoogleTest or Catch2. Perform `TODO 1` through
`TODO 5` in order to use the new test framework.

> **Note:** It may go without saying, but `SimpleTest` is a very poor test framework
> which only facially resembles a functional testing library. While much of
> the CMake code in this tutorial could be used unaltered in other projects,
> you should not use `SimpleTest` outside this tutorial, or try to learn from
> the CMake code it provides.

### Build and Run

First we must install the `SimpleTest` framework. Navigate to the
`Help/guide/Step10/SimpleTest` directory and run the following commands.

```console
cmake --preset tutorial
cmake --install build
```

> **Note:** The `SimpleTest` preset sets up everything needed to install `SimpleTest`
> for the tutorial. For reasons that are beyond the scope of this tutorial,
> there is no need to build or provide any other configuration for
> `SimpleTest`.

We can observe that the `Step10/install` directory has now been populated by
the `SimpleTest` header and package files.

Now we can configure and build the Tutorial project as per usual, navigating to
the `Help/guide/Step10/TutorialProject` and running:

```console
cmake --preset tutorial
cmake --build build
```

Verify that the `SimpleTest` framework has been consumed correctly by running
the tests with CTest.

### Solution

First we call `find_package()` to discover the `SimpleTest` package.
We do this with `REQUIRED` because the tests cannot build without
`SimpleTest`.

**TODO 1: TutorialProject/Tests/CMakeLists.txt**

```cmake
find_package(SimpleTest REQUIRED)
```

Next we add the `SimpleTest::SimpleTest` target to `TestMathFunctions`.

**TODO 2: TutorialProject/Tests/CMakeLists.txt**

```cmake
target_link_libraries(TestMathFunctions
  PRIVATE
    MathFunctions
    SimpleTest::SimpleTest
)
```

Now we can replace our test description code with a call to
`simpletest_discover_tests`.

**TODO 3: TutorialProject/Tests/CMakeLists.txt**

```cmake
simpletest_discover_tests(TestMathFunctions)
```

We ensure `find_package()` can discover `SimpleTest` by
adding the install tree to `CMAKE_PREFIX_PATH`.

**TODO 4: TutorialProject/CMakePresets.json**

```json
"cacheVariables": {
  "CMAKE_PREFIX_PATH": "${sourceParentDir}/install",
  "TUTORIAL_USE_STD_SQRT": "OFF",
  "TUTORIAL_ENABLE_IPO": "OFF"
}
```

Finally, we update the tests to use the macros provided by `SimpleTest` by
removing the placeholders and including the appropriate header.

**TODO 5: TutorialProject/Tests/TestMathFunctions.cxx**

```c++
#include <MathFunctions.h>
#include <SimpleTest.h>

TEST("add")
{
```

## Exercise 2 - Transitive Dependencies

Libraries often build on one another. A multimedia application may depend on a
library which provides support for various container formats, which may in turn
rely on one or more other libraries for compression algorithms.

We need to express these transitive requirements inside the package config
files we place in the install tree. We do so with the
`CMakeFindDependencyMacro` module, which provides a safe mechanism for
installed packages to recursively discover one another.

```cmake
include(CMakeFindDependencyMacro)
find_dependency(zlib)
```

`find_dependency()` also forwards arguments
from the top-level `find_package()` call. If `find_package()` is
called with `QUIET` or `REQUIRED`,
`find_dependency()` will also use `QUIET`
and/or `REQUIRED`.

### Goal

Add a dependency to `SimpleTest` and ensure that packages which rely on
`SimpleTest` also discover this transitive dependency.

### Helpful Resources

- `CMakeFindDependencyMacro`

- `find_package()`

- `target_link_libraries()`

### Files to Edit

- `SimpleTest/CMakeLists.txt`

- `SimpleTest/cmake/SimpleTestConfig.cmake`

### Getting Started

For this step we will only be editing the `SimpleTest` project. The transitive
dependency, `TransitiveDep`, is a dummy dependency which provides no behavior.
However CMake doesn't know this and the `TutorialProject` tests will fail to
configure and build if CMake cannot find all required dependencies.

The `TransitiveDep` package has already been installed to the
`Step10/install` tree. We do not need to install it as we did with
`SimpleTest`.

Complete `TODO 6` through `TODO 8`.

### Build and Run

We need to reinstall the SimpleTest framework. Navigate to the
`Help/guide/Step10/SimpleTest` directory and run the same commands as before.

```console
cmake --preset tutorial
cmake --install build
```

Now we can reconfigure and rebuild the `TutorialProject`, navigate to
`Help/guide/Step10/TutorialProject` and perform the usual steps to do so.

```console
cmake --preset tutorial
cmake --build build
```

If the build passed we have likely successfully propagated the transitive
dependency. Verify this by searching the `CMakeCache.txt` of
`TutorialProject` for an entry named `TransitiveDep_DIR`. This demonstrates
the `TutorialProject` searched for an found `TransitiveDep` even though it
has no direct requirement for it.

### Solution

First we call `find_package()` to discover the `TransitiveDep` package.
We use `REQUIRED` to verify we have found `TransitiveDep`.

**TODO 6: SimpleTest/CMakeLists.txt**

```cmake
find_package(TransitiveDep REQUIRED)
```

Next we add the `TransitiveDep::TransitiveDep` target to `SimpleTest`.

**TODO 7: SimpleTest/CMakeLists.txt**

```cmake
target_link_libraries(SimpleTest
  INTERFACE
    TransitiveDep::TransitiveDep
)
```

> **Note:** If we built `TutorialProject` at this point, we would expect the
> configuration to fail due to the `TransitiveDep::TransitiveDep` target
> being unavailable inside that project.

Finally, we include the `CMakeFindDependencyMacro` and call
`find_dependency()` inside the `SimpleTest`
package config file to propagate the transitive dependency.

**TODO 8: SimpleTest/cmake/SimpleTestConfig.cmake**

```cmake
include(CMakeFindDependencyMacro)
find_dependency(TransitiveDep)
```

## Exercise 3 - Finding Other Kinds of Files

In a perfect world every dependency we care about would be packaged correctly,
or at least some other developer would have written a module that discovers it
for us. We do not live in a perfect world, and sometimes we will have to get
our hands dirty and discover build requirements manually.

For this we have the other find commands enumerated earlier in the step, such
as `find_path()`.

```cmake
find_path(PackageIncludeFolder Package.h REQUIRED
  PATH_SUFFIXES
    Package
)
target_include_directories(MyApp
  PRIVATE
    ${PackageIncludeFolder}
)
```

### Goal

Add an unpackaged header to the `Tutorial` executable of the
`TutorialProject`.

### Helpful Resources

- `find_path()`

- `target_include_directories()`

### Files to Edit

- `TutorialProject/Tutorial/CMakeLists.txt`

- `TutorialProject/Tutorial/Tutorial.cxx`

### Getting Started

For this step we will only be editing the `TutorialProject` project. The
unpackaged header, `Unpackaged/Unpackaged.h` has already been installed to the
`Step10/install` tree.

Complete `TODO 9` through `TODO 11`.

### Build and Run

There are no special build steps for this exercise, navigate to
`Help/guide/Step10/TutorialProject` and perform the usual build.

```console
cmake --build build
```

If the build passed we have successfully added the `Unpackaged` include
directory to the project.

### Solution

First we call `find_path()` to discover the `Unpackaged` include
directory. We use `REQUIRED` because building `Tutorial` will fail if
we cannot locate the `Unpackaged.h` header.

**TODO 9: TutorialProject/Tutorial/CMakeLists.txt**

```cmake
find_path(UnpackagedIncludeFolder Unpackaged.h REQUIRED
  PATH_SUFFIXES
    Unpackaged
)
```

Next we add the discovered path to `Tutorial` using
`target_include_directories()`.

**TODO 10: TutorialProject/Tutorial/CMakeLists.txt**

```cmake
target_include_directories(Tutorial
  PRIVATE
    ${UnpackagedIncludeFolder}
)
```

Finally, we edit `Tutorial.cxx` to include the discovered header.

**TODO 11: TutorialProject/Tutorial/Tutorial.cxx**

```c++
#include <MathFunctions.h>
#include <Unpackaged.h>
```
