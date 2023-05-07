---
author: "Kalyan"
title: "Cross-compiler using crosstool-ng for Beaglebone"
date: "2021-04-27"
description: "Cross-compiler using crosstool-ng for Beaglebone"
tags: ["beaglebone", "crosstool-ng"]
ShowToc: true
---

## Cross-compiler using crosstool-ng for Beaglebone

In the [previous](https://parzival2.github.io/blog/posts/beaglebone-cross-compile/) post, I have used an already available cross-compiling compiler to cross-compile a C++ project for Beaglebone. I want to learn how I can do that so that I can utilize the learned knowledge if the need arises. 
I came to know that we can build a compiler using a tool called `crosstool-ng`. I am going to use it to build a compiler that can be used to cross-compile C and C++ applications for Beaglebone.

### Building crosstool-ng
The `crosstool-ng` has to be built as an application before it can be used to build the compiler. When built, it will produce an executable `ct-ng` that can be used to build the final compiler.

```bash
sudo apt install libtool-bin gperf bison flex texinfo help2man
git clone https://github.com/crosstool-ng/crosstool-ng
cd crosstool-ng
./bootstrap
./configure --enable-local
make
```

After running the above commands, you will see that they have produced an executable `ct-ng`.

Generate the configuration for `ARM cortex A8 processor` using `ct-ng`

```bash
./ct-ng arm-cortex_a8-linux-gnueabi
```

Run `menuconfig` to edit the configurations

```bash
./ct-ng menuconfig
```

Make changes in **`Paths and misc options`** as shown below

{{< rawhtml >}}

<a data-flickr-embed="true" href="https://www.flickr.com/photos/192459396@N08/51142774648/in/dateposted-public/" title="render-tool-chain-ro"><img src="https://live.staticflickr.com/65535/51142774648_0410cbbc17_b.jpg" width="996" height="642" alt="render-tool-chain-ro"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

{{< /rawhtml >}}

and also make changes in **`Target options`** to enable `hard-floating` point calculation capability.

{{< rawhtml >}}

<a data-flickr-embed="true" href="https://www.flickr.com/photos/192459396@N08/51141883977/in/dateposted-public/" title="floating-point"><img src="https://live.staticflickr.com/65535/51141883977_8d7017cc2c_b.jpg" width="997" height="640" alt="floating-point"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

{{< /rawhtml >}} 

Exit and build the compiler using 

```bash
./ct-ng build
```

It will take ~20minutes and if you have followed everything correctly then the build should succeed and you will have a folder in `~/x-tools` and the compiler will be in `~/x-tools/arm-cortex_a8-linux-gnueabihf/bin`

As you can see you didn't change any configurations related to the compiler version and the Linux version on Beaglebone. The compiler you generated will not work if it depends on the libraries that are on Beaglebone and you would like to use mount the file-system using `sshfs` for sysroot as mentioned in the [previous](https://parzival2.github.io/blog/posts/beaglebone-cross-compile/) post.

### Beaglebone side

Before we can build the compiler on the host-pc, we need to gather some data on the Beaglebone side.
It is important that the GCC compiler that we are trying to build and the one that we have on the Beaglebone match. The GCC compiler on the Beaglebone can be found by using the command: 
```bash
debian@beaglebone:~$ gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/arm-linux-gnueabihf/8/lto-wrapper
Target: arm-linux-gnueabihf
Configured with: ../src/configure -v --with-pkgversion='Debian 8.3.0-6' --with-bugurl=file:///usr/share/doc/gcc-8/README.Bugs --enable-languages=c,ada,c++,go,d,fortran,objc,obj-c++ --prefix=/usr --with-gcc-major-version-only --program-suffix=-8 --program-prefix=arm-linux-gnueabihf- --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --enable-bootstrap --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-libitm --disable-libquadmath --disable-libquadmath-support --enable-plugin --enable-default-pie --with-system-zlib --with-target-system-zlib --enable-objc-gc=auto --enable-multiarch --disable-sjlj-exceptions --with-arch=armv7-a --with-fpu=vfpv3-d16 --with-float=hard --with-mode=thumb --disable-werror --enable-checking=release --build=arm-linux-gnueabihf --host=arm-linux-gnueabihf --target=arm-linux-gnueabihf
Thread model: posix
gcc version 8.3.0 (Debian 8.3.0-6) 
```

Mine turned out to be `8.3.0`

Next we need to find the `GLIBC` version

```bash
debian@beaglebone:~$ ldd --version
ldd (Debian GLIBC 2.28-10) 2.28
```

Then `binutils` version

```bash
debian@beaglebone:~$ ld --version
GNU ld (GNU Binutils for Debian) 2.31.1
Copyright (C) 2018 Free Software Foundation, Inc.
```

We also need the current Linux version that our Beaglebone have

```bash
uname -a
Linux beaglebone 4.19.94-ti-rt-r60 #1buster SMP PREEMPT RT Wed Mar 17 16:22:33 UTC 2021 armv7l GNU/Linux
```

If you see the command where we determined `gcc -v` you will notice that `--enable-multiarch` has been enabled. This makes it to have another folder named `arm-linux-gnueabihf` in the `/usr/include` and `/usr/lib` folders as seen below

```bash
/usr/lib/
├── apt
│   ├── methods
│   ├── planners
│   └── solvers
├── arm-linux-gnueabihf
│   ├── audit
│   ├── coreutils
```

The cross-compiler that we are building needs to know that it has to look for some includes and libraries in that folder. This can be solved by applying a patch in `binutils` while building it using `crosstool-ng`.

Install the source for `binutils` on Beaglebone

```bash
sudo apt install binutils-source
```

### Patching binutils

Applying a patch for `binutils` is as simple as copying the patch from Beaglebone over to the host-pc. 

As you have already installed the `binutils-source` the patches are also installed. The source can be found in `/usr/src/binutils` on Beaglebone. In patches folder, copy and paste the `129_multiarch_libpath.patch` in `/crosstool-ng/packages/binutils/2.31.1/`. Please replace the `2.31.1` with the version that you determined by running `ld --version` in one of the previous step.

### Back to host-pc and finishing compilation

Now we are back to the host-pc for finishing up the compilation. We need to configure all the versions of the tools according to the versions that we determined in Beaglebone so as to avoid linker errors.

Run `./ct-ng menuconfig` to bring up the configuration window.

In **`Operating System`** change the version of Linux that closely corresponds to the Linux version of Beaglebone. I am going with `4.18.20`

{{< rawhtml >}}

<a data-flickr-embed="true" href="https://www.flickr.com/photos/192459396@N08/51143369214/in/dateposted-public/" title="linux-version"><img src="https://live.staticflickr.com/65535/51143369214_3cd5c4133a_b.jpg" width="989" height="636" alt="linux-version"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

{{< /rawhtml >}}

Next in **`Binary utilities`** section, change the version of binutils that is going to be used for the build. This has to correspond with the version that was determined earlier in Beaglebone.

{{< rawhtml >}}

<a data-flickr-embed="true" href="https://www.flickr.com/photos/192459396@N08/51141920702/in/dateposted-public/" title="binutils-version"><img src="https://live.staticflickr.com/65535/51141920702_f15a537876_b.jpg" width="982" height="633" alt="binutils-version"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

{{< /rawhtml >}}

Moving on to **`C-library`**, change the glibc version to match with the version from Beaglebone.

{{< rawhtml >}}

<a data-flickr-embed="true" href="https://www.flickr.com/photos/192459396@N08/51142816513/in/dateposted-public/" title="glibc-version"><img src="https://live.staticflickr.com/65535/51142816513_3cf0d34860_b.jpg" width="991" height="640" alt="glibc-version"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

{{< /rawhtml >}}

Now in **`C compiler`**, change the version of `gcc` to closely correspond to the version that is found in Beaglebone and also in `gcc extra config` add `--enable-multiarch` option as shown below. 

{{< rawhtml >}}

<a data-flickr-embed="true" href="https://www.flickr.com/photos/192459396@N08/51142822523/in/dateposted-public/" title="c-compiler-version"><img src="https://live.staticflickr.com/65535/51142822523_eb0ae30cd4_b.jpg" width="989" height="642" alt="c-compiler-version"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

{{< /rawhtml >}}

Run 

```bash
export DEB_TARGET_MULTIARCH=arm-linux-gnueabihf
```

Now build the toolchain again using `./ct-ng build`.

If you run 

```bash
./arm-cortex_a8-linux-gnueabihf-ld -verbose | grep -i "SEARCH"
```

from the compiler installation folder in `~/x-tools` you will see that it also searches for `arm-linux-gnueabihf` folder.

```bash
SEARCH_DIR("=/usr/local/lib/arm-linux-gnueabihf"); SEARCH_DIR("=/lib/arm-linux-gnueabihf"); SEARCH_DIR("=/usr/lib/arm-linux-gnueabihf"); SEARCH_DIR("=/usr/local/lib"); SEARCH_DIR("=/lib"); SEARCH_DIR("=/usr/lib"); SEARCH_DIR("=/home/kalyan/x-tools/arm-cortex_a8-linux-gnueabihf/arm-cortex_a8-linux-gnueabihf/lib");
```

You can use the generated compiler using this toolchain file.

```cmake
# Set the system name
set(CMAKE_SYSTEM_NAME Linux)
# C Compiler
set(COMPILER_PATH /home/kalyan/x-tools/arm-cortex_a8-linux-gnueabihf/bin)
set(COMPILER_FRONT arm-cortex_a8-linux-gnueabihf-)
set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)
set(CMAKE_C_COMPILER ${COMPILER_PATH}/${COMPILER_FRONT}gcc)
set(CMAKE_CXX_COMPILER ${COMPILER_PATH}/${COMPILER_FRONT}g++)
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

