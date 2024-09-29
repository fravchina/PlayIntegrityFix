cmake_minimum_required(VERSION 3.30.3)

project("playintegrityfix")

find_package(cxx REQUIRED CONFIG)

link_libraries(cxx::cxx)

find_program(UNZIP_EXECUTABLE unzip REQUIRED)

file(
  DOWNLOAD
  https://raw.githubusercontent.com/nlohmann/json/v3.11.3/single_include/nlohmann/json.hpp
  "${CMAKE_CURRENT_BINARY_DIR}/json_include/json.hpp"
  EXPECTED_HASH SHA256=9BEA4C8066EF4A1C206B2BE5A36302F8926F7FDC6087AF5D20B417D0CF103EA6
)

file(
  DOWNLOAD
  "https://github.com/bytedance/android-inline-hook/archive/5ccd9152b447afc99a139fa58efc6368cd3b7476.zip"
  "${CMAKE_CURRENT_BINARY_DIR}/android-inline-hook.zip"
  EXPECTED_HASH SHA256=FDDB1413F4D049EC41B182D089314117298229D7D80AD683F0295ABA999CB020
)

if(NOT EXISTS "${CMAKE_CURRENT_BINARY_DIR}/android-inline-hook-src/android-inline-hook-5ccd9152b447afc99a139fa58efc6368cd3b7476")
  execute_process(
    COMMAND_ERROR_IS_FATAL ANY
    COMMAND
    ${CMAKE_COMMAND} -E make_directory "${CMAKE_CURRENT_BINARY_DIR}/android-inline-hook-src"
    COMMAND
    ${UNZIP_EXECUTABLE} "${CMAKE_CURRENT_BINARY_DIR}/android-inline-hook.zip" -d "${CMAKE_CURRENT_BINARY_DIR}/android-inline-hook-src"
  )
endif()


if (${ANDROID_ABI} STREQUAL "arm64-v8a")
    set(ARCH "arm64")
elseif (${ANDROID_ABI} STREQUAL "armeabi-v7a")
    set(ARCH "arm")
endif ()

file(GLOB SHADOWHOOK_SRC
        "${CMAKE_CURRENT_BINARY_DIR}/android-inline-hook-src/android-inline-hook-5ccd9152b447afc99a139fa58efc6368cd3b7476/shadowhook/src/main/cpp/*.c"
        "${CMAKE_CURRENT_BINARY_DIR}/android-inline-hook-src/android-inline-hook-5ccd9152b447afc99a139fa58efc6368cd3b7476/shadowhook/src/main/cpp/arch/${ARCH}/*.c"
        "${CMAKE_CURRENT_BINARY_DIR}/android-inline-hook-src/android-inline-hook-5ccd9152b447afc99a139fa58efc6368cd3b7476/shadowhook/src/main/cpp/common/*.c"
        "${CMAKE_CURRENT_BINARY_DIR}/android-inline-hook-src/android-inline-hook-5ccd9152b447afc99a139fa58efc6368cd3b7476/shadowhook/src/main/cpp/third_party/xdl/*.c"
)

add_library(${CMAKE_PROJECT_NAME} SHARED
        main.cpp
        ${SHADOWHOOK_SRC}
)

target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE
        "${CMAKE_CURRENT_BINARY_DIR}/json_include"
        "${CMAKE_CURRENT_BINARY_DIR}/android-inline-hook-src/android-inline-hook-5ccd9152b447afc99a139fa58efc6368cd3b7476/shadowhook/src/main/cpp"
        "${CMAKE_CURRENT_BINARY_DIR}/android-inline-hook-src/android-inline-hook-5ccd9152b447afc99a139fa58efc6368cd3b7476/shadowhook/src/main/cpp/arch/${ARCH}"
        "${CMAKE_CURRENT_BINARY_DIR}/android-inline-hook-src/android-inline-hook-5ccd9152b447afc99a139fa58efc6368cd3b7476/shadowhook/src/main/cpp/include"
        "${CMAKE_CURRENT_BINARY_DIR}/android-inline-hook-src/android-inline-hook-5ccd9152b447afc99a139fa58efc6368cd3b7476/shadowhook/src/main/cpp/common"
        "${CMAKE_CURRENT_BINARY_DIR}/android-inline-hook-src/android-inline-hook-5ccd9152b447afc99a139fa58efc6368cd3b7476/shadowhook/src/main/cpp/third_party/bsd"
        "${CMAKE_CURRENT_BINARY_DIR}/android-inline-hook-src/android-inline-hook-5ccd9152b447afc99a139fa58efc6368cd3b7476/shadowhook/src/main/cpp/third_party/lss"
        "${CMAKE_CURRENT_BINARY_DIR}/android-inline-hook-src/android-inline-hook-5ccd9152b447afc99a139fa58efc6368cd3b7476/shadowhook/src/main/cpp/third_party/xdl"
)

target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE log)
