cmake_minimum_required(VERSION 3.4.1)

set(PATCHES_PATH "${CMAKE_CURRENT_LIST_DIR}/patches")
set(ASSETS_PATH "${CMAKE_CURRENT_LIST_DIR}/src/main/assets")
set(BUILD_TOOLS_PATH "${CMAKE_CURRENT_LIST_DIR}/build/tools")
set(LLVM_OBFUSCATOR_PATH "${BUILD_TOOLS_PATH}/llvm_obfuscator")
set(EXECUTABLE_OUTPUT_PATH "${ASSETS_PATH}/${ANDROID_ABI}")

if (CMAKE_HOST_WIN32)
    set(EXE_SUFFIX ".exe")
elseif(CMAKE_HOST_UNIX)
    set(EXE_SUFFIX "")
else()
    message(FATAL_ERROR "Unsupported platform!")
endif()

execute_process(
    COMMAND ${CMAKE_COMMAND}
        ${CMAKE_CURRENT_LIST_DIR}/llvm_obfuscator
        -DLLVM_OBFUSCATOR_PATH=${LLVM_OBFUSCATOR_PATH}
        -DPATCHES_PATH=${PATCHES_PATH}
        -DBUILD_TOOLS_PATH=${BUILD_TOOLS_PATH}
        WORKING_DIRECTORY ${LLVM_OBFUSCATOR_PATH}/llvm_obfuscator
    COMMAND ${CMAKE_COMMAND}
        --build .
        WORKING_DIRECTORY ${LLVM_OBFUSCATOR_PATH}/llvm_obfuscator
    )

add_library(native-lib SHARED
    src/main/cpp/native-lib.cpp
    )

set(CMAKE_C_COMPILER ${LLVM_OBFUSCATOR_PATH}/bin/clang${EXE_SUFFIX})
set(CMAKE_CXX_COMPILER ${LLVM_OBFUSCATOR_PATH}/bin/clang++${EXE_SUFFIX})

# Bogus control flow is really unstable
# set(OBFUSCATOR_FLAGS "-mllvm -sub -mllvm -fla -mllvm -bcf")
set(OBFUSCATOR_FLAGS "-mllvm -sub -mllvm -fla")

string(APPEND CMAKE_C_FLAGS " ${OBFUSCATOR_FLAGS}")
string(APPEND CMAKE_CXX_FLAGS " ${OBFUSCATOR_FLAGS}")
