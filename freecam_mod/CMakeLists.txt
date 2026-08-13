cmake_minimum_required(VERSION 3.22)

project(freecam LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

if(NOT DEFINED PRELOADER_SDK)
    set(PRELOADER_SDK "${CMAKE_CURRENT_SOURCE_DIR}/../preloader-sdk")
endif()

add_library(freecam SHARED
    freecam.cpp
)

target_include_directories(freecam PRIVATE
    ${PRELOADER_SDK}/src
    ${PRELOADER_SDK}/include
)

find_library(log-lib log REQUIRED)
find_library(egl-lib EGL REQUIRED)

if(ANDROID_ABI STREQUAL "arm64-v8a")
    set(GLOSSHOOK_LIB
        ${PRELOADER_SDK}/lib/ARM64/libGlossHook.a
    )
elseif(ANDROID_ABI STREQUAL "armeabi-v7a")
    set(GLOSSHOOK_LIB
        ${PRELOADER_SDK}/lib/ARM/libGlossHook.a
    )
else()
    message(FATAL_ERROR "Unsupported ABI: ${ANDROID_ABI}")
endif()

if(NOT EXISTS "${GLOSSHOOK_LIB}")
    message(FATAL_ERROR
        "GlossHook not found: ${GLOSSHOOK_LIB}"
    )
endif()

target_link_libraries(freecam PRIVATE
    ${GLOSSHOOK_LIB}
    ${log-lib}
    ${egl-lib}
)

set_target_properties(freecam PROPERTIES
    OUTPUT_NAME "libfreecam"
    PREFIX ""
)
