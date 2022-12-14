# -----------------------------------------------------------------------------
# @brief  : CmakeFile for building the executable.
# @author : Enrico Fraccaroli
# @create : 02/02/2018
# @update : 20/08/2019
# -----------------------------------------------------------------------------

# -----------------------------------------------------------------------------
# Set the minimum CMake version.
# -----------------------------------------------------------------------------
cmake_minimum_required(VERSION 2.8...3.22)

# Set the project name.
project(ordered_map CXX)

# -----------------------------------------------------------------------------
# Add options.
# -----------------------------------------------------------------------------
option(STRICT_WARNINGS "Enable strict compiler warnings" ON)
option(WARNINGS_AS_ERRORS "Treat all warnings as errors" OFF)
option(BUILD_EXAMPLES "Build examples" OFF)

# -----------------------------------------------------------------------------
# LIBRARY
# -----------------------------------------------------------------------------
# Add the C++ library.
add_library(ordered_map INTERFACE)
add_library(ordered_map::ordered_map ALIAS ordered_map)

# Inlcude header directories.
target_include_directories(ordered_map INTERFACE ${PROJECT_SOURCE_DIR}/include)

# -----------------------------------------------------------------------------
# Set the compilation flags.
# -----------------------------------------------------------------------------
target_compile_options(ordered_map INTERFACE -std=c++98)

if(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    # Disable warnings that suggest using MSVC-specific safe functions
    target_compile_definitions(ordered_map INTERFACE _CRT_SECURE_NO_WARNINGS)

    if(WARNINGS_AS_ERRORS)
        target_compile_options(ordered_map INTERFACE /WX)
    endif()
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    if(WARNINGS_AS_ERRORS)
        target_compile_options(ordered_map INTERFACE -Werror)
    endif()
endif()

if(STRICT_TYPE_CHECK)
    # Pass the define to the code.
    target_compile_definitions(ordered_map INTERFACE STRICT_TYPE_CHECK)
endif()

if(STRICT_WARNINGS)
    if(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
        # Mark system headers as external for MSVC explicitly
        # https://devblogs.microsoft.com/cppblog/broken-warnings-theory
        target_compile_options(ordered_map INTERFACE /experimental:external)
        target_compile_options(ordered_map INTERFACE /external:I ${CMAKE_BINARY_DIR})
        target_compile_options(ordered_map INTERFACE /external:anglebrackets)
        target_compile_options(ordered_map INTERFACE /external:W0)

        target_compile_options(ordered_map INTERFACE /W4)
    elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        target_compile_options(ordered_map INTERFACE -Wall -Wextra -Wconversion -pedantic)
    endif()
endif()

# -----------------------------------------------------------------------------
# DOCUMENTATION
# -----------------------------------------------------------------------------
find_package(Doxygen)

if(DOXYGEN_FOUND)
    set(DOXYGEN_PROJECT_NAME "Ordered Map Library")
    set(DOXYGEN_USE_MDFILE_AS_MAINPAGE ${PROJECT_SOURCE_DIR}/README.md)
    set(DOXYGEN_SHOW_INCLUDE_FILES NO)
    set(DOXYGEN_GENERATE_TREEVIEW NO)
    set(DOXYGEN_HTML_STYLESHEET ${PROJECT_SOURCE_DIR}/docs/doxygen.css)
    doxygen_add_docs(
        ${PROJECT_NAME}_documentation
        ${PROJECT_SOURCE_DIR}/include/ordered_map/ordered_map.hpp
    )
endif()

# -----------------------------------------------------------------------------
# Examples
# -----------------------------------------------------------------------------
if(BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()

# -----------------------------------------------------------------------------
# Testing programs.
# -----------------------------------------------------------------------------
enable_testing()
add_subdirectory(tests)