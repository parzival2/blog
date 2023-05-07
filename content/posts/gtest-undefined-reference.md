---
author: "Kalyan"
title: "Undefined reference to pthread_getspecific"
date: "2023-05-07"
description: "Linker error while using gtest"
tags: ["beaglebone", "c++", "cmake", "cross-compile", "gtest"]
ShowToc: true
---
## Undefined reference to `pthread_getspecific`
In the last two [post1](https://parzival2.github.io/blog/posts/beaglebone-cross-compile/) [post2](https://parzival2.github.io/blog/posts/crosstool-ng-beaglebone/) I tried to setup a working **cross-compilation** environment for **Beaglebone**. While I was trying out a test program, I noticed that I was not able to compile a program if I link it with `gtest`
I was getting a linking error which was not able to find references to `pthread_getspecific` and `pthread_setspecific`. 
If you are encountering an `undefined reference to pthread_getspecific` error while linking your C++ program with the Google Test (gtest) library, it is likely that your program is not linking with the `pthread` library.
### Linking with `pthread` to solve the error
The `pthread` library is a library that provides support for multithreaded programming in C and C++. It is a requirement for using the gtest library because gtest uses pthread for thread synchronization.

To fix the error, you should add the `-pthread` flag to your linker options. This will link your program with the pthread library and resolve the undefined reference error.
```cmake
target_link_libraries(${project_name} PUBLIC fmt::fmt GTest::gtest GTest::gtest_main -pthread)
```
### Compiling *gtest* with proper flags
When you provide `-DBUILD_SHARED_LIBS=ON` option while running CMake, it specifies that you want to build the project as a shared library. This means that the code will be compiled into a shared object file (.so) that can be dynamically linked by other programs at runtime.
So the above problem can also be solved by providing the `-DBUILD_SHARED_LIBS` while doing *cmake*
```bash
cd build
cmake -DBUILD_SHARED_LIBS=ON ..
make 
sudo make install
```
