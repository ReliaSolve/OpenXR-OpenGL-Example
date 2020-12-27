cmake_minimum_required(VERSION 3.0)
project(OpenXROpenGLExample)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

include_directories("${CMAKE_CURRENT_SOURCE_DIR}/gfxwrapper")
include_directories("${CMAKE_CURRENT_SOURCE_DIR}/sdk-files")

find_package(Threads REQUIRED)

# Tell that we are using OpenGL.
set(OpenGL_GL_PREFERENCE GLVND)
find_package(OpenGL REQUIRED)
add_definitions(-DXR_USE_GRAPHICS_API_OPENGL)

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

add_library(openxr-all-supported INTERFACE)
if(BUILD_WITH_XLIB_HEADERS)
    target_compile_definitions(openxr-all-supported INTERFACE XR_USE_PLATFORM_XLIB)
endif()

if(BUILD_WITH_XCB_HEADERS)
    target_compile_definitions(openxr-all-supported INTERFACE XR_USE_PLATFORM_XCB)
endif()

if(BUILD_WITH_WAYLAND_HEADERS)
    target_compile_definitions(openxr-all-supported INTERFACE XR_USE_PLATFORM_WAYLAND)
endif()

set(OpenXROpenGLExample_SRC
  main.cpp
)

add_executable(OpenXROpenGLExample ${OpenXROpenGLExample_SRC})
target_link_libraries(OpenXROpenGLExample 
  openxr_loader
  openxr-gfxwrapper
  openxr-all-supported
  ${CMAKE_THREAD_LIBS_INIT}
)

