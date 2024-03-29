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
debian@BeagleBone:~/libgpiod$ git branch -a
  master
* v2.0.x
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/origin/v0.1.x
  remotes/origin/v0.2.x
  remotes/origin/v0.3.x
  remotes/origin/v1.0.x
  remotes/origin/v1.1.x
  remotes/origin/v1.2.x
  remotes/origin/v1.3.x
  remotes/origin/v1.4.x
  remotes/origin/v1.5.x
  remotes/origin/v1.6.x
  remotes/origin/v2.0.x
debian@BeagleBone:~/libgpiod$ git checkout remotes/origin/v2.0.x
```

Configure and compile it using the following commands

```bash
./autogen.sh --enable-tools=yes --enable-bindings-cxx=yes --prefix=/usr/local
make
sudo make install
```

go {{< notice note >}} If you get an error like this
```bash
debian@BeagleBone:~/libgpiod$ ./autogen.sh --enable-tools=yes  
--enable-bindings-cxx=yes --prefix=/usr/local
./autogen.sh: 12: autoreconf: not found
```
{{< /notice >}}

You need to install `autoconf`.

```bash
debian@BeagleBone:~/libgpiod$ sudo apt install autoconf
```
If debian complains that it was not able to find it
```bash
[sudo] password for debian:
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
E: Unable to locate package autoconf
```
If debian is not able to find the package, then update your packages using 
```bash
sudo apt update
```
and try again.
These are all the packages that I installed to build `libgpiod`. Note that it is a fresh installation of `debian` on `beaglebone`
```bash
sudo apt install pkg-config
sudo apt install autoconf
sudo apt install autoconf-archive
sudo apt install build-essential
# Or a single command
sudo apt install pkg-config autoconf autoconf-archieve build-essential
```
If you don't want to use C++, then remove the `--enable-bindings-cxx=yes` in the above command lines.
Then try compiling **libgpiod** again.
```bash
./autogen.sh --enable-tools=yes --enable-bindings-cxx=yes --prefix=/usr/local
```
The above command will take sometime. After it is done, execute 
```bash
make
sudo make install 
```

The `libgpiod` comes with a bunch of useful command line tools for driving and testing GPIOs.
```bash
debian@BeagleBone:~/libgpiod$ gpiodetect
```
If you get an error like this
```bash
gpiodetect: error while loading shared libraries: libgpiod.so.3: cannot open shared object file: No such file or directory
```
Then the executable is not able to find the library. **libgpiod** gives a hint when we executed `sudo make install`
```bash
----------------------------------------------------------------------
Libraries have been installed in:
   /usr/local/lib

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the '-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the 'LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the 'LD_RUN_PATH' environment variable
     during linking
   - use the '-Wl,-rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to '/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------
```
All we have to do is add the `lib` path to our library path.
```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
```

{{< notice note >}} 
The above command only works for the current terminal. If you want the changes to persist, add it to `~/.bashrc`.
{{< /notice >}}
All the commands take offset along with the gpiochip number that, that GPIO refers to.

E.g. If you want to drive P9-12 pin on the `Beaglebone-Black` then you would set the offset to 28 and the gpiochip as `gpiochip1`

```bash
gpioset gpiochip1 28=1
```
---
### CMAKE_SYSROOT and sshfs

To cross-compile a C++ project, CMake needs to find the libraries in the target. The `CMAKE_SYSROOT` variable will be used to let the CMake know where the target system libraries are located in. 
I thought of using `rsync` to copy the required libraries from `Beaglebone` to the host PC, but, I used `sshfs` to mount the Beaglebone root folder in the host pc.

```bash
cd ~
mkdir Beaglebone-Sysroot
sshfs debian@192.168.7.2:/ /home/kalyan/Beaglebone-Sysroot/ -o transform_symlinks
```
{{< notice info >}}
To unmount the mounted `sshfs` file system, you can use the command
```bash
fusermount -u /home/kalyan/Beaglebone-Sysroot
```
{{< /notice >}}
`/home/kalyan/Beaglebone-Sysroot/` path needs to be the folder that you have created and it will be used as a mount point where the root file system of beaglebone will be mounted.

#### Installing Cross-compiler

I installed the cross-compiler using `sudo apt install crossbuild-essential-armhf` because the compiler that I downloaded from the GNU website doesn't has the correct `sysroot` setup and the builds were failing.

#### CMake toolchain file

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
{{< notice info >}}
Go to the next section if you were not able to run the compiled programs on **Beaglebone**.
If you see an error like this
```
./ConstexprJoinArray: /lib/arm-linux-gnueabihf/libc.so.6: version `GLIBC_2.34' not found (required by ./ConstexprJoinArray)
```
{{< /notice >}}

{{< notice tip >}}
Can also use rsync. I copied entire **Beaglebones** file system
```bash
 rsync --info=progress2  -vaHAX --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"} debian@192.168.0.202:/ /home/kalyan/Beaglebone-Sysroot-rsync/
```
{{< /notice >}}

It seems that there is an issue with certain symlinks, specifically the symlinks `libpthread.so` and `librt.so`. These symlinks currently use absolute paths, which can cause problems. To address this, it is necessary to delete these symlinks and recreate them using relative paths.

One possible solution is to create a clean sysroot by cloning it, which can help resolve this issue. By properly specifying the rootfs in CMake, it is expected that CMake would handle these absolute links. However, it appears that CMake is unable to utilize these symlinks effectively.

I wanted to express my sincere gratitude for providing the Python script that effectively solves the issue I was facing with absolute symlinks in the root filesystem when cross-compiling. The script you shared, which can be found [here](https://forum.beagleboard.org/t/cloning-the-root-filesystem-for-cross-compiling/29828/2), has proven to be incredibly helpful in resolving this problem.

This Python script takes a sysroot directory as input and transforms all the absolute symlinks into relative ones, making the sysroot usable within another system. It accomplishes this by recursively traversing the directory structure and identifying symlinks that have absolute paths. It then replaces those absolute paths with relative ones based on the provided sysroot directory.

This ensures that the sysroot can be utilized seamlessly for cross-compiling purposes.

```python
import sys
import os

# Take a sysroot directory and turn all the abolute symlinks and turn them into
# relative ones such that the sysroot is usable within another system.

if len(sys.argv) != 2:
    print("Usage is " + sys.argv[0] + "<directory>")
    sys.exit(1)

topdir = sys.argv[1]
topdir = os.path.abspath(topdir)

def handlelink(filep, subdir):
    link = os.readlink(filep)
    if link[0] != "/":
        return
    if link.startswith(topdir):
        return
    #print("Replacing %s with %s for %s" % (link, topdir+link, filep))
    print("Replacing %s with %s for %s" % (link, os.path.relpath(topdir+link, subdir), filep))
    os.unlink(filep)
    os.symlink(os.path.relpath(topdir+link, subdir), filep)

for subdir, dirs, files in os.walk(topdir):
    for f in files:
        filep = os.path.join(subdir, f)
        if os.path.islink(filep):
            #print("Considering %s" % filep)
            handlelink(filep, subdir)
```

---
### GLIBC version Mismatch
What actually happened is version mismatch with *GLIBC*. This error can occur when you try to run a cross-compiled executable on a target system that has a different version of glibc than the one used for cross-compiling.
Remember that we have downloaded the `cross-compiler` without checking the version of `gcc` that we have on **Beaglebone** using `sudo apt install crossbuild-essential-armhf`.
If I check the version of compiler on the target
##### Target
```bash
debian@BeagleBone:~/ExamplePrograms$ gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/arm-linux-gnueabihf/10/lto-wrapper
Target: arm-linux-gnueabihf
Configured with: ../src/configure -v --with-pkgversion='Debian 10.2.1-6' --with-bugurl=file:///usr/share/doc/gcc-10/README.Bugs --enable-languages=c,ada,c++,go,d,fortran,objc,obj-c++,m2 --prefix=/usr --with-gcc-major-version-only --program-suffix=-10 --program-prefix=arm-linux-gnueabihf- --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --enable-bootstrap --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-libitm --disable-libquadmath --disable-libquadmath-support --enable-plugin --enable-default-pie --with-system-zlib --enable-libphobos-checking=release --with-target-system-zlib=auto --enable-objc-gc=auto --enable-multiarch --disable-sjlj-exceptions --with-arch=armv7-a --with-fpu=vfpv3-d16 --with-float=hard --with-mode=thumb --disable-werror --enable-checking=release --build=arm-linux-gnueabihf --host=arm-linux-gnueabihf --target=arm-linux-gnueabihf
Thread model: posix
Supported LTO compression algorithms: zlib zstd
gcc version 10.2.1 20210110 (Debian 10.2.1-6)
```
and compare it to host then we can see that the host has a newer version of the compiler.
##### Host
```bash
kalyan@DESKTOP-TT2VIL5:~/Beaglebone-Sysroot$ gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-linux-gnu/11/lto-wrapper
OFFLOAD_TARGET_NAMES=nvptx-none:amdgcn-amdhsa
OFFLOAD_TARGET_DEFAULT=1
Target: x86_64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 11.3.0-1ubuntu1~22.04' --with-bugurl=file:///usr/share/doc/gcc-11/README.Bugs --enable-languages=c,ada,c++,go,brig,d,fortran,objc,obj-c++,m2 --prefix=/usr --with-gcc-major-version-only --program-suffix=-11 --program-prefix=x86_64-linux-gnu- --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --enable-bootstrap --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-vtable-verify --enable-plugin --enable-default-pie --with-system-zlib --enable-libphobos-checking=release --with-target-system-zlib=auto --enable-objc-gc=auto --enable-multiarch --disable-werror --enable-cet --with-arch-32=i686 --with-abi=m64 --with-multilib-list=m32,m64,mx32 --enable-multilib --with-tune=generic --enable-offload-targets=nvptx-none=/build/gcc-11-xKiWfi/gcc-11-11.3.0/debian/tmp-nvptx/usr,amdgcn-amdhsa=/build/gcc-11-xKiWfi/gcc-11-11.3.0/debian/tmp-gcn/usr --without-cuda-driver --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu --with-build-config=bootstrap-lto-lean --enable-link-serialization=2
Thread model: posix
Supported LTO compression algorithms: zlib zstd
gcc version 11.3.0 (Ubuntu 11.3.0-1ubuntu1~22.04)
```
We need to compile the correct version of compiler ourselves. It can be done by following this guide [Cross-compiler using crosstool-ng for Beaglebone | blog (parzival2.github.io)](https://parzival2.github.io/blog/posts/crosstool-ng-beaglebone/)