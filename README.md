# OpenXR-OpenGL-Example

<!--
Copyright (c) 2017-2020 The Kronos Group Inc
Copyright (c) 2020 ReliaSolve LLC
-->

This repository contains an OpenGL example program that links against the OpenXR loader.

It is a pared-down version of the hello-xr sample that comes with the OpenXR DSK Source
repostiory at <https://github.com/KhronosGroup/OpenXR-SDK-Source/> that removes the
class hierarchies to generalize the application.  The goal is to make a base application
from which to build other OpenGL applications that can run on HMDs on Windows, Linux,
and Mac.

The project should build using CMake on Windows, Mac, and Linux.  It requires the
OpenXR loader to have been installed for to compile and it requires an OpenXR runtime
that supports OpenGL to be running at runtime.

As of 12/27/2020 it was compiling and running on Ubuntu 20.04 against the Monado runtime,
with the display in a stereo view on a window.  It also compiles and runs on Windows, but
I don't have an OpenXR runtime that supports OpenGL to test it on.

Files:
- main.cpp: The bulk of the application.  The OpenXR and OpenGL classes from the OpenXR-SDK-Source
repository were brought into the main source file and pared down; their m_ objects converted
to g_ objects.
- gfxwrapper: Contains the gfxwrapper library from the OpenXR-SDK-Source repository.  Used
to find and link against OpenGL on various platforms.
- sdk-files: Additional helper header files brought over from the OpenXR-SDK-Source
repository.
