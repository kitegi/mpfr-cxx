cmake_minimum_required(VERSION 3.16)
project(mpfr-cxx CXX)

get_directory_property(has_parent PARENT_DIRECTORY)

if(has_parent)
  set(top_level FALSE)
else()
  set(top_level TRUE)
endif()

set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake" ${CMAKE_MODULE_PATH})

if(top_level)
  set(CMAKE_CXX_STANDARD 17)
  set(CMAKE_CXX_EXTENSIONS OFF)
endif()

include(cmake/compiler_warnings.cmake)
include(cmake/sanitizers.cmake)
include(cmake/conan.cmake)
include(cmake/static_analyzers.cmake)

if(MPFR_CXX_BUILD_DOCS)
  add_subdirectory("docs")
  return()
endif()

add_library(mpfr-cxx INTERFACE)
target_include_directories(
  mpfr-cxx INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/include
)

if(top_level AND ENABLE_TESTING)
  set(CONAN_REQUIRES
      # MIT License
      fmt/7.1.2 backward-cpp/1.5
      # LGPL v3.0
      mpfr/4.1.0
  )
  set(CONAN_OPTIONS)
  run_conan()

  target_link_libraries(mpfr-cxx INTERFACE CONAN_PKG::fmt CONAN_PKG::mpfr)

  add_library(project_options INTERFACE)
  add_library(project_warnings INTERFACE)
  set_project_warnings(project_warnings)
  enable_sanitizers(project_options)
  target_link_libraries(mpfr-cxx INTERFACE project_options project_warnings)

  enable_testing()
  add_subdirectory(test)
  add_subdirectory(benchmark)
endif()
