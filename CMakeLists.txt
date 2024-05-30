cmake_minimum_required(VERSION 3.27)
project(quickjs)

include(CheckCCompilerFlag)
include(GNUInstallDirs)

set(CMAKE_C_VISIBILITY_PRESET hidden)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS ON)
set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 20)


macro(xcheck_add_c_compiler_flag FLAG)
    string(REPLACE "-" "" FLAG_NO_HYPHEN ${FLAG})
    check_c_compiler_flag(${FLAG} COMPILER_SUPPORTS_${FLAG_NO_HYPHEN})
    if(COMPILER_SUPPORTS_${FLAG_NO_HYPHEN})
        add_compile_options(${FLAG})
    endif()
endmacro()

xcheck_add_c_compiler_flag(-Wall)
if(NOT MSVC AND NOT IOS)
    xcheck_add_c_compiler_flag(-Werror)
    xcheck_add_c_compiler_flag(-Wextra)
endif()
xcheck_add_c_compiler_flag(-Wno-implicit-fallthrough)
xcheck_add_c_compiler_flag(-Wno-sign-compare)
xcheck_add_c_compiler_flag(-Wno-missing-field-initializers)
xcheck_add_c_compiler_flag(-Wno-unused-parameter)
xcheck_add_c_compiler_flag(-Wno-unused-variable)
xcheck_add_c_compiler_flag(-Wno-unused-but-set-variable)
xcheck_add_c_compiler_flag(-Wno-array-bounds)
xcheck_add_c_compiler_flag(-Wno-format-truncation)
xcheck_add_c_compiler_flag(-funsigned-char)

# ClangCL is command line compatible with MSVC, so 'MSVC' is set.
if(MSVC)
    xcheck_add_c_compiler_flag(-Wno-unsafe-buffer-usage)
    xcheck_add_c_compiler_flag(-Wno-sign-conversion)
    xcheck_add_c_compiler_flag(-Wno-nonportable-system-include-path)
    xcheck_add_c_compiler_flag(-Wno-implicit-int-conversion)
    xcheck_add_c_compiler_flag(-Wno-shorten-64-to-32)
    xcheck_add_c_compiler_flag(-Wno-reserved-macro-identifier)
    xcheck_add_c_compiler_flag(-Wno-reserved-identifier)
    xcheck_add_c_compiler_flag(-Wdeprecated-declarations)
    xcheck_add_c_compiler_flag(/experimental:c11atomics)
endif()

if(CMAKE_SYSTEM_NAME STREQUAL "WASI")
    add_compile_definitions(
        _WASI_EMULATED_PROCESS_CLOCKS
        _WASI_EMULATED_SIGNAL
    )
    add_link_options(
        -lwasi-emulated-process-clocks
        -lwasi-emulated-signal
    )
endif()

set(quickjs_sources
        quickjs.h
        quickjs-libc.h
        quickjs.c
        libregexp.c
        libunicode.c
        libbf.c
        cutils.c
        quickjs-libc.c
)



#if (LINUX)
#    target_link_libraries(${PROJECT_NAME} PUBLIC dl)
#endif ()

#if (MSVC)
#    set(MSVC_JS_STRICT_NAN_BOXING "JS_STRICT_NAN_BOXING")
#endif ()

#target_compile_definitions(${PROJECT_NAME}
#        PRIVATE
#        CONFIG_VERSION="nekoray"
#        _GNU_SOURCE
#        CONFIG_BIGNUM
#        ${MSVC_JS_STRICT_NAN_BOXING}
#        )


list(APPEND qjs_defines _GNU_SOURCE)
if(WIN32)
    list(APPEND qjs_defines WIN32_LEAN_AND_MEAN _WIN32_WINNT=0x0602)
endif()
list(APPEND qjs_libs qjs ${CMAKE_DL_LIBS})
find_package(Threads)
if(NOT CMAKE_SYSTEM_NAME STREQUAL "WASI")
    list(APPEND qjs_libs ${CMAKE_THREAD_LIBS_INIT})
endif()
if(NOT MSVC)
    list(APPEND qjs_libs m)
endif()

add_library(${PROJECT_NAME} ${quickjs_sources})
target_compile_definitions(${PROJECT_NAME} PRIVATE ${qjs_defines})

target_include_directories(${PROJECT_NAME} PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>  
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)
