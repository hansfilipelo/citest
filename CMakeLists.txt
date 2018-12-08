cmake_minimum_required(VERSION 3.0)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
set(CMAKE_EXPORT_COMPILE_COMMANDS 1)

project(citesting)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/src)
aux_source_directory(${CMAKE_CURRENT_SOURCE_DIR}/src CITESTING_SOURCES)

add_executable(
  citesting
  ${CITESTING_SOURCES}
  )
