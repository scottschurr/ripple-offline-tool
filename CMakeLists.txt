#########################################################
# CMAKE_C_COMPILER and CMAKE_CXX_COMPILER must be defined
# before the project statement; However, the project
# statement will clear CMAKE_BUILD_TYPE. CACHE variables,
# along with the order of this code, are used to work
# around these constraints.
#
# Don't put any code above or in this block, unless it
# has similar constraints.
cmake_minimum_required(VERSION 3.1.0)
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/extras/ripple-libpp/extras/rippled/Builds/CMake")
include(CMakeFuncs)
set(openssl_min 1.0.1)
parse_target()
project(ripple-offline-tool)
#########################################################

if (nonunity)
  set(target "" CACHE STRING "Build target. Nounity not supported" FORCE)
  message(FATAL_ERROR "Nonunity builds not supported")
endif()

setup_build_cache()

############################################################

add_subdirectory(extras/ripple-libpp/src/unity)

############################################################

determine_build_type()

check_gcc4_abi()

############################################################

include_directories(
  src/
  SYSTEM
  extras/ripple-libpp/extras/rippled/src
  extras/ripple-libpp/extras/rippled/src/beast/include
  extras/ripple-libpp/extras/rippled/src/beast/extras)

special_build_flags()

############################################################

use_boost(
  filesystem
  program_options
  regex
  system
  thread)

use_pthread()

use_openssl(${openssl_min})

setup_build_boilerplate()

############################################################

prepend(app_src
  src/
  RippleKey.cpp
  Serialize.cpp
  OfflineTool.cpp
  test/RippleKey_test.cpp
  test/Serialize_test.cpp
  test/OfflineTool_test.cpp)

############################################################

if (WIN32 OR is_xcode)
  # Rippled sources
  foreach(curdir
      beast/core
      beast/hash
      beast/utility
      basics
      core
      crypto
      json
      protocol)
    file(GLOB_RECURSE cursrcs extras/ripple-libpp/extras/rippled/src/ripple/${curdir}/*.h
      extras/ripple-libpp/extras/rippled/src/ripple/${curdir}/*.cpp)
    list(APPEND rippled_src "${cursrcs}")
    list(APPEND non_unity_srcs "${cursrcs}")
  endforeach()

  file(GLOB_RECURSE all_headers src/*.hpp extras/*.hpp src/*.h)
  list(APPEND rippled_src "${all_headers}")

  # Properties
  set_property(
    SOURCE ${non_unity_srcs}
    APPEND
    PROPERTY HEADER_FILE_ONLY
    true)
  set_property(
    SOURCE ${all_headers}
    APPEND
    PROPERTY HEADER_FILE_ONLY
    true)
    # Doesn't work
    # $<OR:$<CONFIG:Debug>,$<CONFIG:Release>>)
endif()

############################################################

if (NOT is_msvc)
  set(no_unused_w -Wno-unused-function)
else()
  unset(no_unused_w)
endif()

############################################################

if (WIN32 OR is_xcode)
  group_sources(src)
endif()

add_executable(ripple-offline-tool ${app_src} $<TARGET_OBJECTS:ripplelibpp>)

set_startup_project(ripple-offline-tool)

target_link_libraries(ripple-offline-tool
  ${OPENSSL_LIBRARIES} ${SANITIZER_LIBRARIES})

link_common_libraries(ripple-offline-tool)
