# OpenXR-OpenGL-Example

<!--
Copyright (c) 2017-2020 The Kronos Group Inc
Copyright (c) 2020-2023 ReliaSolve LLC
-->

This repository contains an OpenGL example program that links against the OpenXR loader.

It is a pared-down version of the hello-xr sample that comes with the OpenXR DSK Source
repository at <https://github.com/KhronosGroup/OpenXR-SDK-Source/> that removes the
class hierarchies to generalize the application.  The goal is to make a base application
from which to build other OpenGL applications that can run on HMDs on Windows, Linux,
and Mac.

The project should build using CMake on Windows, Mac, and Linux.  **It requires the
OpenXR SDK/loader to have been installed for it to compile and it requires an OpenXR runtime
that supports OpenGL to be running at runtime.**  After installing the SDK on Windows,
CMake finds my OpenXR configuration at "C:/Program Files (x86)/OPENXR/cmake".

As of 7/25/2023 it was compiling and running on Ubuntu 20.04 against the Monado runtime,
with the display in a stereo view on a window.  It also compiles and runs on Windows, using
the HTC Vive OpenXR runtime. **Note:** On systems that have both an integrated GPU and
and nVidia GPU, you must use nVidia settings to prefer only the nVidia GPU, otherwise
the rendering context does not get set correctly and the display may be incorrect or crash
the compositor.

Files:
- main.cpp: The bulk of the application.  The OpenXR and OpenGL classes from the OpenXR-SDK-Source
repository were brought into the main source file and pared down; their m_ objects converted
to g_ objects.
- gfxwrapper: Contains the gfxwrapper library from the OpenXR-SDK-Source repository.  Used
to find and link against OpenGL on various platforms.
- sdk-files: Additional helper header files brought over from the OpenXR-SDK-Source
repository.
