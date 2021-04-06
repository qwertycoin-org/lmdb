cmake_minimum_required(VERSION 3.12)

find_program(CCACHE_PROGRAM ccache)
if(CCACHE_PROGRAM)
    message(STATUS "LMDB-ADV: Found ccache package... Activating...")
    set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE "${CCACHE_PROGRAM}")
endif()

set(LIB_MAJOR_VERSION "0")
set(LIB_MINOR_VERSION "9")
set(LIB_PATCH_VERSION "70")
set(LIB_VERSION_STRING "${LIB_MAJOR_VERSION}.${LIB_MINOR_VERSION}.${LIB_PATCH_VERSION}")

cmake_policy(SET CMP0048 NEW)
project(lmdb-adv VERSION "${LIB_VERSION_STRING}" LANGUAGES CXX)

## This section describes our general CMake setup options
set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_SKIP_INSTALL_RULES OFF FORCE)
set(CMAKE_SKIP_PACKAGE_ALL_DEPENDENCY ON FORCE)
set(CMAKE_SUPPRESS_REGENERATION ON)
set(CMAKE_POLICY_WARNING_CMP0048 OFF)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

# Enable c++17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

## We only build static binaries -- this is left here for our dependencies
set(STATIC ON CACHE BOOL FORCE "Link libraries statically? Forced to ON")
add_definitions(-DSTATICLIB)

add_subdirectory(lmdb)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/lmdb)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/ext)

if(MSVC)
    add_definitions("/EHsc -std:c++17 /DWIN32_LEAN_AND_MEAN /O2 /wd4267 /wd4996 /wd4244 /wd4333 /wd4146 /wd4172")
endif()

if(NOT MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17 -O3 -Wno-return-local-addr")
endif()

set(LMDB_SRC
        lmdb-adv.cpp
        )

add_library(lmdb-adv STATIC ${LMDB_SRC})
target_link_libraries(lmdb-adv lmdb)
target_include_directories(lmdb-adv PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

if(WIN32)
    target_link_libraries(lmdb-adv ws2_32)
endif()

set_property(TARGET lmdb-adv PROPERTY C_STANDARD 90)
set_property(TARGET lmdb-adv PROPERTY SOVERSION 1)

foreach(flag_var
        CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE
        CMAKE_C_FLAGS_MINSIZEREL CMAKE_C_FLAGS_RELWITHDEBINFO
        CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE
        CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO)
    string(REGEX REPLACE "/RTC[^ ]*" "" ${flag_var} "${${flag_var}}")
    string(REGEX REPLACE "/Od" "" ${flag_var} "${${flag_var}}")
endforeach(flag_var)
