cmake_minimum_required(VERSION 3.17)

include(FetchContent)

project(example)

set(CMAKE_CXX_STANDARD 17)

add_compile_definitions(BUILDING_NODE_EXTENSION)

FetchContent_Declare(
        node-addon-lib
        GIT_REPOSITORY https://github.com/cmmw/node-addon-lib.git
)
FetchContent_MakeAvailable(node-addon-lib)

add_library(
        example
        SHARED
        main.cpp
)

target_link_libraries(
        example
        PRIVATE
        node-addon-lib
)

#Place the compiled lib into the node_modules folder so that our test javascript file find's it without specifying the path
set_target_properties(
        example PROPERTIES
        RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/node_modules
        RUNTIME_OUTPUT_DIRECTORY_DEBUG ${PROJECT_SOURCE_DIR}/node_modules
        RUNTIME_OUTPUT_DIRECTORY_RELEASE ${PROJECT_SOURCE_DIR}/node_modules
        LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/node_modules
        LIBRARY_OUTPUT_DIRECTORY_DEBUG ${PROJECT_SOURCE_DIR}/node_modules
        LIBRARY_OUTPUT_DIRECTORY_RELEASE ${PROJECT_SOURCE_DIR}/node_modules
        SUFFIX ".node"
        PREFIX ""
)
