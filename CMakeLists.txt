cmake_minimum_required(VERSION 3.0)
project(OpenXROpenGLExample)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

include_directories("${CMAKE_CURRENT_SOURCE_DIR}/gfxwrapper")
include_directories("${CMAKE_CURRENT_SOURCE_DIR}/sdk-files")

find_package(OpenXR REQUIRED)
find_package(Threads REQUIRED)

# Tell that we are using OpenGL.
set(OpenGL_GL_PREFERENCE GLVND)
find_package(OpenGL REQUIRED)
find_package(JSONCPP REQUIRED)
add_definitions(-DXR_USE_GRAPHICS_API_OPENGL)

# @todo We would like to use the OpenXR::openxr_loader target and have it import its
# include directories and libraries, but it has conflicts with the JSONCPP installer
# on Windows.  So we do this horrible hack to bring it in.
# Add include directories for standard Windows install
if(WIN32)
  include_directories("C:/Program Files (x86)/OPENXR/include")
  link_directories("C:/Program Files (x86)/OPENXR/lib")
endif()

# This is a helper graphics wrapping library that keeps track of what system include and library
# files are needed.  We have to make it a target before we include presentation so that it
# will have the system dependencies added to it.
add_library(openxr-gfxwrapper STATIC gfxwrapper/gfxwrapper_opengl.c gfxwrapper/gfxwrapper_opengl.h)
target_include_directories(openxr-gfxwrapper PUBLIC ${PROJECT_SOURCE_DIR}/external/include)
if(OPENGL_FOUND)
    if(TARGET OpenGL::OpenGL)
        target_link_libraries(openxr-gfxwrapper PUBLIC OpenGL::OpenGL)
    elseif(TARGET OpenGL::GL)
        target_link_libraries(openxr-gfxwrapper PUBLIC OpenGL::GL)
    else()
        target_link_libraries(openxr-gfxwrapper PUBLIC ${OPENGL_LIBRARIES})
    endif()
endif()

# Determine the presentation backend for Linux systems.
# Use an include because the code is pretty big.
if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    include(presentation)
endif()

# Several files use these compile-time OS switches
if(WIN32)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
    add_definitions(-DXR_USE_PLATFORM_WIN32)
    add_definitions(-DXR_OS_WINDOWS)
    add_definitions(-DNOMINMAX)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    add_definitions(-DXR_OS_LINUX)
endif()

# Several files use these compile-time platform switches
if(WIN32)
    add_definitions(-DXR_USE_PLATFORM_WIN32)
elseif(PRESENTATION_BACKEND MATCHES "xlib")
    add_definitions(-DOS_LINUX_XLIB)
elseif(PRESENTATION_BACKEND MATCHES "xcb")
    add_definitions(-DOS_LINUX_XCB)
elseif(PRESENTATION_BACKEND MATCHES "wayland")
    add_definitions(-DOS_LINUX_WAYLAND)
endif()

set(OpenXROpenGLExample_SRC
  main.cpp
)

add_executable(OpenXROpenGLExample ${OpenXROpenGLExample_SRC})
# @todo Remove this when we can link to OpenXR::openxr_loader and have it pull in JSONCPP
# and its other libraries correctly.
if (WIN32)
  target_link_libraries(OpenXROpenGLExample openxr_loader advapi32 pathcch)
else()
  target_link_libraries(OpenXROpenGLExample OpenCR::openxr_loader)
endif()
target_link_libraries(OpenXROpenGLExample 
  jsoncpp_lib
  openxr-gfxwrapper
  OpenXR::openxr-all-supported
  ${CMAKE_THREAD_LIBS_INIT}
)
