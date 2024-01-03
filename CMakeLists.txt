cmake_minimum_required(VERSION 3.2.0)

project(OpusClipTimeline)

option(OPUSCLIP_DECODE "Decode support" ON)
option(OPUSCLIP_ENCODE "Encode support" ON)

# =================================
# ========= Env Setting ===========
# =================================
if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
        set(LINUX TRUE)
endif()

if(CMAKE_SYSTEM_PROCESSOR STREQUAL "arm64" AND CMAKE_SYSTEM_NAME STREQUAL "Darwin")
        set(ARM64_APPLE TRUE)
endif()

# print
message(STATUS "ARM64_APPLE + Darwin: ${ARM64_APPLE}")
message(STATUS "LINUX: ${LINUX}")

# =================================
# ======= Compiler Setting ========
# =================================
if(LINUX)
        set(CMAKE_C_FLAGS "-L /lib64 -fPIC -ffast-math -lc -lpthread -Wno-deprecated-declarations -O2")
        set(CMAKE_CXX_FLAGS "-L /lib64 -std=c++2b -fPIC -ffast-math -lc -lpthread -Wno-deprecated-declarations -O2")
elseif(ARM64_APPLE)
        set(CMAKE_C_FLAGS "-fPIC -ffast-math -lc -lpthread -Wno-deprecated-declarations -O2")
        set(CMAKE_CXX_FLAGS "-std=c++2b -fPIC -ffast-math -lc -lpthread -Wno-deprecated-declarations -O2")
endif()

# ======= Preset Directory ========
set(ROOT_DIR ${CMAKE_SOURCE_DIR}/.)
set(SRC_DIR ${ROOT_DIR}/src)
set(VCPKG_DIR ${ROOT_DIR}/vcpkg_installed/arm64-osx)
set(NODEJS_DIR ${ROOT_DIR}/source/node)
set(NODE_ADDON_DIR ${ROOT_DIR}/source/node-addon-api)
set(UNIT_TEST_DIR ${SRC_DIR}/unitTest)
set(EXAMPLE_DIR ${SRC_DIR}/example)

# HarfBuzz: A text shaping library
# dependency of Pango
find_package(PkgConfig REQUIRED)
pkg_check_modules(HARFBUZZ REQUIRED harfbuzz)

# ======= Include Directory ========
include_directories(
        ${VCPKG_DIR}/include
        ${VCPKG_DIR}/include/pango-1.0
        ${VCPKG_DIR}/include/glib-2.0
        ${VCPKG_DIR}/include/glib-2.0/gobject
        ${VCPKG_DIR}/include/glib-2.0/glib
        ${VCPKG_DIR}/include/glib-2.0/gio
        ${VCPKG_DIR}/include/glib-2.0/gmodule

        ${VCPKG_DIR}/lib/glib-2.0/include
        # ./vcpkg_installed/arm64-osx/lib/glib-2.0/include

        ${NODE_ADDON_DIR}
        ${NODEJS_DIR}/src
        ${SRC_DIR}
        ${SRC_DIR}/backend
        ${SRC_DIR}/bmp
)

include_directories(
        ${VCPKG_DIR}/lib

        # ${ROOT_DIR}/lib/include/pango
)

set(OPUSCLIP_UTIL_SRCS
        ${SRC_DIR}/util/CJSON.cpp
        ${SRC_DIR}/util/OpusClipLog.cpp
        ${SRC_DIR}/util/MsgThread.cpp
        ${SRC_DIR}/util/OpusMatrix.cpp
        ${SRC_DIR}/util/OpusRandom.cpp
        ${SRC_DIR}/util/OpusTimer.cpp
        ${SRC_DIR}/util/OpusVector.cpp
        ${SRC_DIR}/util/OpusUtil.cpp
        ${SRC_DIR}/util/OpusContext.cpp
        ${SRC_DIR}/util/MediaParseUtil.cpp
        ${SRC_DIR}/util/OpusGlobalInfo.cpp
        ${SRC_DIR}/util/OpusData.cpp
        ${SRC_DIR}/util/ColorUtil.cpp
        ${SRC_DIR}/util/PangoUtil.cpp
)

set(OPUSCLIP_EFFECTS_SRCS
        ${SRC_DIR}/glEngine/OpusBaseEffect.cpp
        ${SRC_DIR}/glEngine/OpusBaseEffectGL.cpp
        ${SRC_DIR}/glEngine/OpusEffectMgr.cpp
        ${SRC_DIR}/glEngine/OpusEffectPlugin.cpp
        ${SRC_DIR}/glEngine/OpusOpenGL.cpp
        ${SRC_DIR}/glEngine/OpusTransCropEffect.cpp
        ${SRC_DIR}/glEngine/OpusTransformEffect.cpp
        ${SRC_DIR}/glEngine/OpusBlendEffect.cpp
        ${SRC_DIR}/glEngine/OpusBaseBlendEffect.cpp
        ${SRC_DIR}/glEngine/OpusRgbaToYuv444Effect.cpp
        ${SRC_DIR}/glEngine/OpusGaussianblurEffect.cpp
)

set(OPUSCLIP_TIMELINE_SRCS
        ${SRC_DIR}/editor/AVDataStatistic.cpp
        ${SRC_DIR}/editor/AVEditAVSegInfo.cpp
        ${SRC_DIR}/editor/AVEditBaseTrack.cpp
        ${SRC_DIR}/editor/AVEditHelper.cpp
        ${SRC_DIR}/editor/AVEditSegAudio.cpp
        ${SRC_DIR}/editor/AVEditSegBase.cpp
        ${SRC_DIR}/editor/AVEditSegVideo.cpp
        ${SRC_DIR}/editor/AVEditSegSilence.cpp
        ${SRC_DIR}/editor/AVEditTimeline.cpp
        ${SRC_DIR}/editor/AVEditVisualSegUtil.cpp
        ${SRC_DIR}/editor/DecoderAutoMgr.cpp
        ${SRC_DIR}/editor/AVEngineClient.cpp
        ${SRC_DIR}/editor/AVEditSegText.cpp
        ${SRC_DIR}/editor/AVEditTextInfo.cpp
        ${SRC_DIR}/timeline/OpusAsset.cpp
        ${SRC_DIR}/timeline/OpusAssetMgr.cpp
        ${SRC_DIR}/timeline/OpusBezier.cpp
        ${SRC_DIR}/timeline/OpusDecoderMgr.cpp
        ${SRC_DIR}/timeline/OpusEffect.cpp
        ${SRC_DIR}/timeline/OpusKeyFrame.cpp
        ${SRC_DIR}/timeline/OpusLayer.cpp
        ${SRC_DIR}/timeline/OpusMediaAsset.cpp
        ${SRC_DIR}/timeline/OpusTextAsset.cpp
        ${SRC_DIR}/timeline/OpusTimelineImp.cpp
        ${SRC_DIR}/timeline/OpusTimelineInfo.cpp
)

if(OPUSCLIP_DECODE)
        add_definitions(-DAE_ENABLE_DECODE)
        set(OPUSCLIP_DECODE_SRCS
                ${SRC_DIR}/media/decoder/soft/FFDecoder.cpp
                ${SRC_DIR}/media/decoder/AudioDecHandler.cpp
                ${SRC_DIR}/media/decoder/VideoDecHandler.cpp
                ${SRC_DIR}/media/decoder/AVHandler.cpp
                ${SRC_DIR}/media/decoder/DecoderFactory.cpp
                ${SRC_DIR}/media/decoder/ImageDecoder.cpp
                ${SRC_DIR}/media/decoder/GifDecoder.cpp
        )
endif()

if(OPUSCLIP_ENCODE)
        add_definitions(-DAE_ENABLE_ENCODE)
        set(OPUSCLIP_ENCODE_SRCS
                ${SRC_DIR}/media/encoder/linux/FFEncoder.cpp
                ${SRC_DIR}/compile/linux/LinuxEncodeMux.cpp
        )
endif()

if(LINUX)
        set(LINUX_LIB_DIR ${ROOT_DIR}/lib/linux)
endif()

add_library(OpusClipTimeline-Static

        # Sets the library as a static library.
        STATIC

        ${OPUSCLIP_UTIL_SRCS}
        ${OPUSCLIP_EFFECTS_SRCS}
        ${OPUSCLIP_TIMELINE_SRCS}
        ${OPUSCLIP_DECODE_SRCS}
        ${OPUSCLIP_ENCODE_SRCS}
)

target_link_libraries(OpusClipTimeline-Static
        ${LINUX_LIB_DIR}/libffmpeg_all.a # download manually
        ${LINUX_LIB_DIR}/libfreeimage.so
        ${LINUX_LIB_DIR}/libpango-1.0.so
        ${LINUX_LIB_DIR}/libpangocairo-1.0.so
        ${LINUX_LIB_DIR}/libpangoft2-1.0.so
        libcairo.so
        libfontconfig.so
        libgobject-2.0.so
        libglib-2.0.so
        libnuma.so
        GLESv2
        EGL
        z
        c)

# UnitTest: cairosimple_test
add_executable(StrokeMock ${UNIT_TEST_DIR}/stroke_test.cpp)
target_link_libraries(StrokeMock OpusClipTimeline-Static)
