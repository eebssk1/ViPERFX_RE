cmake_minimum_required(VERSION 3.16.3)
project("ViPER4Android")

set(VERSION_CODE "00000000" CACHE STRING "Build version code")
set(VERSION_NAME "00000000" CACHE STRING "Build version name")

add_compile_definitions(VERSION_CODE=${VERSION_CODE})
add_compile_definitions(VERSION_NAME="${VERSION_NAME}")
add_compile_definitions(NDEBUG)

# the check is not needed for androidndk
if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
set(EXTRA_LLVM_FLAGS
"-mllvm --polly"
"-mllvm --polly-run-inliner"
"-mllvm --polly-invariant-load-hoisting"
"-mllvm --polly-run-dce"
"-mllvm --polly-vectorizer=stripmine"
"-mllvm --polly-default-tile-size=24"
"-mllvm --polly-register-tiling"
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
add_compile_options("SHELL:-mllvm --polly-register-tile-sizes=4,4,1")
add_link_options("SHELL:-Xlinker -mllvm -Xlinker --polly-register-tile-sizes=4,4,1")

add_link_options("-Wl,-O3,--relax,--lto-O3,--icf=safe")
endif()

# No compiler check
set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)

add_compile_options("-funsafe-math-optimizations")
add_compile_options("-fno-math-errno")
add_link_options("-funsafe-math-optimizations")
add_link_options("-fno-math-errno")

#baseline opt
add_compile_options("-march=armv8.2-a+dotprod")
add_compile_options("-mtune=cortex-a710")

add_link_options("-march=armv8.2-a+dotprod")
add_link_options("-mtune=cortex-a710")

set(V4A_RE_COMMON_FLAGS
    -ffunction-sections
    -fdata-sections
    -fno-exceptions
    -fno-rtti
    -fno-unwind-tables
    -fno-asynchronous-unwind-tables
    -fvisibility=hidden
    -fvisibility-inlines-hidden)

add_compile_options(${V4A_RE_COMMON_FLAGS})
add_link_options(${V4A_RE_COMMON_FLAGS})

add_subdirectory(ViPERDSP)

add_library(v4a_re SHARED
    src/ViPER4Android.cpp
    src/ViperContext.cpp
)

target_include_directories(v4a_re PRIVATE
    src
    src/include
)

target_compile_options(v4a_re PRIVATE -Wall -Wno-unused-parameter -Wno-unused-variable
    -Wno-unused-function)

target_link_options(v4a_re PRIVATE -Wl,--gc-sections -Wl,--strip-all
    -Wl,--exclude-libs,ALL -fvisibility=hidden)

target_link_libraries(v4a_re ViPERDSP log)
