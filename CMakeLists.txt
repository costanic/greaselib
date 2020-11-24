# 3.13.4: current version on debian-10
cmake_minimum_required(VERSION 3.13.4)
project (grease LANGUAGES C CXX)

set (GREASE_VERSION_MAJOR 1)
set (GREASE_VERSION_MINOR 0)
set (GREASE_VERSION_PATCH 0)
set (GREASE_VERSION_STRING "${GREASE_VERSION_MAJOR}.${GREASE_VERSION_MINOR}.${GREASE_VERSION_PATCH}")

#
# Build library
#
set(SRCS_C
  grease_client.c
  local_strdup.c
  )

set(SRCS_CXX
  error-common.cc
  logger.cc
  standalone_test_logsink.cc
  grease_lib.cc
  )

set(LIBUV_DIR ${CMAKE_CURRENT_SOURCE_DIR}/deps/libuv-v1.10.1)
set(LIBUV_HEADER_FILES
  ${LIBUV_DIR}/include/uv-errno.h
  ${LIBUV_DIR}/include/uv-linux.h
  ${LIBUV_DIR}/include/uv-threadpool.h
  ${LIBUV_DIR}/include/uv-version.h
  ${LIBUV_DIR}/include/uv.h
  )

set(INSTALL_HEADER_FILES
  grease_lib.h
  grease_client.h
  grease_common_tags.h
  deps/build/include/google/tcmalloc.h
  ${LIBUV_HEADER_FILES}
  )

add_custom_target(version.h
  COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/update_version.sh
  )

add_custom_target(depslibs
  COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/deps/install-deps.sh
  )

add_library(grease SHARED ${SRCS_C} ${SRCS_CXX})
add_dependencies(grease version.h depslibs)
target_compile_features(grease PUBLIC cxx_std_11)
target_compile_options(grease PUBLIC -Wall -Wextra -Wno-error=format-security)
target_compile_definitions(grease PUBLIC GREASE_LIB)

target_include_directories(grease PUBLIC
  $<BUILD_INTERFACE:${LIBUV_DIR}/include>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/deps/build/include>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/deps/twlib/include>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/deps/gperftools-2.4/src>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
  )

#
# Install rules
#
include(GNUInstallDirs)

set_target_properties(grease PROPERTIES
  PUBLIC_HEADER "${INSTALL_HEADER_FILES}"
  VERSION ${GREASE_VERSION_STRING}
  SOVERSION ${GREASE_VERSION_MAJOR}
  )

install(
  TARGETS grease
  EXPORT greaseTargets
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/grease
  RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
  LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
  ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
  )

#
# Add tests
#
#include(CTest) # This module defines BUILD_TESTING option
#if(BUILD_TESTING)
#  add_subdirectory(tests)
#endif()

#
# ConfigVersion file
# Generate 'greaseConfig.cmake', 'greaseConfigVersion.cmake' and 'greaseTargets.cmake'
# so other projects can include require libgrease from their own cmake files.
#
include(CMakePackageConfigHelpers)

write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/greaseConfigVersion.cmake
  VERSION ${GREASE_VERSION_STRING}
  COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(
  ${CMAKE_CURRENT_LIST_DIR}/CMake/greaseConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/greaseConfig.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/grease/cmake
  )

install(
  FILES
    ${CMAKE_CURRENT_BINARY_DIR}/greaseConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/greaseConfigVersion.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/grease/cmake
  )

# Configure 'greaseTargets.cmake' for a build tree
export(TARGETS grease
	FILE ${CMAKE_CURRENT_BINARY_DIR}/greaseTargets.cmake
  )

# Configure and install 'greaseConfig.cmake' for an install tree
#set(CONFIG_DIR_CONFIG ${INSTALL_CMAKE_DIR})
#set(install_config_file ${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/greaseConfig.cmake )
#configure_package_config_file(
#  greaseConfig.cmake.in
#  ${install_config_file}
#  INSTALL_DESTINATION ${CMAKE_INSTALL_PREFIX}/${INSTALL_CMAKE_DIR}
#  PATH_VARS CONFIG_DIR_CONFIG
#  NO_CHECK_REQUIRED_COMPONENTS_MACRO
#  )
#install(
#  FILES ${install_config_file}
#  DESTINATION ${INSTALL_CMAKE_DIR} COMPONENT Development
#  )

# Configure and install 'greaseTargets.cmake' for an install tree
#install(EXPORT greaseTargets
#  FILE greaseTargets.cmake
#  DESTINATION ${CMAKE_INSTALL_DIR}/cmake
#  COMPONENT Development
#  )

