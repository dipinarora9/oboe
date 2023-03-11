#
# Copyright 2018 The Android Open Source Project
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
cmake_minimum_required(VERSION 3.4.1)
project(liveEffect LANGUAGES C CXX)

get_filename_component(SAMPLE_ROOT_DIR
    ${CMAKE_CURRENT_SOURCE_DIR}/../../../.. ABSOLUTE)

### INCLUDE OBOE LIBRARY ###
set (OBOE_DIR ${SAMPLE_ROOT_DIR}/..)
add_subdirectory(${OBOE_DIR} ./oboe-bin)

add_library(liveEffect
    SHARED
        LiveEffectEngine.cpp
        FullDuplexStream.cpp
        jni_bridge.cpp
        ${SAMPLE_ROOT_DIR}/debug-utils/trace.cpp

        audio/AAssetDataSource.cpp

        audio/logging.h
        audio/GameConstants.h
        audio/Player.cpp)

set (TARGET_LIBS log android atomic oboe)

if(${USE_FFMPEG})

    MESSAGE(STATUS "Using FFmpeg extractor")

    add_definitions(-DUSE_FFMPEG=1)
    target_sources( liveEffect PRIVATE audio/FFMpegExtractor.cpp )

    # Add the local path to FFmpeg, you can use the ${ANDROID_ABI} variable to specify the ABI name
    # e.g. /Users/donturner/Code/ffmpeg/build/${ANDROID_ABI}
    set(FFMPEG_DIR "/path/to/ffmpeg")

    include_directories(liveEffect ${FFMPEG_DIR}/include)

    add_library( avformat SHARED IMPORTED)
    set_target_properties(avformat PROPERTIES IMPORTED_LOCATION
            ${FFMPEG_DIR}/lib/libavformat.so)
    add_library( avutil SHARED IMPORTED)
    set_target_properties(avutil PROPERTIES IMPORTED_LOCATION
            ${FFMPEG_DIR}/lib/libavutil.so)
    add_library( avcodec SHARED IMPORTED)
    set_target_properties(avcodec PROPERTIES IMPORTED_LOCATION
            ${FFMPEG_DIR}/lib/libavcodec.so)
    add_library( swresample SHARED IMPORTED)
    set_target_properties(swresample PROPERTIES IMPORTED_LOCATION
            ${FFMPEG_DIR}/lib/libswresample.so)
    set (TARGET_LIBS ${TARGET_LIBS} avformat avutil avcodec swresample)

else()
    MESSAGE(STATUS "Using NDK media extractor")
    add_definitions(-DUSE_FFMPEG=0)
    target_sources( liveEffect PRIVATE audio/NDKExtractor.cpp )
    set (TARGET_LIBS ${TARGET_LIBS} mediandk)
endif()


target_include_directories(liveEffect
    PRIVATE
        ${SAMPLE_ROOT_DIR}/debug-utils
        ${OBOE_DIR}/include)

include_directories (${OBOE_DIR}/samples)

target_link_libraries(liveEffect ${TARGET_LIBS} )

# Enable optimization flags: if having problems with source level debugging,
# disable -Ofast ( and debug ), re-enable it after done debugging.
target_compile_options(liveEffect PRIVATE -std=c++17 -Wall -Werror "$<$<CONFIG:RELEASE>:-Ofast>")

