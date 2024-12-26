cmake_minimum_required(VERSION 2.8.11 FATAL_ERROR)

project(QTAV)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

set(QTAV_MAJOR 1)
set(QTAV_MINOR 13)
set(QTAV_PATCH 0)
set(PROJECT_VERSION ${QTAV_MAJOR}.${QTAV_MINOR}.${QTAV_PATCH})
set(SO_VERSION ${QTAV_MAJOR})

set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

if(POLICY CMP0063) # visibility. since 3.3
  cmake_policy(SET CMP0063 NEW)
endif()
if(POLICY CMP0071)
  cmake_policy(SET CMP0071 NEW)
endif()
set(CMAKE_CXX_VISIBILITY_PRESET hidden) #use with -fdata-sections -ffunction-sections to reduce target size
set(CMAKE_VISIBILITY_INLINES_HIDDEN ON)

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

include(MacroUtils)

execute_process(COMMAND uname OUTPUT_VARIABLE uname)
if (uname MATCHES "^MSYS" OR uname MATCHES "^MINGW")
	execute_process(COMMAND cygpath -m / OUTPUT_VARIABLE MSYS_ROOT OUTPUT_STRIP_TRAILING_WHITESPACE)
	message(${MSYS_ROOT})
	set(FFMPEG_DIR ${MSYS_ROOT}mingw64 CACHE PATH "path to the ffmpeg build (support up to version 4)")
	set(CMAKE_CXX_FLAGS_RELEASE "-fpermissive")
	set(CMAKE_CXX_FLAGS_DEBUG "-fpermissive")
else()
	set(FFMPEG_DIR ${CMAKE_CURRENT_SOURCE_DIR}/3rd/ffmpeg CACHE PATH "path to the ffmpeg build (support up to version 4)")
endif()

set(FFMPEG_INCLUDE_DIRS     ${FFMPEG_DIR}/include/ffmpeg4.4)
set(AVCODEC_INCLUDE_DIRS    ${FFMPEG_DIR}/include/ffmpeg4.4/libavcodec)
set(AVDEVICE_INCLUDE_DIRS   ${FFMPEG_DIR}/include/ffmpeg4.4/libavdevice)
set(AVFILTER_INCLUDE_DIRS   ${FFMPEG_DIR}/include/ffmpeg4.4/libavfilter)
set(AVFORMAT_INCLUDE_DIRS   ${FFMPEG_DIR}/include/ffmpeg4.4/libavformat)
set(AVUTIL_INCLUDE_DIRS     ${FFMPEG_DIR}/include/ffmpeg4.4)
set(SWSCALE_INCLUDE_DIRS    ${FFMPEG_DIR}/include/ffmpeg4.4/libswscale)
set(SWRESAMPLE_INCLUDE_DIRS ${FFMPEG_DIR}/include/ffmpeg4.4/libswresample)

set(AVCODEC_LIBRARIES       ${FFMPEG_DIR}/lib/ffmpeg4.4/libavcodec.dll.a)
set(AVDEVICE_LIBRARIES      ${FFMPEG_DIR}/lib/ffmpeg4.4/libavdevice.dll.a)
set(AVFILTER_LIBRARIES      ${FFMPEG_DIR}/lib/ffmpeg4.4/libavfilter.dll.a)
set(AVFORMAT_LIBRARIES      ${FFMPEG_DIR}/lib/ffmpeg4.4/libavformat.dll.a)
set(AVUTIL_LIBRARIES        ${FFMPEG_DIR}/lib/ffmpeg4.4/libavutil.dll.a)
set(SWRESAMPLE_LIBRARIES    ${FFMPEG_DIR}/lib/ffmpeg4.4/libswresample.dll.a)
set(SWSCALE_LIBRARIES       ${FFMPEG_DIR}/lib/ffmpeg4.4/libswscale.dll.a)

find_package(FFmpeg REQUIRED COMPONENTS AVCODEC
                                        AVDEVICE
                                        AVFILTER
                                        AVFORMAT
                                        AVUTIL
                                        SWSCALE
                                        SWRESAMPLE
)

option(BUILD_EXAMPLES "Build examples" OFF)
option(BUILD_PLAYERS "Build players" OFF)
option(BUILD_TESTS "Build tests" OFF)
option(BUILD_QT6OPENGL "Build with Qt6 OpenGL module" ON)
option(BUILD_QML "Build QML interfaces" OFF)

list(APPEND CMAKE_FIND_ROOT_PATH ${QTDIR})

find_package(Qt6 COMPONENTS Core Widgets Sql Qml Core5Compat OpenGL REQUIRED)
get_target_property(qmake Qt6::qmake LOCATION)
execute_process(
    COMMAND ${qmake} -query QT_INSTALL_HEADERS
    OUTPUT_VARIABLE QT_INSTALL_HEADERS
    OUTPUT_STRIP_TRAILING_WHITESPACE
)
execute_process(
    COMMAND ${qmake} -query QT_INSTALL_LIBS
    OUTPUT_VARIABLE QT_INSTALL_LIBS
    OUTPUT_STRIP_TRAILING_WHITESPACE
)
execute_process(
    COMMAND ${qmake} -query QT_INSTALL_BINS
    OUTPUT_VARIABLE QT_INSTALL_BINS
    OUTPUT_STRIP_TRAILING_WHITESPACE
)
execute_process(
    COMMAND ${qmake} -query QT_INSTALL_QML
    OUTPUT_VARIABLE QT_INSTALL_QML
    OUTPUT_STRIP_TRAILING_WHITESPACE
)
execute_process(
    COMMAND ${qmake} -query QT_INSTALL_PREFIX
    OUTPUT_VARIABLE QT_INSTALL_PREFIX
    OUTPUT_STRIP_TRAILING_WHITESPACE
)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${QT6_CFLAGS}")

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

if(POLICY CMP0063) # visibility. since 3.3
  cmake_policy(SET CMP0063 NEW)
endif()
set(CMAKE_CXX_VISIBILITY_PRESET hidden) #use with -fdata-sections -ffunction-sections to reduce target size
set(CMAKE_VISIBILITY_INLINES_HIDDEN ON)

# Find includes in corresponding build directories
set(CMAKE_INCLUDE_CURRENT_DIR ON) # for .moc
# Instruct CMake to run moc automatically when needed.
set(CMAKE_AUTOMOC ON)
if(NOT CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  set(QTAV_INSTALL_HEADERS ${CMAKE_INSTALL_PREFIX}/include)
  set(QTAV_INSTALL_LIBS ${CMAKE_INSTALL_PREFIX}/lib)
  set(QTAV_INSTALL_BINS ${CMAKE_INSTALL_PREFIX}/bin)
  set(QTAV_INSTALL_QML ${QTAV_INSTALL_LIBS}/qml)
else()
  set(CMAKE_INSTALL_PREFIX ${QT_INSTALL_PREFIX} CACHE PATH "default install path" FORCE)
  set(QTAV_INSTALL_HEADERS ${QT_INSTALL_HEADERS})
  set(QTAV_INSTALL_LIBS ${QT_INSTALL_LIBS})
  set(QTAV_INSTALL_BINS ${QT_INSTALL_BINS})
  set(QTAV_INSTALL_QML ${QT_INSTALL_QML})
endif()

message(STATUS "Qt version: ${Qt6Core_VERSION_STRING}")
message(STATUS "Qt prefix: ${QT_INSTALL_PREFIX}")
message(STATUS "QtAV headers prefix: ${QTAV_INSTALL_HEADERS}")
message(STATUS "QtAV libs prefix: ${QTAV_INSTALL_LIBS}")
message(STATUS "QtAV bins prefix: ${QTAV_INSTALL_BINS}")
message(STATUS "QtAV qml prefix: ${QTAV_INSTALL_QML}")

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

#set(CMAKE_DEBUG_POSTFIX d)

install(FILES
  cmake/FindQtAV.cmake
  DESTINATION ${QTAV_INSTALL_LIBS}/cmake
)

if(CMAKE_GENERATOR MATCHES "Visual Studio.*")
  set(UCRT_INCLUDE $ENV{UniversalCRTSDKDir}Include\\$ENV{UCRTVersion}\\ucrt)
  set(VC_ARCH $ENV{Platform})
  if(NOT VC_ARCH)
    set(VC_ARCH x86)
  endif()
  set(UCRT_LIB $ENV{UniversalCRTSDKDir}Lib\\$ENV{UCRTVersion}\\ucrt\\${VC_ARCH})
  if(EXISTS ${UCRT_INCLUDE})
  message("VS project. ucrt: ${UCRT_LIB}")
    include_directories(${UCRT_INCLUDE})
    link_libraries(${CMAKE_LIBRARY_PATH_FLAG}${UCRT_LIB})
  endif()
endif()

add_subdirectory(src)

if(Qt6Widgets_FOUND)
  add_subdirectory(widgets)
endif()

if(BUILD_QML)
  find_package(Qt6 COMPONENTS Qml Quick)
  if(Qt6Qml_FOUND AND Qt6Quick_FOUND)
      #add_subdirectory(qml)
  endif()
endif()

if(BUILD_EXAMPLES OR BUILD_PLAYERS)
  add_subdirectory(examples)
endif()


