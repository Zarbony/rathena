cmake_minimum_required(VERSION 3.13)
include(./ext/c4core/cmake/c4Project.cmake)
project(ryml
    DESCRIPTION "Rapid YAML parsing and emitting"
    HOMEPAGE_URL "https://github.com/biojppm/rapidyaml"
    LANGUAGES CXX)
include(./compat.cmake)

c4_project(VERSION 0.4.0 STANDALONE
    AUTHOR "Joao Paulo Magalhaes <dev@jpmag.me>")


#-------------------------------------------------------

option(RYML_WITH_TAB_TOKENS "Enable parsing of tabs after ':' and '-'. This is costly and disabled by default." OFF)
option(RYML_DEFAULT_CALLBACKS "Enable ryml's default implementation of callbacks: allocate(), free(), error()" ON)
option(RYML_BUILD_TOOLS "build tools" OFF)
option(RYML_BUILD_API "Enable API generation (python, etc)" OFF)
option(RYML_DBG "Enable (very verbose) ryml debug prints." OFF)


#-------------------------------------------------------

c4_require_subproject(c4core INCORPORATE
    SUBDIRECTORY ${RYML_EXT_DIR}/c4core)

c4_add_library(ryml
    SOURCES
        ryml.hpp
        ryml_std.hpp
        c4/yml/detail/checks.hpp
        c4/yml/detail/parser_dbg.hpp
        c4/yml/detail/print.hpp
        c4/yml/detail/stack.hpp
        c4/yml/common.hpp
        c4/yml/common.cpp
        c4/yml/emit.def.hpp
        c4/yml/emit.hpp
        c4/yml/export.hpp
        c4/yml/node.hpp
        c4/yml/node.cpp
        c4/yml/parse.hpp
        c4/yml/parse.cpp
        c4/yml/preprocess.hpp
        c4/yml/preprocess.cpp
        c4/yml/std/map.hpp
        c4/yml/std/std.hpp
        c4/yml/std/string.hpp
        c4/yml/std/vector.hpp
        c4/yml/tree.hpp
        c4/yml/tree.cpp
        c4/yml/writer.hpp
        c4/yml/yml.hpp
        ryml.natvis
    SOURCE_ROOT ${RYML_SRC_DIR}
    INC_DIRS
        $<BUILD_INTERFACE:${RYML_SRC_DIR}>
        $<INSTALL_INTERFACE:include>
    LIBS c4core
    INCORPORATE c4core
    )

if(RYML_WITH_TAB_TOKENS)
    target_compile_definitions(ryml PUBLIC RYML_WITH_TAB_TOKENS)
endif()

if(NOT RYML_DEFAULT_CALLBACKS)
    target_compile_definitions(ryml PRIVATE RYML_NO_DEFAULT_CALLBACKS)
endif()

if(RYML_DBG)
    target_compile_definitions(ryml PRIVATE RYML_DBG)
endif()


#-------------------------------------------------------

c4_install_target(ryml)
c4_install_exports(DEPENDENCIES c4core)
c4_pack_project()


#-------------------------------------------------------
# developer targets


# extern libraries, used only for testing/benchmarking
if(RYML_BUILD_TESTS OR RYML_BUILD_BENCHMARKS OR RYML_BUILD_TOOLS)
    include(ext/testbm.cmake)
endif()

if(RYML_BUILD_TOOLS)
    add_subdirectory(tools)
endif()

c4_add_dev_targets()

add_custom_target(ryml-uninstall
    "${CMAKE_COMMAND}" -P "${PROJECT_SOURCE_DIR}/cmake/uninstall.cmake"
)
