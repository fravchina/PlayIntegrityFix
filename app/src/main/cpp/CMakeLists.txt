cmake_minimum_required(VERSION 3.22.1)

project(playintegrityfix)

find_package(cxx REQUIRED CONFIG)

find_program(UNZIP_EXECUTABLE unzip REQUIRED)
file(
  DOWNLOAD
  https://raw.githubusercontent.com/nlohmann/json/v3.11.3/single_include/nlohmann/json.hpp
  "${CMAKE_CURRENT_BINARY_DIR}/json_include/json.hpp"
  EXPECTED_HASH SHA256=9BEA4C8066EF4A1C206B2BE5A36302F8926F7FDC6087AF5D20B417D0CF103EA6
)

file(
  DOWNLOAD
  https://github.com/jmpews/Dobby/archive/0932d69c320e786672361ab53825ba8f4245e9d3.zip
  "${CMAKE_CURRENT_BINARY_DIR}/Dobby.zip"
  EXPECTED_HASH SHA256=A08CD9449ED39246E9A2B493A55FCC4F843A8EBE79ECEF7747A02180B58B21DB
)

if(NOT EXISTS "${CMAKE_CURRENT_BINARY_DIR}/dobby-src/Dobby-0932d69c320e786672361ab53825ba8f4245e9d3")
  execute_process(
    COMMAND_ERROR_IS_FATAL ANY
    COMMAND
    ${CMAKE_COMMAND} -E make_directory "${CMAKE_CURRENT_BINARY_DIR}/dobby-src"
    COMMAND
    ${UNZIP_EXECUTABLE} "${CMAKE_CURRENT_BINARY_DIR}/Dobby.zip" -d "${CMAKE_CURRENT_BINARY_DIR}/dobby-src"
  )
endif()

link_libraries(log)

link_libraries(cxx::cxx)

add_library(${CMAKE_PROJECT_NAME} SHARED main.cpp)

target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE "${CMAKE_CURRENT_BINARY_DIR}/json_include")

set(CMAKE_POLICY_DEFAULT_CMP0077 NEW)
set(Plugin.Android.BionicLinkerUtil ON)
add_subdirectory(
  "${CMAKE_CURRENT_BINARY_DIR}/dobby-src/Dobby-0932d69c320e786672361ab53825ba8f4245e9d3"
  "${CMAKE_CURRENT_BINARY_DIR}/dobby-bin"
  EXCLUDE_FROM_ALL
)

target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE dobby_static)