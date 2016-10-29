# Building with GCC 4.6 and Xcode 4

Sources:
- http://thecoderslife.blogspot.fr/2015/07/building-with-gcc-46-and-xcode-4.html
- http://web.archive.org/web/20140803170624/http://implbits.com/About/Blog/tabid/78/post/building-with-gcc-4-6-on-osx/Default.aspx


 NOTE: This is the resurrection of blog post from 2012, on a blog that is now defunct.   I am moving it over here because I think parts of it can still be somewhat useful although I am sure it is dated, and I am not sure the method for replacing the compiler that was used here will work with modern versions of Xcode

Begin:

I was recently faced with a problem building some C++ 11 code on OSX. I discovered that for all of the improvements Apple has put into LLVM, it has at least one glaring failure. It does not support lambda expressions. So I was forced to try to get GCC 4.6 building my project on OSX. I ran into two major issues.

1. GCC does not support fat binaries. (Mac Universal binaries)
2. XCode is configured to use clang, but I needed to configure it to run a different compiler.
3. I was able to resolve both of these problems and packaged a small project that you can use to easily overcome these as well!  If you want the Readers Digest version on how to get this working relatively quickly, skip to the end of the article to find the steps.  For the detailed explanation read on…


Building compilers isn’t my favorite pastime, so I looked around and found that the MacPorts project (http://www.macports.org/) makes downloading and installing GCC 4.6 (or a number of other versions) pretty painless.  
After downloading and installing the MacPorts installer I ran the command port install gcc46 +universal.    After a while the install finished, and I thought that maybe this wouldn’t be as bad as I had first imagined.  So I began my build (autoconf based command-line project). 
I discovered very quickly one of my first problems:

```
gcc-mp-4.6: error: unrecognized option '-arch'
gcc-mp-4.6: error: unrecognized option '-arch'
```

Ok, I guess I should have realized that this would happen.   I was trying to build “fat” (or universal) binaries using the `–arch i386` and `–arch x86_64` flags.  This allows you to include support for more than one platform architecture in a single binary. These are Apple extensions, so of course they wouldn’t be in the FSF version of gcc.  Being understandable doesn’t mean that it wasn’t a pain to work around.  I needed fat libraries/binaries to be compiled.  I started thinking about how to get around this problem.  I knew that you could use lipo to combine multiple object files of different architectures, so I thought maybe I could write a compiler wrapper that would honor the –arch flag and compile once with `–m64` and once with `–m32` and then call lipo to smash them back together and make a fat object file.

First, I searched to see if anyone else had gone to this effort before I spent all the time on it.  After a bit of web searching I discovered that Apple’s version of gcc 4.2 turns out to be exactly what I described in the previous paragraph. (http://lists.macosforge.org/pipermail/macports-dev/2011-September/016210.html).   Not only that, but the source was also freely available (http://opensource.apple.com/source/gcc/gcc-5666.3/driverdriver.c).  So if I could compile this and get it to wrap the MacPorts version of gcc, I would be golden.
I spent a bit of time hacking it and discovered that the driver basically calls a different compiler for each architecture in its architecture map.    So I simply made a 2 line script for i386, and x86_64:

```
#!/bin/sh
/opt/local/bin/gcc-mp-4.6 -m32 $@
```
```
#!/bin/sh
/opt/local/bin/gcc-mp-4.6 –m64 $@
```

I put this in the MacPorts /opt/local/bin and named them in a way so that when I compile driverdriver.c it will call the first for the 32 bit arch and the second for the 64 bit arch.    My new “compiler” is called gcc-mp and I dump it in /opt/local/bin.  The mp stands for MacPorts.
I repeated these steps for g++.  I hoped that the MacPorts version of gcc and g++ would be wrapped sufficiently to honor the –arch flags just like the Apple version of gcc 4.2.  I modified my configure file to use gcc-mp and g++-mp and sure enough it works like a charm.  I am now compiling and it honors the arch flag, and correctly creates fat binaries with both 32bit intel and 64bit intel architectures.

It wouldn't properly compile PPC, ARM, or any other architecture,  but that was just mainly because I didn’t need those architectures.  I assume it would be possible to get the MacPorts gcc 4.6 ARM compiler and do the same steps for ARM architectures.  That will be an exercise for the reader if you need ARM support. :)

So at this point my build was humming along until I hit a portion that calls xcodebuild to build an XCode project file.   Of course, I cannot choose my new compiler in the XCode project file.  It is still trying to use clang.  Well, surely someone had already fixed this problem.  After a few moments of Googling, I found a good blog post that got me started:

http://skurganov.blogspot.com/2010/10/integrating-gcc-44-to-xcode-macossnow.html

Basically, you can add compiler definitions to XCode by modifying/creating some XML definition files.  After a bit of fiddling I discovered that this article must have been based upon modifications to a version of XCode 3, and I was using XCode 4.2.1. These compiler plug-ins now appear to be located at:

```
/Developer/Library/Xcode/PrivatePlugIns/Xcode3Core.ideplugin/Contents/SharedSupport/Developer/Library/Xcode/Plug-ins/
```

If you happen to be using Xcode 4.3+ the location is similar, except the root is in /Applications instead of /Developer/Library:

```
/Applications/Xcode.app/Contents/PlugIns/Xcode3Core.ideplugin/Contents/SharedSupport/Developer/Library/Xcode/Plug-ins/
```

Additionally, just creating a gcc 4.6 compiler plug-in didn’t work.  That wasn’t too surprising since there was a gcc 4.2 compiler definition there already, but you can't choose it in XCode.   I recalled hearing that Apple wasn’t going to support gcc anymore.  It was llvm-gcc or clang in Lion (10.7).  Since the LLVM-GCC-4.2 plug-in showed up in XCode, I decided that I would make two plug-ins: one for 4.6 and one that pretends to be llvm-gcc based on the 4.6 plug-in.  This actually worked:

![GCC 4.6 shows up in Xcode](http://web.archive.org/web/20140803170624im_/http://implbits.com/Portals/0/MacportsGCC_SS.png)

 Well, that was exciting!  I even modified the compiler definition so it automatically compiles with the std=c++0x flag. (Why else would you ever go through this pain?)   I failed to include it originally and struggled for a few minutes to figure out why my C++ 11 code still wasn’t compiling.

At this point, I tried to compile my project and I ran into an error where it couldn’t find a .hmap file.  I don’t really know what is going on here, but I discovered that you can turn off the use of header maps by adding a custom build setting to your project “USE_HEADERMAP=NO”.  Sounds like a plan to me.  If anyone has a better suggestion for this, please leave a comment:

![USE_HEADERMAP](http://web.archive.org/web/20140803170624im_/http://implbits.com/Portals/0/img/HeaderMap.png)

After adding this, I was good for about 30 seconds until I get to the portion of my project where some COCOA Objective-C UI stuff was being compiled.

/Developer/SDKs/MacOSX10.7.sdk/System/Library/Frameworks/Foundation.framework/Headers/NSTask.h:75:24: error: expected unqualified-id before '^' token

/Developer/SDKs/MacOSX10.7.sdk/System/Library/Frameworks/Foundation.framework/Headers/NSTask.h:75:24: error: expected ')' before '^' token 

Uh oh.  I know what this is.  This is the Apple "blocks" language extension.  It appears that blocks are used in a bunch of the system header files.  I don't think there is going to be a way to get around this using MacPorts gcc.  The FSF gcc just doesn’t know about blocks.  Fortunately for me, I didn't have anything in the Objective C/C++ code that needed to be compiled with gcc 4.6 so I just had this target compile using clang. 

This works alright when I link to only C libs compiled by GCC 4.6, but when I try to link to a C++ lib built by GCC 4.6 I get a bunch of linker problems.  I was able to restructure the code to remove the dependency on the gcc 4.6 C++ library so that I was only linking with C libraries.  I should probably look into this some more, but If anyone else out there has had this problem and knows how to resolve the C++ linkage issues, please leave a comment.

After all of this, the project finally finished compiling and it works. 

Victory!  Until I have to remember and repeat all of this stuff on a new machine 3 months from now.   So I decided to make a little CMAKE project to create the compiler wrappers and extend XCode.  So now, you too can use MacPorts gcc from within XCode to create fat binaries.

## Follow these 6 steps to easily replicate what I have done:

1. Download CMAKE at: http://cmake.org/cmake/resources/software.html. This project requires it.
2. Install macports for your OSX version from this URL:  http://www.macports.org/install.php
3. Install the UNIVERSAL version of gcc46 using macports (this might take a while):
    /opt/local/bin/port install gcc46 +universal
4. Download and unzip the macportsgccfixup tarball I created from here: MacPorts GCC Fix-up
5. Run `./configure` (which wraps the cmake configuration command)
6. Run `make` and then `make install`. 

NOTE:  If you don’t install the universal version of macports GCC you will eventually get some linking errors when it comes to finding the c++ libs for the non-native architecture:
```
ld: warning: ignoring file /opt/local/lib/gcc46/libstdc++.dylib, file was built for unsupported file format which is not the architecture being linked (i386)
ld: warning: ignoring file /opt/local/lib/gcc46/libgcc_ext.10.5.dylib, missing required architecture i386 in file
ld: warning: ignoring file /opt/local/lib/gcc46/gcc/x86_64-apple-darwin10/4.6.2/libgcc.a, file was built for archive which is not the architecture being linked (i386)
```

## Limitations/Caveats:

1. I did all of this using XCode 4.2.1 on 10.7.  I tested it on 10.6 as well, but it probably won’t work on any earlier versions of OSX (I used Xcode 4.2 on 10.6 as well).
2. As earlier noted it only supports 32 bit and 64 bit intel.  More work would be required to get anything else working, so no ARM and no PPC
3. It almost certainly won’t work with XCode 4.3 yet, but that is alright since I don’t think macports works with XCode 4.3 yet either.
4. It MIGHT work with XCode 3.6.x.  It would need to be tested.
5. Any c++ binaries built using GCC 4.6 will have a dependency upon the c++ libs in /opt/local/lib.  If you want to distribute something you have built this way you will either need to first install the macports GCC on the target machine, or distribute all of the c++ libraries in your distribution.

As aFinal note: it should be easy to change version of GCC that all this works with by simple changing the version of GCC in the `CMakeLists.txt`.  I haven’t tested it yet, but if you want 4.7 you might give it a try.

Get the code here:  https://github.com/jhnbwrs/macportsGCCfixup
