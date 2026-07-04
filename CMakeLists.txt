cmake_minimum_required(VERSION 3.12)
if(POLICY CMP0177)
    cmake_policy(SET CMP0177 NEW)
endif()

project(luanti)
set(PROJECT_NAME_CAPITALIZED "Luanti")

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED TRUE)
set(GCC_MINIMUM_VERSION "7.5")
set(CLANG_MINIMUM_VERSION "7.0.1")

set(VERSION_MAJOR 5)
set(VERSION_MINOR 17)
set(VERSION_PATCH 0)
set(VERSION_EXTRA "" CACHE STRING "Stuff to append to version string")
set(DEVELOPMENT_BUILD TRUE)

set(VERSION_STRING "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")
if(VERSION_EXTRA)
        set(VERSION_STRING "${VERSION_STRING}-${VERSION_EXTRA}")
elseif(DEVELOPMENT_BUILD)
        set(VERSION_STRING "${VERSION_STRING}-dev")
endif()

if (CMAKE_BUILD_TYPE STREQUAL Debug)
        set(VERSION_STRING "${VERSION_STRING}-debug")
endif()

set(BUILD_CLIENT TRUE CACHE BOOL "Build client")
set(BUILD_SERVER FALSE CACHE BOOL "Build server")
set(BUILD_UNITTESTS TRUE CACHE BOOL "Build unittests")
set(BUILD_BENCHMARKS FALSE CACHE BOOL "Build benchmarks")
set(BUILD_DOCUMENTATION TRUE CACHE BOOL "Build documentation")

set(DEFAULT_ENABLE_LTO TRUE)
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
  set(DEFAULT_ENABLE_LTO FALSE)
endif()

if((WIN32 AND CMAKE_CXX_COMPILER_ID STREQUAL "GNU") OR APPLE)
        set(DEFAULT_ENABLE_LTO FALSE)
endif()
set(ENABLE_LTO ${DEFAULT_ENABLE_LTO} CACHE BOOL "Use Link Time Optimization")

set(BUILD_WITH_TRACY FALSE CACHE BOOL "Fetch and build with the Tracy profiler client")
set(FETCH_TRACY_GIT_TAG "master" CACHE STRING "Git tag for fetching Tracy client")

set(DEFAULT_RUN_IN_PLACE FALSE)
if(WIN32)
        set(DEFAULT_RUN_IN_PLACE TRUE)
endif()
set(RUN_IN_PLACE ${DEFAULT_RUN_IN_PLACE} CACHE BOOL "Run directly in source directory structure")

set(WARN_ALL TRUE CACHE BOOL "Enable -Wall for Release build")

if(NOT CMAKE_BUILD_TYPE)
        set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type: Debug or Release" FORCE)
endif()

set(ENABLE_UPDATE_CHECKER (NOT ${DEVELOPMENT_BUILD}) CACHE BOOL "Whether to enable update checks by default")

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")

if(ANDROID)
        cmake_minimum_required(VERSION 3.20)
        include(AndroidLibs)
endif()

if(BUILD_CLIENT)
    add_subdirectory(irr EXCLUDE_FROM_ALL)
    if(NOT TARGET IrrlichtMt)
        message(FATAL_ERROR "IrrlichtMt project is missing a CMake target?!")
    endif()
else()
    add_library(IrrlichtMt::IrrlichtMt INTERFACE IMPORTED)
    set_target_properties(IrrlichtMt::IrrlichtMt PROPERTIES
        INTERFACE_INCLUDE_DIRECTORIES "${CMAKE_CURRENT_SOURCE_DIR}/irr/include")
endif()

if(ENABLE_LTO OR CMAKE_INTERPROCEDURAL_OPTIMIZATION)
    include(CheckIPOSupported)
    check_ipo_supported(RESULT lto_supported OUTPUT lto_output)
    if(lto_supported)
        set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
    endif()
endif()

# Installation paths and Logic
if(WIN32)
    set(SHAREDIR ".")
    set(BINDIR "bin")
    set(DOCDIR "doc")
    set(EXAMPLE_CONF_DIR ".")
    set(LOCALEDIR "locale")
elseif(UNIX)
    include(GNUInstallDirs)
    set(SHAREDIR "${CMAKE_INSTALL_FULL_DATADIR}/${PROJECT_NAME}")
    set(BINDIR "${CMAKE_INSTALL_FULL_BINDIR}")
    set(DOCDIR "${CMAKE_INSTALL_FULL_DOCDIR}")
    set(MANDIR "${CMAKE_INSTALL_FULL_MANDIR}")
    set(EXAMPLE_CONF_DIR ${DOCDIR})
endif()

# Conditionals to safely add subdirectories
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/builtin")
    add_subdirectory(builtin)
endif()

add_subdirectory(lib/lstrpack)
add_subdirectory(lib/sha256)
add_subdirectory(lib/tiniergltf)

if(BUILD_UNITTESTS OR BUILD_BENCHMARKS)
    add_subdirectory(lib/catch2)
endif()

add_subdirectory(src)

include(CPack)
