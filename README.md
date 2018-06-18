# **Guide - Building XNU 17.4 (xnu-4570.41.2)**
For building XNU 17.4 (from start to finish) on macOS High Sierra.

**Requirements:** macOS 10.13.3 and Xcode 9.2

**These must be done in given order or the build will fail.**

Open `Terminal.app` and copy each command in as you go.   

### Creating Workspace
*************************************
> 
```
mkdir -p ~/Desktop/xnu_build
```
************************************
>
```
cd ~/Desktop/xnu_build
```

#### Curling Needed Sources
************************************
>
```
curl -O https://opensource.apple.com/tarballs/dtrace/dtrace-262.tar.gz && 
curl -O https://opensource.apple.com/tarballs/AvailabilityVersions/AvailabilityVersions-32.30.1.tar.gz && 
curl -O https://opensource.apple.com/tarballs/xnu/xnu-4570.41.2.tar.gz && 
curl -O https://opensource.apple.com/tarballs/libdispatch/libdispatch-913.30.4.tar.gz && 
curl -O https://opensource.apple.com/tarballs/libplatform/libplatform-161.20.1.tar.gz
```

#### Extracting and Removing tar.gz Files
************************************
>
```
for file in *.tar.gz; do tar -zxf $file; done && rm -f *.tar.gz
```

### Building Dtrace CTF Binaries
************************************
>
```
cd dtrace-262
```

************************************
>
```
mkdir -p obj sym dst
```

************************************
>
```
xcodebuild install -target ctfconvert -target ctfdump -target ctfmerge ARCHS="x86_64" SRCROOT=$PWD OBJROOT=$PWD/obj SYMROOT=$PWD/sym DSTROOT=$PWD/dst
```

************************************
>
```
sudo ditto $PWD/dst/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain
```

### Installing Availability
************************************
>
```
cd ../AvailabilityVersions-32.30.1/
```

>
```
mkdir -p dst
```
>
```
make install SRCROOT=$PWD DSTROOT=$PWD/dst
```

************************************
>
```
sudo ditto $PWD/dst/usr/local `xcrun -sdk macosx -show-sdk-path`/usr/local
```
#### Installing headers to build `libfirehose_kernel.a` for XNU
************************************
>
```
cd ../xnu-4570.41.2/
```

>
```
mkdir -p BUILD.hdrs/obj BUILD.hdrs/sym BUILD.hdrs/dst
```

>
```
make installhdrs SDKROOT=macosx ARCH_CONFIGS=X86_64 SRCROOT=$PWD OBJROOT=$PWD/BUILD.hdrs/obj SYMROOT=$PWD/BUILD.hdrs/sym DSTROOT=$PWD/BUILD.hdrs/dst
```

We must create this file in order for `xcodebuild` to succeed.
>
```
touch libsyscall/os/thread_self_restrict.h
```

>
```
sudo xcodebuild installhdrs -project libsyscall/Libsyscall.xcodeproj -sdk macosx ARCHS='x86_64 i386' SRCROOT=$PWD/libsyscall OBJROOT=$PWD/BUILD.hdrs/obj SYMROOT=$PWD/BUILD.hdrs/sym DSTROOT=$PWD/BUILD.hdrs/dst 
```
>
```
sudo chown -R root:wheel BUILD.hdrs/dst/
```
>
```
sudo ditto BUILD.hdrs/dst `xcrun -sdk macosx -show-sdk-path`
```

#### Copying libplatform sources needed for `libfirehose_kernel.a`
************************************
> 
```
cd ../libplatform-161.20.1
```
************************************
> 
```
sudo ditto $PWD/include `xcrun -sdk macosx -show-sdk-path`/usr/local/include
```
>
```
sudo ditto $PWD/private `xcrun -sdk macosx -show-sdk-path`/usr/local/include
```

#### Building libfirehose_kernel.a
************************************
> 
```
cd ../libdispatch-913.30.4
```

> 
```
mkdir -p obj sym dst
```
> 
```
sudo xcodebuild install -project libdispatch.xcodeproj -target libfirehose_kernel -sdk macosx ARCHS='x86_64 i386' SRCROOT=$PWD OBJROOT=$PWD/obj SYMROOT=$PWD/sym DSTROOT=$PWD/dst
```
>
```
sudo ditto $PWD/dst/usr/local `xcrun -sdk macosx -show-sdk-path`/usr/local
```

### Building XNU
************************************
> 
```
cd ../xnu-4570.41.2/
```
************************************
> 
```
sudo make SDKROOT=macosx ARCH_CONFIGS=X86_64 KERNEL_CONFIGS=RELEASE
```
