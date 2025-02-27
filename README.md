# project_options

A general-purpose CMake library that provides functions that improve the
CMake experience following the best practices.

[![documentation](https://img.shields.io/badge/documentation-blue?style=flat&logo=docs.rs&link=https://aminya.github.io/project_options/)](https://aminya.github.io/project_options/)

[![ci](https://github.com/aminya/project_options/actions/workflows/ci.yml/badge.svg)](https://github.com/aminya/project_options/actions/workflows/ci.yml)

## Features

-   `project_options`:
    -   compiler warnings,
    -   compiler optimizations (intraprocedural, native),
    -   caching (ccache, sccache),
    -   sanitizers,
    -   static code analyzers (clang-tidy, cppcheck, visual studio,
        include-what-you-use),
    -   document generation (doxygen),
    -   test coverage analysis,
    -   precompiled headers,
    -   build time measurement,
    -   unity builds
    -   using custom linkers (e.g. lld)
-   `package_project`: automatic packaging/installation of the project
    for seamless usage via find_package/target_link through CMake's
    FetchContent, vcpkg, etc.
-   `run_vcpkg`: automatic installation of vcpkg and the project
    dependencies
-   `run_conan`: automatic installation of conan and the project
    dependencies
-   `dynamic_project_options`: a wrapper around `project_options` to
    change the options on the fly dynamically
-   `target_link_system_libraries` and
    `target_include_system_directories`: linking/including external
    dependencies/headers without warnings
-   `target_link_cuda`: linking Cuda to a target

## Documentation

The full documentation is available here:

<https://aminya.github.io/project_options/>

## `project_options` function

See the `project_options()` in action in [this template
repository](https://github.com/aminya/cpp_vcpkg_project).
[cpp_vcpkg_project](https://github.com/aminya/cpp_vcpkg_project) has
prepared all the best practices for a production-ready C++ project.

### `project` and `project_options`

Here is an example of the usage:

``` cmake
cmake_minimum_required(VERSION 3.20)

# set a default CXX standard for the tools and targets that do not specify them.
# If commented, the latest supported standard for your compiler is automatically set.
# set(CMAKE_CXX_STANDARD 20)

include(FetchContent)
if(CMAKE_VERSION VERSION_GREATER_EQUAL "3.24.0")
  cmake_policy(SET CMP0135 NEW)
endif()

# Add project_options from https://github.com/aminya/project_options
# Change the version in the following URL to update the package (watch the releases of the repository for future updates)
set(PROJECT_OPTIONS_VERSION "v0.41.0")
FetchContent_Declare(
  _project_options
  URL https://github.com/aminya/project_options/archive/refs/tags/${PROJECT_OPTIONS_VERSION}.zip)
FetchContent_MakeAvailable(_project_options)
include(${_project_options_SOURCE_DIR}/Index.cmake)

# install vcpkg dependencies: - should be called before defining project()
run_vcpkg(
    VCPKG_URL "https://github.com/microsoft/vcpkg.git"
    VCPKG_REV "86a181505ac6460f98496a79abdee6a0f49905ec"
)
# Install conan dependencies: - should be called before defining project()
run_conan()

# Set the project name and language
project(myproject LANGUAGES CXX C)

# Build Features
option(FEATURE_TESTS "Enable the tests" OFF)
option(FEATURE_DOCS "Enable the docs" OFF)

# vcpkg test feature
if(FEATURE_TESTS)
  list(APPEND VCPKG_MANIFEST_FEATURES "tests")
endif()

# Enable sanitizers and static analyzers when running the tests
if(FEATURE_TESTS)
  set(ENABLE_CLANG_TIDY "ENABLE_CLANG_TIDY")
  set(ENABLE_CPPCHECK "ENABLE_CPPCHECK")
  set(ENABLE_COVERAGE "ENABLE_COVERAGE")

  set(ENABLE_SANITIZER_ADDRESS "ENABLE_SANITIZER_ADDRESS")
  set(ENABLE_SANITIZER_UNDEFINED "ENABLE_SANITIZER_UNDEFINED")
endif()

# Enable doxgen for the docs
if(FEATURE_DOCS)
  set(ENABLE_DOXYGEN "ENABLE_DOXYGEN")
endif()

# Initialize project_options variable related to this project
# This overwrites `project_options` and sets `project_warnings`
# uncomment to enable the options. Some of them accept one or more inputs:
project_options(
      PREFIX "myproject"
      ENABLE_CACHE
      ${ENABLE_CPPCHECK}
      ${ENABLE_CLANG_TIDY}
      ENABLE_VS_ANALYSIS
      # ENABLE_INTERPROCEDURAL_OPTIMIZATION
      # ENABLE_NATIVE_OPTIMIZATION
      ${ENABLE_DOXYGEN}
      ${ENABLE_COVERAGE}
      ${ENABLE_SANITIZER_ADDRESS}
      ${ENABLE_SANITIZER_UNDEFINED}
      # ${ENABLE_SANITIZER_THREAD}
      # ${ENABLE_SANITIZER_MEMORY}
      # ENABLE_SANITIZER_POINTER_COMPARE
      # ENABLE_SANITIZER_POINTER_SUBTRACT
      # ENABLE_CONTROL_FLOW_PROTECTION
      # ENABLE_STACK_PROTECTION
      # ENABLE_OVERFLOW_PROTECTION
      # ENABLE_ELF_PROTECTION
      # ENABLE_RUNTIME_SYMBOLS_RESOLUTION
      # ENABLE_COMPILE_COMMANDS_SYMLINK
      # ENABLE_PCH
      # PCH_HEADERS
      # WARNINGS_AS_ERRORS
      # ENABLE_INCLUDE_WHAT_YOU_USE
      # ENABLE_GCC_ANALYZER
      # ENABLE_BUILD_WITH_TIME_TRACE
      # ENABLE_UNITY
      # LINKER "lld"
)
```

Then add the executables or libraries to the project:

### [executable usage](https://github.com/aminya/cpp_vcpkg_project/tree/main/my_exe)

``` cmake
add_executable(main main.cpp)

# link project_options/warnings
target_link_libraries(main
  PRIVATE myproject_project_options myproject_project_warnings
)

# Find dependencies:
target_find_dependencies(main
  PRIVATE_CONFIG
  fmt
  Eigen3
)

# Link dependencies
target_link_system_libraries(main
  PRIVATE
  fmt::fmt
  Eigen3::Eigen
)

# Package the project
package_project(TARGETS main)
```

### [library usage](https://github.com/aminya/cpp_vcpkg_project/tree/main/my_lib)

``` cmake
add_library(my_lib "./src/my_lib/lib.cpp")

# link project_options/warnings
target_link_libraries(my_lib
  PRIVATE myproject_project_options myproject_project_warnings
)

# Includes:
target_include_interface_directories(my_lib "${CMAKE_CURRENT_SOURCE_DIR}/include")

# Find dependencies:
target_find_dependencies(my_lib
  PRIVATE_CONFIG
  fmt
  Eigen3
)

# Link dependencies:
target_link_system_libraries(my_lib
  PRIVATE
  fmt::fmt
  Eigen3::Eigen
)

# Package the project
package_project(
  # Note that you must export `myproject_project_options` and `myproject_project_warnings` for `my_lib`
  TARGETS my_lib myproject_project_options myproject_project_warnings
)
```

### [header-only library usage](https://github.com/aminya/cpp_vcpkg_project/tree/main/my_header_lib)

``` cmake
add_library(my_header_lib INTERFACE)

# link project_options/warnings
target_link_libraries(my_header_lib
  INTERFACE myproject_project_options myproject_project_warnings
)

# Includes:
target_include_interface_directories(my_header_lib "${CMAKE_CURRENT_SOURCE_DIR}/include")

# Find dependencies:
target_find_dependencies(my_header_lib
  INTERFACE_CONFIG
  fmt
  Eigen3
)

# Link dependencies:
target_link_system_libraries(my_header_lib
  INTERFACE
  fmt::fmt
  Eigen3::Eigen
)

# Package the project
package_project(
  TARGETS my_header_lib myproject_project_options myproject_project_warnings
)
```

## License

This project can be used under the terms of either the [MIT
license](../../LICENSE.txt) or the [Unlicense](../../Unlicense.txt)
depending on your choice.
