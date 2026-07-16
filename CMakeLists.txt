cmake_minimum_required(VERSION 3.16.3)
project("ViPER4Android")

set(VERSION_CODE "20260322" CACHE STRING "Build version code")
set(VERSION_NAME "1.2.1" CACHE STRING "Build version name")

add_compile_definitions(VERSION_CODE=${VERSION_CODE})
add_compile_definitions(VERSION_NAME="${VERSION_NAME}")

# the check is not needed for androidndk
if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
set(EXTRA_LLVM_FLAGS
"-mllvm --polly"
"-mllvm --polly-run-inliner"
"-mllvm --polly-invariant-load-hoisting"
"-mllvm --polly-run-dce"
"-mllvm --polly-vectorizer=stripmine"
"-mllvm --enable-loopinterchange"
"-mllvm --enable-partial-inlining"
"-mllvm --disable-mr-partial-inlining"
"-mllvm --enable-loop-flatten"
"-mllvm --enable-jump-table-to-switch"
"-mllvm --enable-dfa-jump-thread"
"-mllvm --enable-knowledge-retention"
"-mllvm --extra-vectorizer-passes"
"-mllvm --hot-cold-split"
"-mllvm --regalloc-enable-advisor=release"
"-mllvm --enable-local-reassign=true"
"-mllvm --regalloc-eviction-max-interference-cutoff=20"
)

foreach(FLAG IN LISTS EXTRA_LLVM_FLAGS)
    add_compile_options("SHELL:${FLAG}")
endforeach()
foreach(FLAG IN LISTS EXTRA_LLVM_FLAGS)
    string(REPLACE " " "," LINKER_FLAG "${FLAG}")
    add_link_options("-Wl,${LINKER_FLAG}")
endforeach()

add_compile_options("-funsafe-math-optimizations")
add_link_options("-funsafe-math-optimizations")

add_link_options("-Wl,-O3,--relax,--lto-O3,-S")
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
