# 3.13.4: current version on debian-10
cmake_minimum_required(VERSION 3.13.4)
project (grease LANGUAGES C CXX)

set (GREASE_VERSION_MAJOR 0)
set (GREASE_VERSION_MINOR 1)
set (GREASE_VERSION_PATCH 0)
set (GREASE_VERSION_STRING "${GREASE_VERSION_MAJOR}.${GREASE_VERSION_MINOR}.${GREASE_VERSION_PATCH}")

option(BUILD_SHARED_LIBS "Build libgrease as a shared library" OFF)
option(GREASE_STATIC_LIB_NAME "base name of static library output" grease)

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

add_library(grease ${SRCS_C} ${SRCS_CXX})

if(NOT BUILD_SHARED_LIBS)
  set_target_properties(grease
	  PROPERTIES OUTPUT_NAME ${GREASE_STATIC_LIB_NAME}
  )
endif()

target_include_directories(grease PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>
  $<INSTALL_INTERFACE:${INSTALL_INCLUDE_DIR}>
  )

#
# Install rules
#
include(GNUInstallDirs)

install(
  FILES
    include/grease_lib.h
  DESTINATION include COMPONENT Development
  )

install(
  TARGETS grease
  EXPORT greaseTargets
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
# Generate 'greaseConfig.cmake', 'greaseConfigVersion.cmake' and 'greaseTargets.cmake'
#
#include(CMakePackageConfigHelpers)

# Configure 'greaseConfig.cmake' for a build tree
#set(CONFIG_DIR_CONFIG ${PROJECT_BINARY_DIR})
#set(config_file ${PROJECT_BINARY_DIR}/greaseConfig.cmake)
#configure_package_config_file(
#  greaseConfig.cmake.in
#  ${config_file}
#  INSTALL_DESTINATION ${PROJECT_BINARY_DIR}
#  PATH_VARS CONFIG_DIR_CONFIG
#  NO_CHECK_REQUIRED_COMPONENTS_MACRO
#  )

# Configure 'greaseTargets.cmake' for a build tree
#export(TARGETS grease
#  FILE ${PROJECT_BINARY_DIR}/greaseTargets.cmake
#  )

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
#  DESTINATION ${INSTALL_CMAKE_DIR}
#  COMPONENT Development
#  )
#
## Configure 'greaseConfigVersion.cmake' for a build tree
#set(config_version_file ${PROJECT_BINARY_DIR}/greaseConfigVersion.cmake)
#write_basic_package_version_file(
#    ${config_version_file}
#    VERSION ${YAML_VERSION_STRING}
#    COMPATIBILITY AnyNewerVersion
#)
## ... and install for an install tree
#install(
#  FILES ${config_version_file}
#  DESTINATION ${INSTALL_CMAKE_DIR} COMPONENT Development
#  )
#
