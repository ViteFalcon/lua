###############################################################################
#             This file is part of the CMake build system for Lua             #
###############################################################################
cmake_minimum_required(VERSION 2.8)
set(CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS TRUE)
cmake_policy(SET CMP0003 NEW)

#------------------------------------------------------------------------------
# Set up the basic build environment
#------------------------------------------------------------------------------
# Just debug / release since that's all that's all we need
set (CMAKE_CONFIGURATION_TYPES "Debug;Release" CACHE STRING "" FORCE)
set (CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)
set (CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)
set (CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin)

#------------------------------------------------------------------------------
# Use relative paths
# This is mostly to reduce path size for command-line limits on windows
#------------------------------------------------------------------------------
if (WIN32)
    # This seems to break Xcode projects so definitely don't enable on Apple builds
    set(CMAKE_USE_RELATIVE_PATHS true)
    set(CMAKE_SUPPRESS_REGENERATION true)
endif()

project(lua)

#------------------------------------------------------------------------------
# Add a suffix to debug binaries for MSVC builds
#------------------------------------------------------------------------------
if (MSVC)
    # Create debug libraries with _d postfix
    set(CMAKE_DEBUG_POSTFIX "_d")
endif()

#------------------------------------------------------------------------------
# Disable some warnings in MSVC and also add a suffix to debug binaries
#------------------------------------------------------------------------------
if (MSVC)
    add_definitions(-D_CRT_SECURE_NO_DEPRECATE)
    add_definitions(-D_CRT_NONSTDC_NO_DEPRECATE)
    add_definitions(-D_SCL_SECURE_NO_WARNINGS)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
endif()

#------------------------------------------------------------------------------
# Set some vars to help us
#------------------------------------------------------------------------------
set(LUA_VERSION_SUFFIX "52")
set(LUA_SRC_DIR "${CMAKE_CURRENT_SOURCE_DIR}/src")
set(LUA_INC_DIR "${CMAKE_CURRENT_SOURCE_DIR}/include")

#------------------------------------------------------------------------------
# Copy files to 'include' directory
#------------------------------------------------------------------------------
set(LUA_INCLUDE_FILES
    ${LUA_SRC_DIR}/lauxlib.h
    ${LUA_SRC_DIR}/lua.h
    ${LUA_SRC_DIR}/luaconf.h
    ${LUA_SRC_DIR}/lualib.h
)
file(COPY ${LUA_INCLUDE_FILES} DESTINATION ${LUA_INC_DIR})

#------------------------------------------------------------------------------
# Set list of 'core' files here
#------------------------------------------------------------------------------
set(LUA_CORE_FILES
    ${LUA_SRC_DIR}/lapi.c
    ${LUA_SRC_DIR}/lcode.c
    ${LUA_SRC_DIR}/lctype.c
    ${LUA_SRC_DIR}/ldebug.c
    ${LUA_SRC_DIR}/ldo.c
    ${LUA_SRC_DIR}/ldump.c
    ${LUA_SRC_DIR}/lfunc.c
    ${LUA_SRC_DIR}/lgc.c
    ${LUA_SRC_DIR}/llex.c
    ${LUA_SRC_DIR}/lmem.c
    ${LUA_SRC_DIR}/lobject.c
    ${LUA_SRC_DIR}/lopcodes.c
    ${LUA_SRC_DIR}/lparser.c
    ${LUA_SRC_DIR}/lstate.c
    ${LUA_SRC_DIR}/lstring.c
    ${LUA_SRC_DIR}/ltable.c
    ${LUA_SRC_DIR}/ltm.c
    ${LUA_SRC_DIR}/lundump.c
    ${LUA_SRC_DIR}/lvm.c
    ${LUA_SRC_DIR}/lzio.c
)


#------------------------------------------------------------------------------
# Add project for shared-library (luaxx.so/dll)
#------------------------------------------------------------------------------
# Required to 'export' API
set(LUA_SHARED_LIB_TARGET "lua${LUA_VERSION_SUFFIX}")
set(LUA_SHARED_LIB_FILES
    ${LUA_CORE_FILES}
    ${LUA_SRC_DIR}/lauxlib.c
    ${LUA_SRC_DIR}/lbaselib.c
    ${LUA_SRC_DIR}/lbitlib.c
    ${LUA_SRC_DIR}/lcorolib.c
    ${LUA_SRC_DIR}/ldblib.c
    ${LUA_SRC_DIR}/liolib.c
    ${LUA_SRC_DIR}/lmathlib.c
    ${LUA_SRC_DIR}/loadlib.c
    ${LUA_SRC_DIR}/loslib.c
    ${LUA_SRC_DIR}/lstrlib.c
    ${LUA_SRC_DIR}/ltablib.c
    ${LUA_SRC_DIR}/linit.c
)
ADD_LIBRARY(${LUA_SHARED_LIB_TARGET} SHARED ${LUA_SHARED_LIB_FILES})


#------------------------------------------------------------------------------
# Add project for interpreter (lua)
#------------------------------------------------------------------------------
set(LUA_INTERPRETER_TARGET "lua")
set(LUA_INTERPRETER_FILES
    ${LUA_SRC_DIR}/lua.c
)
ADD_EXECUTABLE(${LUA_INTERPRETER_TARGET} ${LUA_INTERPRETER_FILES})


#------------------------------------------------------------------------------
# List of source files for compiler (luac)
#------------------------------------------------------------------------------
set(LUA_COMPILER_TARGET "luac")
set(LUA_INTERPRETER_FILES
    ${LUA_CORE_FILES}
    ${LUA_SRC_DIR}/luac.c
)
ADD_EXECUTABLE(${LUA_COMPILER_TARGET} ${LUA_INTERPRETER_FILES})

#------------------------------------------------------------------------------
# Set project dependencies
#------------------------------------------------------------------------------
add_dependencies(${LUA_INTERPRETER_TARGET} ${LUA_SHARED_LIB_TARGET})
add_dependencies(${LUA_COMPILER_TARGET} ${LUA_SHARED_LIB_TARGET})

#------------------------------------------------------------------------------
# Set pre-processor definition for shared-library
#------------------------------------------------------------------------------
set_target_properties(${LUA_SHARED_LIB_TARGET} PROPERTIES COMPILE_FLAGS "-DLUA_BUILD_AS_DLL")

#------------------------------------------------------------------------------
# Setup projects to link to shared-library
#------------------------------------------------------------------------------
target_link_libraries(${LUA_INTERPRETER_TARGET} ${LUA_SHARED_LIB_TARGET})
target_link_libraries(${LUA_COMPILER_TARGET} ${LUA_SHARED_LIB_TARGET})