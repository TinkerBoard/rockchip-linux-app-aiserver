cmake_minimum_required(VERSION 2.8)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

project(aiserver LANGUAGES CXX C)

set(DESCRIPTION "aiserver projects for smart screen")

set(PROJECT_THIRDPARTY_DIR "${CMAKE_SOURCE_DIR}/thirdparty")
include(clang-format)
include(cppcheck)

if (NOT ${CMAKE_SYSTEM_NAME} STREQUAL "Linux" OR NOT DEFINED CMAKE_SYSTEM_PROCESSOR)
    message(FATAL_ERROR "this project can only be built for linux")
endif()

if (CMAKE_BUILD_TYPE STREQUAL Debug)
    add_definitions(-DDEBUG)
endif()

add_definitions(-DDBUS_HAS_RECURSIVE_MUTEX)

add_definitions(-fPIC)

option(SANITIZER_STATIC "compile with sanitizer (static library linker)" OFF)
if(SANITIZER_STATIC)
    add_definitions(-fsanitize=address -static-libasan -g -ggdb -gdwarf -funwind-tables -rdynamic -O0)
    add_definitions(-fno-stack-protector -fno-omit-frame-pointer -fsanitize-recover=address)
    # add_definitions(-fsanitize=undefined)
    link_libraries(libasan.a dl m rt)
endif()

option(SANITIZER_DYNAMIC "compile with sanitizer (dynamic library linker)" OFF)
if(SANITIZER_DYNAMIC)
    # NOTE: copy libasan.so with manual operation
    add_definitions(-fsanitize=address -g -ggdb -gdwarf -funwind-tables -rdynamic -O0)
    add_definitions(-fno-stack-protector -fno-omit-frame-pointer -fsanitize-recover=address)
    # add_definitions(-fsanitize=undefined)
    link_libraries(asan)
endif()

add_compile_options(-std=c++11 -Wno-psabi)

find_package(Threads REQUIRED)
find_package(Dbus REQUIRED)
find_package(DbusC++ REQUIRED)

add_subdirectory(src)
