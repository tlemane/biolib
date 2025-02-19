cmake_minimum_required(VERSION 3.13)
project(BIOLIB VERSION 0.0.1)

set(CMAKE_CXX_STANDARD 17)
if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release")
endif ()

option(BIOLIB_X86_64 "x86-64 build" OFF)
option(BIOLIB_USE_SANITIZERS "Compile with asan" OFF)
option(BIOLIB_TESTS "Compile tests" OFF)

add_library(build_flags INTERFACE)
target_compile_options(build_flags INTERFACE
  $<$<CONFIG:Debug>:-O -ggdb>
  $<$<CONFIG:Release>:-O3 -DNDEBUG>
  $<$<BOOL:${BIOLIB_X86_64}>:-march=x86-64 -march=native -mbmi2 -msse4.2>
)

target_link_options(build_flags INTERFACE
  $<$<CXX_COMPILER_ID:Clang>:-stdlib=libc++ -lc++abi>
)

add_library(warning_flags INTERFACE)
target_compile_options(warning_flags INTERFACE
  "-Wall"
  "-Wextra"
  "-Werror"
  "-Wno-sign-compare"
)

add_library(asan_flags INTERFACE)
target_compile_options(asan_flags INTERFACE
  $<$<BOOL:${BIOLIB_USE_SANITIZERS}>:-fsanitize=address -fno-omit-frame-pointer>
)
target_link_options(asan_flags INTERFACE
  $<$<BOOL:${BIOLIB_USE_SANITIZERS}>:-fsanitize=address -fno-omit-frame-pointer>
)
target_link_libraries(asan_flags INTERFACE
  $<$<BOOL:${BIOLIB_USE_SANITIZERS}>:-fsanitize=address -fno-omit-frame-pointer>
)

file(GLOB_RECURSE BUNDLED_SOURCES "${CMAKE_SOURCE_DIR}/bundled/*.cpp")

add_subdirectory(src)

if (${BIOLIB_TESTS})
  enable_testing()
  add_subdirectory(tests)
endif()
