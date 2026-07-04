cmake_minimum_required(VERSION 3.12)
project(luanti)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED TRUE)

# Configuration minimale pour Android
set(BUILD_CLIENT TRUE CACHE BOOL "Build client")
set(BUILD_SERVER FALSE CACHE BOOL "Build server")
set(BUILD_UNITTESTS FALSE CACHE BOOL "No tests")
set(BUILD_DOCUMENTATION FALSE CACHE BOOL "No docs")

# Inclusion forcée des répertoires vitaux
add_subdirectory(lib/lstrpack)
add_subdirectory(lib/sha256)
add_subdirectory(lib/tiniergltf)
add_subdirectory(src)
