cmake_minimum_required(VERSION 3.16.3)
project("ViPER4Android")

set(VERSION_CODE "20260322" CACHE STRING "Build version code")
set(VERSION_NAME "1.2.1" CACHE STRING "Build version name")

add_compile_definitions(VERSION_CODE=${VERSION_CODE})
add_compile_definitions(VERSION_NAME="${VERSION_NAME}")

# This check is not needed for NDK.
if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
add_compile_options("SHELL:-mllvm --enable-loopinterchange")
add_compile_options("SHELL:-mllvm --enable-partial-inlining")
add_compile_options("SHELL:-mllvm --enable-dfa-jump-thread")
add_compile_options("SHELL:-mllvm --enable-knowledge-retention")
add_compile_options("SHELL:-mllvm --extra-vectorizer-passes")
add_compile_options("SHELL:-mllvm --enable-loop-flatten")
add_compile_options("SHELL:-mllvm --hot-cold-split")
add_compile_options("SHELL:-mllvm --enable-constraint-elimination")
add_compile_options("SHELL:-funsafe-math-optimizations")
add_link_options("-Wl,-O3,--relax,--lto-O3")
endif()

add_subdirectory(ViPERDSP)

add_library(v4a_re SHARED
    src/ViPER4Android.cpp
    src/ViperContext.cpp
)

target_include_directories(v4a_re PRIVATE
    src
    src/include
)

target_link_libraries(v4a_re ViPERDSP log)
target_compile_options(v4a_re PRIVATE -flto -O3 -DNDEBUG)
