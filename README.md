# Compiling universal binaries using a recent GCC on Mac OS X

This repository is forked from https://github.com/jhnbwrs/macportsGCCfixup

It contains a simple executable that replaces GCC, so that when compiling formore than one architecture:

- it compiles each architecture in separate object files
- it executes `lipo` to merge the architectures in a single fat object

This method was implemented by Apple for the GCC 4.2.1 provided in older SDKs, and the source code is available as [driverdriver.c](http://opensource.apple.com/source/gcc/gcc-5666.3/driverdriver.c). See the [original README](README-original.md) for details about the full history of this package. The original version used auxiliary shell scripts and did not work with arguments containing spaces. Now, it simply replaces the main GCC and executes the architecture-specific compilers, using the right options.

We just made a few modifications to make it work with more recent GCC, added `-m32` when compiling for i386 and friends, and added support for the `-fconstant-cfstrings` option.

The default installation method (see below) works on MacPorts, and replaces the main gcc/g++ binaries, so `make install` should be re-run each time GCC is upgraded.

It could be adapted to homebrew, or any custom-built GCC, but MacPorts globally has a much better support for building universal binaries.

If required, older Mac OS X SDKs can be installed using [xcodelegacy](https://github.com/devernay/xcodelegacy)

## Installation on MacPorts

    sudo port install gcc48 +universal gcc49 +universal gcc5 +universal gcc6 +universal
    
    mkdir build-gcc48
    cd build-gcc48
    cmake .. -DGCC_VERSION=4.8.5 -DIBERTY_LIB=/opt/local/lib/gcc48/libiberty.a
    make
    sudo make install
    cd ..
    
    mkdir build-gcc49
    cd build-gcc49
    cmake .. -DGCC_VERSION=4.9.4 -DIBERTY_LIB=/opt/local/lib/gcc48/libiberty.a
    make
    sudo make install
    cd ..
    
    mkdir build-gcc5
    cd build-gcc5
    cmake .. -DGCC_VERSION=5.4.0 -DIBERTY_LIB=/opt/local/lib/gcc48/libiberty.a
    make
    sudo make install
    cd ..
    
    mkdir build-gcc6
    cd build-gcc6
    cmake .. -DGCC_VERSION=6.4.0 -DIBERTY_LIB=/opt/local/lib/gcc48/libiberty.a
    make
    sudo make install
    cd ..

    mkdir build-gcc7
    cd build-gcc7
    cmake .. -DGCC_VERSION=7.2.0 -DIBERTY_LIB=/opt/local/lib/gcc48/libiberty.a
    make
    cd ..
