# CMake Tutorial

## Introduction

The CMake tutorial provides a step-by-step guide that covers common build
system issues that CMake helps address. Seeing how various topics all
work together in an example project can be very helpful.

## Steps

The tutorial source code examples are available in
[`this archive`](../../_downloads/0a258b4d00376f5addcda21ec90aeac3/cmake-4.4.0-rc3-tutorial-source.zip).
Each step has its own subdirectory containing code that may be used as a
starting point. The tutorial examples are progressive so that each step
provides the complete solution for the previous step.

- [Step 0: Before You Begin](01-Before-You-Begin_en.md)
  - [Getting the Tutorial Exercises](01-Before-You-Begin_en.md#getting-the-tutorial-exercises)
  - [Getting CMake](01-Before-You-Begin_en.md#getting-cmake)
  - [CMake Generators](01-Before-You-Begin_en.md#cmake-generators)
  - [Single and Multi-Configuration Generators](01-Before-You-Begin_en.md#single-and-multi-configuration-generators)
  - [Other Usage Basics](01-Before-You-Begin_en.md#other-usage-basics)
  - [Try It Out](01-Before-You-Begin_en.md#try-it-out)
  - [Getting Help and Additional Resources](01-Before-You-Begin_en.md#getting-help-and-additional-resources)
- [Step 1: Getting Started with CMake](02-Getting-Started-with-CMake_en.md)
  - [Background](02-Getting-Started-with-CMake_en.md#background)
  - [Exercise 1 - Building an Executable](02-Getting-Started-with-CMake_en.md#exercise-1-building-an-executable)
  - [Exercise 2 - Building a Library](02-Getting-Started-with-CMake_en.md#exercise-2-building-a-library)
  - [Exercise 3 - Linking Together Libraries and Executables](02-Getting-Started-with-CMake_en.md#exercise-3-linking-together-libraries-and-executables)
  - [Exercise 4 - Subdirectories](02-Getting-Started-with-CMake_en.md#exercise-4-subdirectories)
- [Step 2: CMake Language Fundamentals](03-CMake-Language-Fundamentals_en.md)
  - [Background](03-CMake-Language-Fundamentals_en.md#background)
  - [Exercise 1 - Macros, Functions, and Lists](03-CMake-Language-Fundamentals_en.md#exercise-1-macros-functions-and-lists)
  - [Exercise 2 - Conditionals and Loops](03-CMake-Language-Fundamentals_en.md#exercise-2-conditionals-and-loops)
  - [Exercise 3 - Organizing with Include](03-CMake-Language-Fundamentals_en.md#exercise-3-organizing-with-include)
- [Step 3: Configuration and Cache Variables](04-Configuration-and-Cache-Variables_en.md)
  - [Background](04-Configuration-and-Cache-Variables_en.md#background)
  - [Exercise 1 - Using Options](04-Configuration-and-Cache-Variables_en.md#exercise-1-using-options)
  - [Exercise 2 - `CMAKE` Variables](04-Configuration-and-Cache-Variables_en.md#exercise-2-cmake-variables)
  - [Exercise 3 - CMakePresets.json](04-Configuration-and-Cache-Variables_en.md#exercise-3-cmakepresets-json)
- [Step 4: In-Depth CMake Target Commands](05-In-Depth-CMake-Target-Commands_en.md)
  - [Background](05-In-Depth-CMake-Target-Commands_en.md#background)
  - [Exercise 1 - Features and Definitions](05-In-Depth-CMake-Target-Commands_en.md#exercise-1-features-and-definitions)
  - [Exercise 2 - Compile and Link Options](05-In-Depth-CMake-Target-Commands_en.md#exercise-2-compile-and-link-options)
  - [Exercise 3 - Include and Link Directories](05-In-Depth-CMake-Target-Commands_en.md#exercise-3-include-and-link-directories)
- [Step 5: In-Depth CMake Library Concepts](06-In-Depth-CMake-Library-Concepts_en.md)
  - [Background](06-In-Depth-CMake-Library-Concepts_en.md#background)
  - [Exercise 1 - Static and Shared](06-In-Depth-CMake-Library-Concepts_en.md#exercise-1-static-and-shared)
  - [Exercise 2 - Interface Libraries](06-In-Depth-CMake-Library-Concepts_en.md#exercise-2-interface-libraries)
  - [Exercise 3 - Object Libraries](06-In-Depth-CMake-Library-Concepts_en.md#exercise-3-object-libraries)
- [Step 6: In-Depth System Introspection](07-In-Depth-System-Introspection_en.md)
  - [Background](07-In-Depth-System-Introspection_en.md#background)
  - [Exercise 1 - Check Include File](07-In-Depth-System-Introspection_en.md#exercise-1-check-include-file)
  - [Exercise 2 - Check Source Compiles](07-In-Depth-System-Introspection_en.md#exercise-2-check-source-compiles)
  - [Exercise 3 - Check Interprocedural Optimization](07-In-Depth-System-Introspection_en.md#exercise-3-check-interprocedural-optimization)
- [Step 7: Custom Commands and Generated Files](08-Custom-Commands-and-Generated-Files_en.md)
  - [Background](08-Custom-Commands-and-Generated-Files_en.md#background)
  - [Exercise 1 - Using a Code Generator](08-Custom-Commands-and-Generated-Files_en.md#exercise-1-using-a-code-generator)
- [Step 8: Testing and CTest](09-Testing-and-CTest_en.md)
  - [Background](09-Testing-and-CTest_en.md#background)
  - [Exercise 1 - Adding Tests](09-Testing-and-CTest_en.md#exercise-1-adding-tests)
- [Step 9: Installation Commands and Concepts](10-Installation-Commands-and-Concepts_en.md)
  - [Background](10-Installation-Commands-and-Concepts_en.md#background)
  - [Exercise 1 - Installing Artifacts](10-Installation-Commands-and-Concepts_en.md#exercise-1-installing-artifacts)
  - [Exercise 2 - Exporting Targets](10-Installation-Commands-and-Concepts_en.md#exercise-2-exporting-targets)
  - [Exercise 3 - Exporting a Version File](10-Installation-Commands-and-Concepts_en.md#exercise-3-exporting-a-version-file)
- [Step 10: Finding Dependencies](11-Finding-Dependencies_en.md)
  - [Background](11-Finding-Dependencies_en.md#background)
  - [Exercise 1 - Using `find_package()`](11-Finding-Dependencies_en.md#exercise-1-using-find-package)
  - [Exercise 2 - Transitive Dependencies](11-Finding-Dependencies_en.md#exercise-2-transitive-dependencies)
  - [Exercise 3 - Finding Other Kinds of Files](11-Finding-Dependencies_en.md#exercise-3-finding-other-kinds-of-files)
- [Step 11: Miscellaneous Features](12-Miscellaneous-Features_en.md)
  - [Exercise 1: Target Aliases](12-Miscellaneous-Features_en.md#exercise-1-target-aliases)
  - [Exercise 2: Generator Expressions](12-Miscellaneous-Features_en.md#exercise-2-generator-expressions)
