#
# Copyright (C) 2012, 2013, 2014
# Alexander Lamaison <alexander.lamaison@gmail.com>
#
# This material is provided "as is", with absolutely no warranty
# expressed or implied. Any use is at your own risk. Permission to
# use or copy this software for any purpose is hereby granted without
# fee, provided the above notices are retained on all copies.
# Permission to modify the code and to distribute modified code is
# granted, provided the above notices are retained, and a notice that
# the code was modified is included with the above copyright notice.
#
# This file is part of Comet version 2.
# https://github.com/alamaison/comet
#

cmake_minimum_required(VERSION 3.0.0) # Interface libraries introduced in 3

set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})

include(HunterGate)
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.14.3.tar.gz"
    SHA1 "e05bc95f6d62b7c55188044e39e8be5afb1e8af4")

project(comet)

set(SOURCES
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/array.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/assert.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/atl_module.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/auto_buffer.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/bstr.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/calllog.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/cmd_line_parser.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/comet.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/common.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/config.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/cp.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/cptraits.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/currency.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/date.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/datetime.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/dispatch.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/enum.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/enum_common.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/enum_iterator.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/error.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/error_fwd.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/exe_server.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/functor.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/git.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/handle.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/handle_except.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/impqi.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/interface.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/invariant_lock.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/lw_lock.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/module.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/oleidl_comtypes.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/ptr.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/reference_count.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/registry.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/regkey.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/safearray.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/scope_guard.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/server.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/smart_enum.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/static_assert.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/stl.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/stl_enum.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/stream.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/threading.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/tlbinfo.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/tstring.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/type_traits.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/typelist.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/unittest.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/util.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/uuid.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/uuid_fwd.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/variant.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/variant_iterator.h)

add_custom_target(comet-src SOURCES ${SOURCES})
install(
  DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/comet
  DESTINATION include)

# Parse version

file(READ ${CMAKE_CURRENT_SOURCE_DIR}/include/comet/config.h CONFIG_CONTENTS)
string(
  REGEX REPLACE ".*#define COMET_MAJOR_VER[ \t]+([0-9]+).*" "\\1"
  COMET_VERSION_MAJOR "${CONFIG_CONTENTS}")
string(
  REGEX REPLACE ".*#define COMET_MINOR_VER[ \t]+([0-9]+).*" "\\1"
  COMET_VERSION_MINOR "${CONFIG_CONTENTS}")
string(
  REGEX REPLACE ".*#define COMET_PATCH_VER[ \t]+([0-9]+).*" "\\1"
  COMET_VERSION_PATCH "${CONFIG_CONTENTS}")

if(NOT COMET_VERSION_MAJOR MATCHES "^[0-9]+$" OR
   NOT COMET_VERSION_MINOR MATCHES "^[0-9]+$" OR
   NOT COMET_VERSION_PATCH MATCHES "^[0-9]+$")
  message(
    FATAL_ERROR
    "Unable to parse comet version from"
    "${CMAKE_CURRENT_SOURCE_DIR}/include/comet/config.h")
endif()

# Interface library for usage requireents

add_library(comet INTERFACE)
target_include_directories(comet
  INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>)

option(BUILD_TESTING "Build Comet test suite" ON)
if(BUILD_TESTING)
  enable_testing()
  add_subdirectory(tests)
endif()

# Package config

## During package installation, install CometConfig.cmake
install(TARGETS comet EXPORT CometConfig)
install(EXPORT CometConfig NAMESPACE Comet:: DESTINATION lib/cmake/Comet)

## Directly from build tree
# create CometConfig.cmake
export(TARGETS comet NAMESPACE Comet:: FILE CometConfig.cmake)
export(PACKAGE Comet) # register it

## Versioning

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/CometConfigVersion.cmake
  VERSION "${COMET_VERSION_MAJOR}.${COMET_VERSION_MINOR}.${COMET_VERSION_PATCH}"
  COMPATIBILITY SameMajorVersion)
install(
  FILES ${CMAKE_CURRENT_BINARY_DIR}/CometConfigVersion.cmake
  DESTINATION lib/cmake/Comet)
