cmake_minimum_required(VERSION 3.4.1)

project(llvm_obfuscator)

if (NOT BUILD_TOOLS_PATH)
    message(FATAL_ERROR "Please define BUILD_TOOLS_PATH!")
endif()

if (NOT LLVM_OBFUSCATOR_PATH)
    message(FATAL_ERROR "Please define LLVM_OBFUSCATOR_PATH!")
endif()

if (NOT PATCHES_PATH)
    message(FATAL_ERROR "Please define PATCHES_PATH!")
endif()

set(LLVM_OBFUSCATOR_LOCK_PATH ${LLVM_OBFUSCATOR_PATH}/.llvm_obfuscator_lock)

include (ExternalProject)

if (EXISTS ${LLVM_OBFUSCATOR_LOCK_PATH})
    message(STATUS "    Using prebuilt LLVM-obfuscator.")
else()
    message(STATUS "    Building LLVM-obfuscator from source...")

    ExternalProject_add(llvm_obfuscator_host
        PREFIX "${BUILD_TOOLS_PATH}"
        GIT_REPOSITORY https://github.com/obfuscator-llvm/obfuscator.git
        GIT_TAG llvm-3.6.1
        PATCH_COMMAND patch -p1 < "${PATCHES_PATH}/ndk_driver_patch.patch"
        COMMAND patch -p1 < "${PATCHES_PATH}/windows_random.patch"
        CONFIGURE_COMMAND ${CMAKE_COMMAND} <SOURCE_DIR>
            -DCMAKE_OBJECT_PATH_MAX=4096
            -DCMAKE_INSTALL_PREFIX:String=${LLVM_OBFUSCATOR_PATH}
            -DLLVM_TARGETS_TO_BUILD:String=X86$<SEMICOLON>ARM
            -DCMAKE_BUILD_TYPE:String=Release
        BUILD_COMMAND ${CMAKE_COMMAND} --build . --config Release
        INSTALL_COMMAND ${CMAKE_COMMAND} --build . --target install --config Release
        )

    execute_process(
        COMMAND ${CMAKE_COMMAND} -E touch ${LLVM_OBFUSCATOR_LOCK_PATH})
endif()
