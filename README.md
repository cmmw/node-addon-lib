# node-addon-lib

[![License: MIT](https://img.shields.io/badge/License-MIT-brightgreen.svg)](https://opensource.org/licenses/MIT)

A CMake project for developing native node addons with `MinGW`, `MSVC` or `GCC` without `node-gyp` or `cmake-js`. All
necessary files are downloaded during configuration and a target called `node-addon-lib` is created. To use the library,
simply add this project via `add_subdirectory` or `FetchContent` and link it to your node addon library:

```cmake
FetchContent_Declare(
        node-addon-lib
        GIT_REPOSITORY https://github.com/cmmw/node-addon-lib.git
)
FetchContent_MakeAvailable(node-addon-lib)

target_link_libraries(mylib PRIVATE node-addon-lib)
```

The CMake script will automatically download the correct version of all the necessary node headers corresponding to the
node version installed on the system.

The project was successfully tested with MinGW, MSVC and GCC.

### MSVC

If you are using MSVC keep in mind that the default toolset used by CMake is 32-bit which will generate a 32-bit
library. If you are using a 64-bit installation of node, linking will fail. To build a 64-bit library with MSVC define
the toolset when calling CMake:

```shell
cmake -DCMAKE_GENERATOR_PLATFORM=x64 -S . -B build
```

## Requirements

An installation of `node` and `npm` is required for the CMake script to create the library.

## Motivation

Since most C++ IDE's today work with CMake projects and don't require to set up the project manually I wanted to have a
CMake project for developing node addons without having to use external build systems/software.

## Example

A self-contained example program which includes this project with FetchContent can be found in the `example` directory.

## License

The content of this project (excluding 3rd party libraries) is licensed under
the [MIT license](https://github.com/cmmw/imgui-glfw-glad/blob/master/LICENSE.md)
. Please refer to the particular libraries homepage/repository for more
information regarding licensing.
