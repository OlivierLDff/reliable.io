#
#   reliable.io CMake build script
#
#   Copyright Olivier Le Doeuff 2019
#
#   Revision:
#   - Olivier Le Doeuff [19/08/2019] : Initial work
#       - reliable.io static or shared library build (RELIABLE_BUILD_SHARED)
#       - Test integration with ctest (RELIABLE_ENABLE_TESTS)
#       - Generate examples (RELIABLE_ENABLE_EXAMPLES)
#       - Install reliable.io (RELIABLE_ENABLE_INSTALL)
#

# ┌──────────────────────────────────────────────────────────────────┐
# │                       CMAKE PROPERTIES                           │
# └──────────────────────────────────────────────────────────────────┘

CMAKE_MINIMUM_REQUIRED( VERSION 3.11 )

# ┌──────────────────────────────────────────────────────────────────┐
# │                       PROJECT SETTINGS                           │
# └──────────────────────────────────────────────────────────────────┘

# OPTIONS

# General
SET( RELIABLE_PROJECT "reliable" CACHE STRING "Project Name")
SET( RELIABLE_TARGET ${RELIABLE_PROJECT} CACHE STRING "reliable.io library name" )
SET( RELIABLE_VERSION "1.2" CACHE STRING "reliable.io current version, this is only decorative and will not configure any files" FORCE )
SET( RELIABLE_BUILD_SHARED OFF CACHE BOOL "Build as a shared library (ON) or as static (OFF)" )
SET( RELIABLE_FOLDER_PREFIX ${RELIABLE_PROJECT} CACHE STRING "Prefix folder for all reliable.io generated targets in generated project (only decorative)" )

# Tests
SET( RELIABLE_ENABLE_TESTS OFF CACHE BOOL "Create or not a target for test (compatible with CTests)" )
SET( RELIABLE_TESTS_PREFIX ${RELIABLE_PROJECT} CACHE STRING "Prefix for all reliable tests" )

# Examples
SET( RELIABLE_ENABLE_EXAMPLES OFF CACHE BOOL "Create or not a target for examples" )
SET( RELIABLE_EXAMPLES_PREFIX ${RELIABLE_PROJECT} CACHE STRING "Prefix for all reliable tests" )

# Install
SET( RELIABLE_ENABLE_INSTALL OFF CACHE BOOL "Create or not a target for install" )
SET( RELIABLE_INSTALL_PREFIX ${RELIABLE_PROJECT} CACHE STRING "Prefix for all reliable headers in the install folder" )

# CREATE PROJECT

PROJECT( ${RELIABLE_PROJECT} VERSION ${RELIABLE_VERSION} LANGUAGES C CXX )
SET_PROPERTY(GLOBAL PROPERTY USE_FOLDERS ON)
SET(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

# LOG OPTIONS

MESSAGE("---------------- RELIABLE.IO OPTIONS. ----------------")

#General
MESSAGE(STATUS "RELIABLE_PROJECT          : " ${RELIABLE_PROJECT})
MESSAGE(STATUS "RELIABLE_TARGET           : " ${RELIABLE_TARGET})
MESSAGE(STATUS "RELIABLE_VERSION          : " ${RELIABLE_VERSION})
MESSAGE(STATUS "RELIABLE_BUILD_SHARED     : " ${RELIABLE_BUILD_SHARED})
MESSAGE(STATUS "RELIABLE_FOLDER_PREFIX    : " ${RELIABLE_FOLDER_PREFIX})

# Tests
MESSAGE(STATUS "RELIABLE_ENABLE_TESTS     : " ${RELIABLE_ENABLE_TESTS})
IF(RELIABLE_ENABLE_TESTS)
MESSAGE(STATUS "RELIABLE_TESTS_PREFIX     : " ${RELIABLE_TESTS_PREFIX})
ENDIF(RELIABLE_ENABLE_TESTS)

# Examples
MESSAGE(STATUS "RELIABLE_ENABLE_EXAMPLES  : " ${RELIABLE_ENABLE_EXAMPLES})
IF(RELIABLE_ENABLE_EXAMPLES)
MESSAGE(STATUS "RELIABLE_EXAMPLES_PREFIX  : " ${RELIABLE_EXAMPLES_PREFIX})
ENDIF(RELIABLE_ENABLE_EXAMPLES)

# Install
MESSAGE(STATUS "RELIABLE_ENABLE_INSTALL   : " ${RELIABLE_ENABLE_INSTALL})
IF(RELIABLE_ENABLE_INSTALL)
MESSAGE(STATUS "RELIABLE_INSTALL_PREFIX   : " ${RELIABLE_INSTALL_PREFIX})
ENDIF(RELIABLE_ENABLE_INSTALL)

MESSAGE("---------------- DONE WITH OPTIONS. -----------------")
MESSAGE("                                                     ")

# ┌──────────────────────────────────────────────────────────────────┐
# │                          RELIABLE.IO                              │
# └──────────────────────────────────────────────────────────────────┘

# RELIABLE SOURCES
SET(RELIABLE_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/reliable.c)
SET(RELIABLE_HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/reliable.h)

# RELIABLE.IO TARGET
IF(RELIABLE_BUILD_SHARED)

    TARGET_COMPILE_DEFINITIONS(${RELIABLE_TARGET} PRIVATE "-DRELIABLE_DLL_EXPORT")
    ADD_LIBRARY(${RELIABLE_TARGET} SHARED ${RELIABLE_SOURCES} ${RELIABLE_HEADERS})

ELSE(RELIABLE_BUILD_SHARED)

    ADD_LIBRARY(${RELIABLE_TARGET} STATIC  ${RELIABLE_SOURCES} ${RELIABLE_HEADERS})
    TARGET_COMPILE_DEFINITIONS(${RELIABLE_TARGET} PUBLIC "-DRELIABLE_STATIC")

ENDIF(RELIABLE_BUILD_SHARED)

# Set include directory
TARGET_INCLUDE_DIRECTORIES(${RELIABLE_TARGET} PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}> $<INSTALL_INTERFACE:${RELIABLE_INSTALL_PREFIX}>)

# Avoid libreliable.io on darwin for example.
SET_TARGET_PROPERTIES(${RELIABLE_TARGET} PROPERTIES PREFIX "")

# Set correct folder in solution for superbuild (decorative)
SET_TARGET_PROPERTIES(${RELIABLE_TARGET} PROPERTIES FOLDER ${RELIABLE_FOLDER_PREFIX})

# Link to math library when not using msvc
IF(NOT MSVC)
    TARGET_LINK_LIBRARIES(${RELIABLE_TARGET} m)
ENDIF(NOT MSVC)

INCLUDE(CheckCCompilerFlag)

IF(MSVC)

    CHECK_C_COMPILER_FLAG("/fp:fast" IS_FFAST_MATH_AVAILABLE)
    IF(IS_FFAST_MATH_AVAILABLE)
        MESSAGE(STATUS "/fp:fast is available")
        SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /fp:fast")
    ELSE(IS_FFAST_MATH_AVAILABLE)
        MESSAGE(STATUS "/fp:fast isn't available")
    ENDIF(IS_FFAST_MATH_AVAILABLE)

    # Disable warning about vsprintf that is unsafe
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /wd4996")
ELSE(MSVC)

    CHECK_C_COMPILER_FLAG("-ffast-math" IS_FFAST_MATH_AVAILABLE)
    IF(IS_FFAST_MATH_AVAILABLE)
        MESSAGE(STATUS "ffast-math is available")
        SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -ffast-math")
    ELSE(IS_FFAST_MATH_AVAILABLE)
        MESSAGE(STATUS "ffast-math isn't available")
    ENDIF(IS_FFAST_MATH_AVAILABLE)

    CHECK_C_COMPILER_FLAG("-msse2" IS_SSE2_AVAILABLE)
    IF(IS_SSE2_AVAILABLE)
        MESSAGE(STATUS "sse2 is available")
        SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -msse2")
    ELSE(IS_SSE2_AVAILABLE)
        MESSAGE(STATUS "sse2 isn't available")
    ENDIF(IS_SSE2_AVAILABLE)

ENDIF(MSVC)

# ┌──────────────────────────────────────────────────────────────────┐
# │                           TESTS                                  │
# └──────────────────────────────────────────────────────────────────┘

IF(RELIABLE_ENABLE_TESTS)
    INCLUDE(${CMAKE_CURRENT_SOURCE_DIR}/CTestLists.txt)
ENDIF(RELIABLE_ENABLE_TESTS)

# ┌──────────────────────────────────────────────────────────────────┐
# │                         EXAMPLES                                 │
# └──────────────────────────────────────────────────────────────────┘

IF(RELIABLE_ENABLE_EXAMPLES)

    MACRO(RELIABLE_ADD_EXAMPLE EXAMPLE_NAME)

        SET(EXAMPLE_FULL_NAME ${RELIABLE_EXAMPLES_PREFIX}_${EXAMPLE_NAME})

        MESSAGE(STATUS "Add Example: ${EXAMPLE_FULL_NAME}")

        ADD_EXECUTABLE       (${EXAMPLE_FULL_NAME} ${EXAMPLE_NAME}.c)
        TARGET_LINK_LIBRARIES(${EXAMPLE_FULL_NAME} ${RELIABLE_TARGET})
        SET_TARGET_PROPERTIES(${EXAMPLE_FULL_NAME} PROPERTIES FOLDER ${RELIABLE_FOLDER_PREFIX}/Examples)

    ENDMACRO()

    RELIABLE_ADD_EXAMPLE(soak)
    RELIABLE_ADD_EXAMPLE(stats)
    RELIABLE_ADD_EXAMPLE(fuzz)

ENDIF(RELIABLE_ENABLE_EXAMPLES)

# ┌──────────────────────────────────────────────────────────────────┐
# │                         INSTALL                                  │
# └──────────────────────────────────────────────────────────────────┘

IF(RELIABLE_ENABLE_INSTALL)
    INCLUDE(${CMAKE_CURRENT_SOURCE_DIR}/CInstallLists.txt)
ENDIF(RELIABLE_ENABLE_INSTALL)
