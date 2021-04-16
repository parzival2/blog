---
author: "Kalyan"
title: "Cross compile C++ project for beaglebone"
date: "2021-04-12"
description: "Cross compile C++ project for beaglebone"
tags: ["beaglebone", "c++", "cmake", "cross-compile", "libgpiod"]
ShowToc: true
---

## Cross compiling a C++ project for Beaglebone

I am thinking of getting into Embedded Linux, so I have ordered a Beaglebone. I don't want to use Python to program it as I want to do some real-time processing with it. For that, I have to figure out how to compile C++ programs on the host PC instead of doing it on the Beaglebone as it has very little ram. 

Initially, I wanted to do the cross-compilation using `QEMU` but quickly gave up the idea as there is no out-of-the-box `QEMU` system for Beaglebone. Raspberry pi has a nice `QEMU` system that runs out of the box. 
I will revisit the idea of using `QEMU` once I got accompanied on the procedure on building Linux kernel. 
Everything new has to start simple, so, a simple that I have in mind is to compile and run a LED blinking application using [libgpiod](https://git.kernel.org/pub/scm/libs/libgpiod/libgpiod.git/) library which is the preferred method for driving GPIOs from user-space. 

I built and installed the `libgpiod` library on the `Beaglebone` itself. I can also install it using `sudo apt install libgpiod` but I don't want to do that as it was a very old version. 

### Compiling and installing libgpiod

The `libgpiod` library can be built and installed using the following procedure.

```bash
git clone https://git.kernel.org/pub/scm/libs/libgpiod/libgpiod.git/
```

You will checkout the master branch. Feel free to checkout a release if you are more comfortable like that using

```bash
git checkout remotes/origin/v1.6.x
```

Configure and compile it using the following commands

```bash
./autogen.sh --enable-tools=yes --enable-bindings-cxx=yes --prefix=/usr/local
make
sudo make install
```

If you don't want to use C++, then remove the `--enable-bindings-cxx=yes` in the above command lines.

The `libgpiod` comes with a bunch of useful command line tools for driving and testing GPIOs.

All the commands take offset along with the gpiochip number that, that GPIO refers to.

E.g. If you want to drive P9-12 pin on the `Beaglebone-Black` then you would set the offset to 28 and the gpiochip as `gpiochip1`

```bash
gpioset gpiochip1 28=1
```

### CMAKE_SYSROOT and sshfs

To cross-compile a C++ project, CMake needs to find the libraries in the target. The `CMAKE_SYSROOT` variable will be used to let the CMake know where the target system libraries are located in. 
I thought of using `rsync` to copy the required libraries from `Beaglebone` to the host PC, but, I used `sshfs` to mount the Beaglebone root folder in the host pc.

```bash
cd ~
mkdir Beaglebone-Sysroot
sshfs debian@192.168.7.2:/ /home/kalyan/Beaglebone-Sysroot/ -o transform_symlinks
```

`/home/kalyan/Beaglebone-Sysroot/` path needs to be the folder that you have created and it will be used as a mount point where the root file system of beaglebone will be mounted.

## Installing Cross-compiler

I installed the cross-compiler using `sudo apt install crossbuild-essential-armhf` because the compiler that I downloaded from the GNU website doesn't has the correct `sysroot` setup and the builds were failing.

### CMake toolchain file

This is the CMake toolchain file that I am using

```cmake
# Set the system name
set(CMAKE_SYSTEM_NAME Linux)
# C Compiler
set(CMAKE_C_COMPILER /usr/bin/arm-linux-gnueabihf-gcc)
set(CMAKE_CXX_COMPILER /usr/bin/arm-linux-gnueabihf-g++)
# Sysroot location
# It is mounted using sshfs
set(CMAKE_SYSROOT /home/kalyan/Beaglebone-Sysroot)
# These lines are necessary to let cmake know to only look in the
# beaglebone for the libraries instead of looking in host pc.
# sshfs debian@192.168.7.2:/ /home/kalyan/Beaglegone-Sysroot/ -o transform_symlinks
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
```

The project can be found in [github](https://github.com/parzival2/ExploringBeaglebone).

