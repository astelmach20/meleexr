cmake_minimum_required(VERSION 3.20)
project(meleexr LANGUAGES C)

# Phase 0: compile every game + HSD translation unit from the decomp against
# Aurora's Dolphin SDK headers (mirrors doldecomp/melee/.nix/CMakeLists.txt,
# but derived by glob so it tracks upstream automatically).
set(MELEE_SRC "${CMAKE_CURRENT_SOURCE_DIR}/extern/melee/src")
set(AURORA_SRC "${CMAKE_CURRENT_SOURCE_DIR}/extern/aurora")

file(GLOB_RECURSE MELEE_SOURCES CONFIGURE_DEPENDS
    "${MELEE_SRC}/melee/*.c"
    "${MELEE_SRC}/sysdolphin/*.c")
list(LENGTH MELEE_SOURCES MELEE_TU_COUNT)
message(STATUS "meleexr: ${MELEE_TU_COUNT} decomp translation units")

option(MELEEXR_M32 "Build with -m32 (decomp assumes 32-bit pointers)" OFF)
if (MELEEXR_M32)
    add_compile_options(-m32)
    add_link_options(-m32)
endif ()

# MELEE_DECOMP_COMPAT enables Aurora's arity shims for SDK-shaped GXSetArray/CARDInit calls.
add_compile_definitions(TARGET_PC MELEE_DECOMP_COMPAT)

# The font atlases (*.inc) are generated from game assets by the decomp's build and are
# not in the tree. Phase 0 only needs the TUs to compile, so provide empty placeholders.
set(MELEEXR_GEN "${CMAKE_BINARY_DIR}/gen")
foreach (inc sysdolphin/baselib/sislib_font.inc sysdolphin/baselib/debug_font.inc)
    if (NOT EXISTS "${MELEE_SRC}/${inc}")
        file(WRITE "${MELEEXR_GEN}/${inc}"
             "/* meleexr: placeholder for generated font atlas (Phase 0, compile-only) */\n")
    endif ()
endforeach ()
add_library(melee STATIC ${MELEE_SOURCES})
target_include_directories(melee PRIVATE
    "${AURORA_SRC}/include"
    "${MELEE_SRC}"
    "${MELEEXR_GEN}")
target_compile_options(melee PRIVATE
    -std=gnu99 -fno-strict-aliasing -fwrapv -ffp-contract=off
    -Wno-everything -w)
