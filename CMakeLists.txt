cmake_minimum_required(VERSION 3.17)

project(node-addon-lib VERSION 1.0.0)

set(CMAKE_CXX_STANDARD 17)

include(FetchContent)

if (POLICY CMP0135)
    cmake_policy(SET CMP0135 NEW)
endif ()

message(STATUS "Setting up node addon environment...")

find_program(npm NAMES npm.cmd npm REQUIRED)
message(STATUS "Found npm: ${npm}")

find_program(node node REQUIRED)
message(STATUS "Found node: ${node}")

execute_process(
        COMMAND ${node} --version
        OUTPUT_VARIABLE NODE_VERSION
        COMMAND_ERROR_IS_FATAL ANY
)
string(REGEX REPLACE "[\r\n]" "" NODE_VERSION ${NODE_VERSION})
message(STATUS "Node version: ${NODE_VERSION}")

execute_process(
        COMMAND ${node} -p "process.arch"
        OUTPUT_VARIABLE NODE_ARCH
        COMMAND_ERROR_IS_FATAL ANY
)
string(REGEX REPLACE "[\r\n]" "" NODE_ARCH ${NODE_ARCH})
message(STATUS "Node architecture: ${NODE_ARCH}")

set(ARCH64_LIST "x64;x86_64;arm64;ppc64")
if (${NODE_ARCH} IN_LIST ARCH64_LIST)
    set(NODE_ARCH x64)
    set(NODE_ARCH_BIT 64)
else ()
    set(NODE_ARCH x86)
    set(NODE_ARCH_BIT 32)
endif ()

if (CMAKE_SIZEOF_VOID_P EQUAL 8)
    set(CONFIG_ARCH_BIT 64)
else ()
    set(CONFIG_ARCH_BIT 32)
endif ()

if (NOT CONFIG_ARCH_BIT EQUAL NODE_ARCH_BIT)
    message(WARNING "Using ${CONFIG_ARCH_BIT}-bit toolset while ${NODE_ARCH_BIT}-bit node version detected. Linking will not work with the node version (${NODE_ARCH_BIT}-bit) installed on this system.")
endif ()

if (WIN32 AND MINGW)
    if (CMAKE_DLLTOOL)
        message(STATUS "Found dlltool: ${CMAKE_DLLTOOL}")
    else ()
        message(FATAL_ERROR "dlltool not found, set CMAKE_DLLTOOL")
    endif ()
endif ()

message(STATUS "Downloading node headers from https://nodejs.org/dist/${NODE_VERSION}/node-${NODE_VERSION}-headers.tar.gz")
FetchContent_Declare(
        node-${NODE_VERSION}
        URL https://nodejs.org/dist/${NODE_VERSION}/node-${NODE_VERSION}-headers.tar.gz
)
FetchContent_MakeAvailable(node-${NODE_VERSION})
set(NODE_INC ${node-${NODE_VERSION}_SOURCE_DIR}/include)
message(STATUS "Installing node api and addon headers ${CMAKE_CURRENT_BINARY_DIR}")
execute_process(
        COMMAND ${npm} install node-api-headers node-addon-api
        WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
        OUTPUT_QUIET
        COMMAND_ERROR_IS_FATAL ANY
)
execute_process(
        COMMAND ${node} -p "require ('node-api-headers').include_dir"
        WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
        OUTPUT_VARIABLE NODE_API_INC
        COMMAND_ERROR_IS_FATAL ANY
)
execute_process(
        COMMAND ${node} -p "require ('node-addon-api') .include.replaceAll ('\"','')"
        WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
        OUTPUT_VARIABLE NODE_ADDON_INC
        COMMAND_ERROR_IS_FATAL ANY
)
string(REGEX REPLACE "[\r\n]" "" NODE_API_INC ${NODE_API_INC})
set(NODE_API_INC ${NODE_API_INC})
string(REGEX REPLACE "[\r\n]" "" NODE_ADDON_INC ${NODE_ADDON_INC})
set(NODE_ADDON_INC ${NODE_ADDON_INC})

if (WIN32 AND MINGW)
    message(STATUS "Creating node.def file...")
    execute_process(
            COMMAND ${node} -e
            "const symbols = require ('node-api-headers') .symbols;
            console.log ('NAME NODE.EXE');
            console.log ('EXPORTS');
            for (const symbol in symbols) {
            console.log (symbols[symbol].js_native_api_symbols.join ('\\n'));
            }
            for (const symbol in symbols) {
            console.log (symbols[symbol].node_api_symbols.join ('\\n'));
            }
            "
            WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
            OUTPUT_FILE node.def
            COMMAND_ERROR_IS_FATAL ANY
    )
    message(STATUS "Generating node.lib...")
    execute_process(COMMAND ${CMAKE_DLLTOOL} -d ${CMAKE_CURRENT_BINARY_DIR}/node.def -l ${CMAKE_CURRENT_BINARY_DIR}/node.lib COMMAND_ERROR_IS_FATAL ANY)
    set(NODE_LIB ${CMAKE_CURRENT_BINARY_DIR}/node.lib)
elseif (MSVC)
    message(STATUS "Downloading import library for msvc from https://nodejs.org/dist/${NODE_VERSION}/win-${NODE_ARCH}/node.lib")
    FetchContent_Declare(
            node-imp-lib
            URL https://nodejs.org/dist/${NODE_VERSION}/win-${NODE_ARCH}/node.lib
            DOWNLOAD_NO_EXTRACT TRUE
    )
    FetchContent_MakeAvailable(node-imp-lib)
    set(NODE_LIB ${node-imp-lib_SOURCE_DIR}/node.lib)
endif ()

message(STATUS "NODE_LIB: ${NODE_LIB}")
message(STATUS "NODE_INC: ${NODE_INC}")
message(STATUS "NODE_API_INC: ${NODE_API_INC}")
message(STATUS "NODE_ADDON_INC: ${NODE_ADDON_INC}")

add_library(
        node-addon-lib
        INTERFACE
)

target_compile_definitions(
        node-addon-lib
        INTERFACE
        BUILDING_NODE_EXTENSION
)

target_link_libraries(
        node-addon-lib
        INTERFACE
        ${NODE_LIB}
)

target_include_directories(
        node-addon-lib
        INTERFACE
        ${NODE_INC}
        ${NODE_API_INC}
        ${NODE_ADDON_INC}
)
