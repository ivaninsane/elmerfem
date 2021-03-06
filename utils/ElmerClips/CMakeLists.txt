PROJECT(ElmerClips CXX C)
CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

SET(ElmerClipsSTANDALONE FALSE CACHE BOOL "Bundle necessary libraries in windows for standalone installation.")
MARK_AS_ADVANCED(ElmerClipsSTANDALONE)



SET(FFmpegROOT "${CMAKE_CURRENT_BINARY_DIR}/ffmpeg-build" CACHE PATH "Path to FFmpeg installation")

FIND_LIBRARY(AVCODEC_LIBRARY NAMES avcodec HINTS ${FFmpegROOT}/lib $ENV{FFmpegROOT}/lib)
FIND_LIBRARY(AVUTIL_LIBRARY NAMES avutil HINTS ${FFmpegROOT}/lib $ENV{FFmpegROOT}/lib)
FIND_LIBRARY(SWSCALE_LIBRARY NAMES swscale HINTS ${FFmpegROOT}/lib $ENV{FFmpegROOT}/lib)
FIND_PATH(AVCODEC_INCLUDE_DIR NAMES libavcodec/avcodec.h PATHS ${FFmpegROOT}/include $ENV{FFmpegROOT}/include)


SET(FFmpeg_INCLUDE_DIR ${AVCODEC_INCLUDE_DIR})
SET(CodecID "AVCodecID")

MARK_AS_ADVANCED(FFmpegROOT AVCODEC_LIBRARY AVCODEC_INCLUDE_DIR SWSCALE_LIBRARY AVUTIL_LIBRARY FFmpeg_INCLUDE_DIR)

SET(CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH};${CMAKE_CURRENT_SOURCE_DIR}/cmake/Modules")
FIND_PACKAGE(Qt4 COMPONENTS QtOpenGL QtXml QtScript QtGui QtCore REQUIRED)
FIND_PACKAGE(Threads REQUIRED)

MESSAGE(STATUS "------------------------------------------------")
MESSAGE(STATUS "  [ElmerClips] Qt4:                 " ${Qt4_FOUND})
MESSAGE(STATUS "  [ElmerClips] Qt4_LIBRARIES:       " ${Qt4_LIBRARY})
MESSAGE(STATUS "  [ElmerClips] AVCODEC_LIBRARY      " ${AVCODEC_LIBRARY})
MESSAGE(STATUS "  [ElmerClips] AVUTIL_LIBRARY       " ${AVUTIL_LIBRARY})
MESSAGE(STATUS "  [ElmerClips] SWSCALE_LIBRARY      " ${SWSCALE_LIBRARY})
MESSAGE(STATUS "  [ElmerClips] FFmpeg_INCLUDE_DIR   " ${FFmpeg_INCLUDE_DIR})
MESSAGE(STATUS "------------------------------------------------")

INCLUDE_DIRECTORIES(${FFmpeg_INCLUDE_DIR} src/ ${CMAKE_CURRENT_BINARY_DIR})
CONFIGURE_FILE(config.h.cmake config.h)

SET(CMAKE_INCLUDE_CURRENT_DIR ON)
SET(CMAKE_AUTOMOC ON)
SET(CMAKE_AUTORCC ON)
SET(CMAKE_AUTOUIC ON)

SET(HEADERS src/preview.h src/encoder.h)
SET(SOURCES src/main.cpp src/preview.cpp src/encoder.cpp)
SET(RESOURCES ElmerClips.qrc)
SET(RC_FILE ElmerClips.rc)


QT4_ADD_RESOURCES(UI_RESOURCES ElmerClips.qrc)
MESSAGE(WARNING "QT_USE_FILE: ${QT_USE_FILE}")
INCLUDE(${QT_USE_FILE})
ADD_DEFINITIONS(${QT_DEFINITIONS})


IF(MINGW AND WIN32)
  SET(CMAKE_RC_COMPILER_INIT windres)
  ENABLE_LANGUAGE(RC)
  SET(CMAKE_RC_COMPILE_OBJECT
    "<CMAKE_RC_COMPILER> <FLAGS> -O coff <DEFINES> -i <SOURCE> -o <OBJECT>")
  SET(SOURCES ${SOURCES} ElmerClips.rc)
ENDIF()

ADD_EXECUTABLE(ElmerClips WIN32 ${SOURCES} ${UI_RESOURCES})

TARGET_LINK_LIBRARIES(ElmerClips ${QT_LIBRARIES} ${AVCODEC_LIBRARY}
  ${AVUTIL_LIBRARY} ${SWSCALE_LIBRARY}  ${CMAKE_THREAD_LIBS_INIT})
INSTALL(TARGETS ElmerClips RUNTIME DESTINATION "bin" COMPONENT "elmergui")

IF(ElmerClipsSTANDALONE)
  IF(WIN32)
    INCLUDE(cmake/windows_bundle.cmake)
  ENDIF(WIN32)
ENDIF(ElmerClipsSTANDALONE)
