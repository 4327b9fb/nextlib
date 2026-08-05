# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html

# Sets the minimum version of CMake required to build the native library.

cmake_minimum_required(VERSION 3.22.1)

# Declares and names the project.

project("media3ext")

set(ffmpeg_dir ${CMAKE_SOURCE_DIR}/../../../../ffmpeg/output)
set(ffmpeg_libs ${ffmpeg_dir}/lib/${ANDROID_ABI})

include_directories(${ffmpeg_dir}/include/${ANDROID_ABI})

# FFmpeg static libraries
set(
        # List variable name
        ffmpeg_libs_names
        # Values in the list
        avutil avcodec swresample swscale)

foreach (ffmpeg_lib_name ${ffmpeg_libs_names})
    add_library(
            ${ffmpeg_lib_name}
            STATIC
            IMPORTED)
    set_target_properties(
            ${ffmpeg_lib_name}
            PROPERTIES
            IMPORTED_LOCATION
            ${ffmpeg_libs}/lib${ffmpeg_lib_name}.a)
endforeach ()

# Dependency static libraries (libvpx for VP8/VP9, mbedtls for TLS)
set(
        dep_libs_names
        vpx mbedtls mbedx509 mbedcrypto)

foreach (dep_lib_name ${dep_libs_names})
    add_library(
            ${dep_lib_name}
            STATIC
            IMPORTED)
    set_target_properties(
            ${dep_lib_name}
            PROPERTIES
            IMPORTED_LOCATION
            ${ffmpeg_libs}/lib${dep_lib_name}.a)
endforeach ()

add_library(${CMAKE_PROJECT_NAME} SHARED
        # List C/C++ source files with relative paths to this CMakeLists.txt.
        ffmain.cpp
        ffcommon.cpp
        ffaudio.cpp
        ffvideo.cpp)

set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,-z,max-page-size=16384")

target_link_libraries(${CMAKE_PROJECT_NAME}
        # List libraries link to the target library
        log
        android
        z
        m
        ${ffmpeg_libs_names}
        ${dep_libs_names})