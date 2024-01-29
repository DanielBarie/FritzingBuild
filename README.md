# FritzingBuild
Building Fritzing with and for Ubuntu 22.04 LTS on AMD64.

Please don't ask for binaries. For obvious reasons, see [this explanation](https://forum.fritzing.org/t/can-t-find-source-code/19723/6).   
Please don't ask for help. StackOverflow is your friend.  

Quite an expensive process figuring this out, about €Multi-K when taking my hourly wage. At least I've dusted off my software build skills. Repeated builds will of course be cheaper...

![About Window Fritzing](https://github.com/DanielBarie/FritzingBuild/assets/73287620/6673ce88-e0bf-4592-9234-d20469ed429b)

![Fritzing Simulation with Smoke](https://github.com/DanielBarie/FritzingBuild/assets/73287620/ef05ea46-73ee-443e-af6a-48635b21f49d)

![grafik](https://github.com/DanielBarie/FritzingBuild/assets/73287620/e97830fd-37e5-4151-91bf-8deac4182c02)

This is a more (final build instructions) or less (separate instructions for static / dynamic builds) structured set of steps. The less well structured parts are a documentation of what worked and what didn't work. Also included: Fixes for errors/difficulties encountered along the way.

# What?
Obviously building Fritzing for teaching purposes. The goal is having a release and building a VM for students to explore electrical circuits and their simulation using Fritzing.

We'll build with the latest (as of writing) version of Qt which is 6.6.1. For the brave: Download any version greater than 6.5.2 (minimum requirement for Fritzing) at [Qt Release Archive](https://download.qt.io/archive/qt/). Please be advised to change paths accordingly.

Compilation will give some deprecation warnings regarding Qt functions. Will work anyway, we'll maybe have to re-visit this if Qt remove/change these functions.

# Why?
- There's instructions for building fritzing on their Github: https://github.com/fritzing/fritzing-app/wiki/1.-Building-Fritzing
- These do only refer to building from Qt Studio, so you'll have to go through Qt studio every time you might want to launch the app. Suboptimal for teaching purposes.
- Release build instructions (https://github.com/fritzing/fritzing-app/wiki/4.-Publishing-a-Release) are outdated with a link to setting up a build VM leading nowhere. The missing instructions are located at https://github.com/fritzing/fritzing-app/wiki/4.1-Via-Linux-on-a-virtual-box but refer to a very old Ubuntu release as build platform.
- There's another page called linux notes: https://github.com/fritzing/fritzing-app/wiki/1.3-Linux-notes

All that documentation is inconclusive, inconsistent and outdated. This just is a huge pain.

So I decided to try this with a newer release of Ubuntu, here we go.

# How ?
We'll set up a sufficiently beefy VM and get going in there. No containerized building.

This is a little diatribe concerning possible options and also giving the reasoning behind the chosen solution. Skip this, the To Do and Alternatives, if you feel like you don't want to be bothered with these aspects.

I originally intended having a static build with everything bundled for easier deployment:
- Qt is at a lower version in most distros than the one Fritzing depends on, Qt can easily be built as static libraries.
- SpiceNG as well (Fritzing is at 40, Ubuntu 22.04 at 36), I thought a static build was easy, after all, the library build is specified by just setting a parameter
- Clipping lib doesn't come as binaries
- Quazip is closely linked to Qt version

So a static build was a reasonable solution. 

Well, Fritzing and SpiceNG threw some wrenches:  

- Fritzing explicitly checks for loading of a dynamic version of SpiceNG
- SpiceNG needs modifications in its automake files at three different places for a static build (which in the end I succeeded doing but couldn't use without patching the Fritzing source).

So in the end, I did a mixed build. Some parts were linked statically, some dynamically and copied over to the `.lib` directory where the Fritzing binary resides.

Static:
- Qt
- Clipping
- Quazip

Dynamic:
- SpiceNG
- git2 (from distro)
- svgpp (which isn't a library in the original sense) (from distro)

# To Do
- re-structure (static and dynamic build steps)
- create script for patching pri files
- maybe create a snap or flatpack?

# Alternatives?
## Build System
I went for Ubuntu because of the LTS release and because I wrongly assumed having an easier build process. Boy, was I mistaken (see above). But with 22.04 LTS,  Qt is at 6.2.4. Fritzing 1.0.2 depends on Qt > 6.5.2. Don't choose Ubuntu 23.10 because, guess what, it's running ```Using Qt version 6.4.2 in /usr/lib/x86_64-linux-gnu``` When focusing on LTS, we might as well take Debian Bookworm (five years from mid 2023 on). But this also uses Qt 6.4.2. 

So having the correct version of Qt is the main issue if you want to skip building Qt yourself. You can, of course get pre-built binaries of any Qt version from the Qt site if you have/create an account.

## Purely Dynamic Linking
Sure, possible. Be prepared to copy all Qt dependencies and required libraries (git2, Quazip, Clipping).
Which makes for a ton of dependencies to manually keep tabs on.

## Purely Static Linking
Sure, possible, see notes below.
But you need to patch the Fritzing source for that to work.


# Steps for Ubuntu 22.04 Build Environment
Set up the basic build environment (VM, dependencies) for building Fritzing.  

Caveat: Ubuntu 22.04 is at node.js 12.22.9 so too low for qt to build some components (nodejs > 14 required, doesn't matter for us (pdf components))
- get Ubuntu 22.04 LTS
- create VM (Qt build will take approx 7 minutes with below resource configuration, AMD Epyc at 2.3GHz)
  - 128 GB disk space (SSD) (Qt build takes approx. 32 GB)
  - 262144 MB RAM (64GB will be ok, too)
  - 64 cores (if you have some to spare, hand them over...main issue is qt compilation, get the binaries if in a hurry)
  - network
- install Ubuntu
  - minimal install
- post-install:
  - be root (or pre-pend sudo to each of the commands below) 
  - ```apt install openssh-server mc qemu-guest-agent```
  - ```apt-get update```
  - ```apt-get upgrade```
- prepare for build, assuming you're happy to build below your user home dir
	- ```sudo apt-get install build-essential git cmake libssl-dev libudev-dev libglu1-mesa-dev freeglut3-dev mesa-common-dev libdrm-dev libgles2-mesa-dev pkg-config```
- get and untar Boost:
	- ```wget https://boostorg.jfrog.io/artifactory/main/release/1.84.0/source/boost_1_84_0.tar.gz```
 	- ```tar xzvf boost_1_84_0.tar.gz```

# Build Dependencies and Fritzing
This will be multiple steps involving download of various dependencies, unpacking, editing, and building these. For your orientation, there's a diagram of the resulting directory structure in a [separate section](#directory-structure-of-build). 
For version 1.0.2b of the Fritzing repo, there's a [tarball](dbfiles.tar) in this repo that will produce the required edited project file (`phoenix.pro`) and `.pri` files. Untar it after unpacking the components and save yourself the manual edits.

## Do Qt build
- `sudo apt-get install libwayland-dev  libwayland-egl1-mesa libwayland-server0 libgles2-mesa-dev libxkbcommon-dev`
- `sudo apt-get install libfontconfig1-dev libfreetype6-dev libx11-dev libx11-xcb-dev libxext-dev libxfixes-dev libxi-dev libxrender-dev libxcb1-dev libxcb-cursor-dev libxcb-glx0-dev`
- `sudo apt-get install libxcb-keysyms1-dev libxcb-image0-dev libxcb-shm0-dev libxcb-icccm4-dev libxcb-sync-dev libxcb-xfixes0-dev libxcb-shape0-dev libxcb-randr0-dev libxcb-render-util0-dev libxcb-util-dev libxcb-xinerama0-dev libxcb-xkb-dev libxkbcommon-dev libxkbcommon-x11-dev`
- `sudo apt-get install libxrender1 libxcb-render-util0 libxcb-shape0 libxcb-randr0 libxcb-xfixes0 libxcb-xkb1 libxcb-sync1 libxcb-shm0  libxcb-icccm4 libxcb-keysyms1-dev libxcb-image0  libxcb-util1 libxcb-cursor0 libxkbcommon-tools libxkbcommon-x11-0 libxkbcommon0 libfontconfig libfreetype6 libxext6 libx11-6 libxcb1 libx11-xcb1 libsm6 libice6 libglib2.0-0 libglib2.0-bin`
- `mkdir ~/qt-build`
- `cd ~/qt-build`
- get qt sources from https://download.qt.io/official_releases/qt/6.6/6.6.1/single/: ```https://download.qt.io/official_releases/qt/6.6/6.6.1/single/qt-everywhere-src-6.6.1.tar.xz```
- untar the sources: `tar xf...` takes a while in silence...
- `mkdir qt-static`
- `cd qt-static`
- ` ../qt-everywhere-src-6.6.1/configure -static -qt-zlib -prefix /opt/Qt6.6.1`
- optional: check config.result for appropriate configuration (X11 / Wayland platform plugins, qt-zlib = yes, shared libraries = no)
- ```cmake --build . --parallel```, libs will be in `~/qt-build/qt-static/qtbase/lib`
- ```sudo cmake --install .``` will install to `/opt/Qt6.6.1`
- ```sudo apt-get install qtchooser```
- ```qtchooser -install qt6 /opt/Qt6.6.1/bin/qmake```
- ```export QT_SELECT=qt6``` (you need to do this after each login or make it permanent in .bashrc)

## Satisfy libgit2 requirement
Install libgit2-dev and libgit2:
`sudo apt-get install libgit2-dev libgit2-1.1`

## Do spiceng Build
- get sources from https://ngspice.sourceforge.io/download.html
- untar
- ```sudo apt-get install libxaw7-dev automake libtool```
- change to unzip dir
- `./compile_linux_shared.sh`
- Library file will be in `~/ngspice-42/releasesh/src/.libs`: `libngspice.so`

## Do quazip Build
- `sudo apt-get install zlib1g-dev libbz2-dev`
- `wget https://github.com/stachenov/quazip/archive/refs/tags/v1.4.tar.gz`
- untar
- Skip with modified `pri/quazipdetect.pri`: rename to expected dir name e.g. `mv quazip-1.4 quazip-6.6.1-1.4`, made of Qt version number and expected version of quazip (1.4)
- `cd quazip-1.4`
- `nano CMakelists.txt`
	- change
  	```
    	else()
 		option(BUILD_SHARED_LIBS "" ON)
   		option(QUAZIP_INSTALL "" ON)
  		option(QUAZIP_USE_QT_ZLIB "" OFF)
   	```
   	to be
  	```
   	else()
 		option(BUILD_SHARED_LIBS "" OFF)
   		option(QUAZIP_INSTALL "" OFF)
  		option(QUAZIP_USE_QT_ZLIB "" OFF)
   	```
- `mkdir build-dir`
- cmake needs to be called with path to qt6 files: `cmake .. -D QUAZIP_QT_MAJOR_VERSION=6 -D CMAKE_PREFIX_PATH="/opt/Qt6.6.1/lib/cmake"`
- `cmake --build ./ --parallel`
- resulting library `libquazip1-qt6.a` will be in `~/quazip-1.4/build-dir/quazip`

## Do Clipping Library Build (static)
- get it from sourceforge: `https://sourceforge.net/projects/polyclipping/files/latest/download`
- `mkdir Clipper1`, UPPERCASE!
- `mv clipper_ver6.4.2.zip Clipper1`
- `cd Clipper1`
- unzip
- `cd cpp`
- change `cpp/CMakeLists.txt`:
	- find
 		```
   		SET(BUILD_SHARED_LIBS ON CACHE BOOL
    		"Build shared libraries (.dll/.so) instead of static ones (.lib/.a)")
		```  
 	- change to:
  - 		```
   		SET(BUILD_SHARED_LIBS OFF CACHE BOOL
    		"Don't Build shared libraries (.dll/.so) instead of static ones (.lib/.a)")
		```  
- `mkdir build-dir`
- `cd build-dir`
- `cmake ..`
- `make -j`
- Indicator of success: watch for ```[100%] Linking CXX static library libpolyclipping.a```
- library file `libpolyclipping.a` will be in `~/Clipper1/cpp/build-dir`

## Satisfy svgpp Requirement
If you don't mind the Megabytes, just do a quick
```
sudo apt-get install libsvgpp-dev
```
Else see below for build instructions.

## Prepare Fritzing build
- `cd ~`
- ```git clone https://github.com/fritzing/fritzing-app.git```
	- optional: ```git clone https://github.com/fritzing/fritzing-parts```
	- change compile script (phoenix.pro):
		- allow for later versions of qt
   	- ```nano ./fritzing-app/phoenix.pro```
   		- change QT_MOST to 6.6.10 (or whatever is sufficiently high)
   		- in unix!macx section, below `LIBS += -lz` add new line `QTPLUGIN.platforms = qminimal qxcb qoffscreen qwayland qvnc` (see: https://doc.qt.io/qt-6/qpa.html)
- change boost detect script (will only accept up to 81)
    - ```nano ./fritzing-app/pri/boostdetect.pri```
   	- find `BOOSTS = 81`
   	- change to `BOOSTS = 84`
- fix ngspice detection (will not set correct include dir)
   	- include must point to `../ngspice-42/src/include`but instead points to `ngspice-40/include`
   	- `nano ./pri/spicedetect.pri`
   	- find `NGSPICEPATH = ../../ngspice-40`, change to `NGSPICEPATH = ../../ngspice-42`
   	- find `INCLUDEPATH += $$NGSPICEPATH/include`, change to `INCLUDEPATH += $$NGSPICEPATH/src/include`
   	- add (e.g. below NGSPICEPATH directive): ```LIBS += -L$$absolute_path($${NGSPICEPATH}/releasesh/src/.libs) -lngspice``` 
- fix clipper detect script:
   	- `nano ./pri/clipper1detect.pri`
   	- in unix branch: change ```CLIPPER1 = $$absolute_path($$PWD/../../Clipper1/6.4.2)``` to be ``` CLIPPER1 = $$absolute_path($$PWD/../../Clipper1)``` (no trailing slash!)
   	- change `LIBS += -L$$absolute_path($${CLIPPER1}/lib) -lpolyclipping` to be ```LIBS += -L$$absolute_path($${CLIPPER1}/cpp/build-dir) -lpolyclipping``` 
   	- change ```INCLUDEPATH += $$absolute_path($${CLIPPER1}/include/polyclipping)``` to be ```INCLUDEPATH += $$absolute_path($${CLIPPER1}/cpp)```
- fix quazip detect script:
   	- `nano pri/quazipdetect.pri`
   	- change `QUAZIP_PATH=$$absolute_path($$PWD/../../quazip-$$QT_VERSION-$$QUAZIP_VERSION)` to be `QUAZIP_PATH=$$absolute_path($$PWD/../../quazip-1.4)`
   	- change ```QUAZIP_INCLUDE_PATH=$$QUAZIP_PATH/include/QuaZip-Qt6-$$QUAZIP_VERSION```to be ```QUAZIP_INCLUDE_PATH=$$QUAZIP_PATH``` (no trailing slash, will set quazip-prefix in source code include define)
   	- change `QUAZIP_LIB_PATH=$$QUAZIP_PATH/lib` to be `QUAZIP_LIB_PATH=$$QUAZIP_PATH/build-dir/quazip`
   	- change ```LIBS += -L $$QUAZIP_LIB_PATH -lquazip1-qt$$QT_MAJOR_VERSION``` to be ```LIBS += -L $$QUAZIP_LIB_PATH -lquazip1-qt$$QT_MAJOR_VERSION -lbz2```
 	
- Test build
  - `qmake`
  	- qmake failed to get called? Did you set the export `QT_SELECT=qt6`?
  - `make -j`
- Everything ok? proceed... if not: fix
  - `make clean`
  - `rm Makefile*`

You're now ready for calling the release script.

# Building Release
We've taken some DIY shortcuts: 
- We're set on doing a development release
- But with the master branch parts repo
- Please be aware that the script will normally work without a GUI except if parts from the fritzing-parts repo have errors in their definitions.
  
## Now, go do it:
- `cd ~/fritzing-app/tools/linux_release_script`
- Drop the [modified release script](https://github.com/DanielBarie/FritzingBuild/blob/main/release_db.sh) (`release_db.sh`) from this repo into that folder
- `./release_db.sh <your version string>`: Version string must include `develop` because we've taken some shortcuts.

You'll end up with a .tar.bz2 file including the only (specific) dependency (libspiceng). 

## Deploy it
There still are a ton of dependencies, many of which are bread and butter libraries:
```
ldd Fritzing
        linux-vdso.so.1 (0x00007ffcb2d7c000)
        libz.so.1 => /lib/x86_64-linux-gnu/libz.so.1 (0x00007f8269bb3000)
        libgit2.so.1.1 => /lib/x86_64-linux-gnu/libgit2.so.1.1 (0x00007f8267300000)
        libbz2.so.1.0 => /lib/x86_64-linux-gnu/libbz2.so.1.0 (0x00007f8269ba0000)
        libxkbcommon-x11.so.0 => /lib/x86_64-linux-gnu/libxkbcommon-x11.so.0 (0x00007f8269b95000)
        libxcb-cursor.so.0 => /lib/x86_64-linux-gnu/libxcb-cursor.so.0 (0x00007f8267000000)
        libxcb-icccm.so.4 => /lib/x86_64-linux-gnu/libxcb-icccm.so.4 (0x00007f8269b8c000)
        libxcb-image.so.0 => /lib/x86_64-linux-gnu/libxcb-image.so.0 (0x00007f8269b86000)
        libxcb-keysyms.so.1 => /lib/x86_64-linux-gnu/libxcb-keysyms.so.1 (0x00007f8269b81000)
        libxcb-randr.so.0 => /lib/x86_64-linux-gnu/libxcb-randr.so.0 (0x00007f8269b6e000)
        libxcb-render-util.so.0 => /lib/x86_64-linux-gnu/libxcb-render-util.so.0 (0x00007f8269b67000)
        libxcb-shm.so.0 => /lib/x86_64-linux-gnu/libxcb-shm.so.0 (0x00007f8269b62000)
        libxcb-sync.so.1 => /lib/x86_64-linux-gnu/libxcb-sync.so.1 (0x00007f82672f6000)
        libxcb-xfixes.so.0 => /lib/x86_64-linux-gnu/libxcb-xfixes.so.0 (0x00007f82672ec000)
        libxcb-render.so.0 => /lib/x86_64-linux-gnu/libxcb-render.so.0 (0x00007f82672dd000)
        libxcb-shape.so.0 => /lib/x86_64-linux-gnu/libxcb-shape.so.0 (0x00007f82672d8000)
        libxcb-xkb.so.1 => /lib/x86_64-linux-gnu/libxcb-xkb.so.1 (0x00007f82672ba000)
        libSM.so.6 => /lib/x86_64-linux-gnu/libSM.so.6 (0x00007f82672af000)
        libICE.so.6 => /lib/x86_64-linux-gnu/libICE.so.6 (0x00007f8267292000)
        libxcb-glx.so.0 => /lib/x86_64-linux-gnu/libxcb-glx.so.0 (0x00007f8267275000)
        libdrm.so.2 => /lib/x86_64-linux-gnu/libdrm.so.2 (0x00007f826725f000)
        libX11-xcb.so.1 => /lib/x86_64-linux-gnu/libX11-xcb.so.1 (0x00007f826725a000)
        libxcb.so.1 => /lib/x86_64-linux-gnu/libxcb.so.1 (0x00007f8267230000)
        libEGL.so.1 => /lib/x86_64-linux-gnu/libEGL.so.1 (0x00007f826721d000)
        libfreetype.so.6 => /lib/x86_64-linux-gnu/libfreetype.so.6 (0x00007f8266f38000)
        libfontconfig.so.1 => /lib/x86_64-linux-gnu/libfontconfig.so.1 (0x00007f8266eee000)
        libX11.so.6 => /lib/x86_64-linux-gnu/libX11.so.6 (0x00007f8266dae000)
        libxkbcommon.so.0 => /lib/x86_64-linux-gnu/libxkbcommon.so.0 (0x00007f8266d67000)
        libbrotlidec.so.1 => /lib/x86_64-linux-gnu/libbrotlidec.so.1 (0x00007f826720f000)
        libudev.so.1 => /lib/x86_64-linux-gnu/libudev.so.1 (0x00007f8266d3d000)
        libGLX.so.0 => /lib/x86_64-linux-gnu/libGLX.so.0 (0x00007f8266d09000)
        libOpenGL.so.0 => /lib/x86_64-linux-gnu/libOpenGL.so.0 (0x00007f8266cdd000)
        libstdc++.so.6 => /lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f8266a00000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f8266919000)
        libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f8266cbd000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f8266600000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f8269be0000)
        libmbedtls.so.14 => /lib/x86_64-linux-gnu/libmbedtls.so.14 (0x00007f8266c8a000)
        libmbedx509.so.1 => /lib/x86_64-linux-gnu/libmbedx509.so.1 (0x00007f8266c68000)
        libmbedcrypto.so.7 => /lib/x86_64-linux-gnu/libmbedcrypto.so.7 (0x00007f8266898000)
        libpcre.so.3 => /lib/x86_64-linux-gnu/libpcre.so.3 (0x00007f826658a000)
        libhttp_parser.so.2.9 => /lib/x86_64-linux-gnu/libhttp_parser.so.2.9 (0x00007f8266c5c000)
        libssh2.so.1 => /lib/x86_64-linux-gnu/libssh2.so.1 (0x00007f8266854000)
        libgssapi_krb5.so.2 => /lib/x86_64-linux-gnu/libgssapi_krb5.so.2 (0x00007f8266536000)
        libxcb-util.so.1 => /lib/x86_64-linux-gnu/libxcb-util.so.1 (0x00007f8266c53000)
        libuuid.so.1 => /lib/x86_64-linux-gnu/libuuid.so.1 (0x00007f8266c4a000)
        libbsd.so.0 => /lib/x86_64-linux-gnu/libbsd.so.0 (0x00007f8266c32000)
        libXau.so.6 => /lib/x86_64-linux-gnu/libXau.so.6 (0x00007f8266c2c000)
        libXdmcp.so.6 => /lib/x86_64-linux-gnu/libXdmcp.so.6 (0x00007f826684c000)
        libGLdispatch.so.0 => /lib/x86_64-linux-gnu/libGLdispatch.so.0 (0x00007f826647e000)
        libpng16.so.16 => /lib/x86_64-linux-gnu/libpng16.so.16 (0x00007f8266443000)
        libexpat.so.1 => /lib/x86_64-linux-gnu/libexpat.so.1 (0x00007f8266412000)
        libbrotlicommon.so.1 => /lib/x86_64-linux-gnu/libbrotlicommon.so.1 (0x00007f8266829000)
        libcrypto.so.3 => /lib/x86_64-linux-gnu/libcrypto.so.3 (0x00007f8265e00000)
        libkrb5.so.3 => /lib/x86_64-linux-gnu/libkrb5.so.3 (0x00007f8266345000)
        libk5crypto.so.3 => /lib/x86_64-linux-gnu/libk5crypto.so.3 (0x00007f8266316000)
        libcom_err.so.2 => /lib/x86_64-linux-gnu/libcom_err.so.2 (0x00007f8266310000)
        libkrb5support.so.0 => /lib/x86_64-linux-gnu/libkrb5support.so.0 (0x00007f8266302000)
        libmd.so.0 => /lib/x86_64-linux-gnu/libmd.so.0 (0x00007f82662f5000)
        libkeyutils.so.1 => /lib/x86_64-linux-gnu/libkeyutils.so.1 (0x00007f82662ec000)
        libresolv.so.2 => /lib/x86_64-linux-gnu/libresolv.so.2 (0x00007f82662d8000)
```


## Directory structure of build:
```
.
├── boost_1_84_0
│   ├── boost
│   ├── doc
│   ├── libs
│   ├── more
│   ├── status
│   └── tools
├── Clipper1
│   ├── C#
│   ├── cpp
│   ├── Delphi
│   ├── Documentation
│   └── Third Party
├── fritzing-app
│   ├── config.tests
│   ├── debug
│   ├── docker
│   ├── help
│   ├── pri
│   ├── release
│   ├── resources
│   ├── sketches
│   ├── src
│   ├── test
│   ├── tests
│   ├── tools
│   └── translations
├── ngspice-42
│   ├── autom4te.cache
│   ├── examples
│   ├── m4
│   ├── man
│   ├── releasesh
│   ├── src
│   ├── tests
│   └── visualc
├── qt-build
│   ├── qt-everywhere-src-6.6.1
│   └── qt-static
├── quazip-1.4
    ├── build-dir
    ├── cmake
    ├── quazip
    └── qztest

```

# These are some random notes on dynamic and static builds of various things (dependencies and Fritzing)
Below steps are different for shared libs / dynamic linking and static linking.

  
# Dynamically linked, Qt-zlib
Works, use [this release script](release_dyn.sh) which tries to copy over required libraries (Qt, spice, git2, quazip, clipping).
Needs some finishing touches, see ugly fix for problems involving symbol references.

## Prepare and Do Qt Build
  - note: Be careful when re-configuring. There may be artifacts from previous builds. See https://stackoverflow.com/questions/6067271/qt-when-building-qt-from-source-how-do-i-clean-old-configure-configurations
  - note: Documentation: https://doc.qt.io/qt-6/configure-options.html section "Reconfiguring Existing Builds"
  - note: Do also try removing 'CMakeCache.txt' from the build directory
  - note: Care about modules?  `/usr/local/Qt-6.6.1/bin/qt-configure-module`
  - `sudo apt-get install libwayland-dev libwayland-egl1-mesa libwayland-server0 libgles2-mesa-dev libxkbcommon-dev`
  - `sudo apt-get install libfontconfig1-dev libfreetype6-dev libx11-dev libx11-xcb-dev libxext-dev libxfixes-dev libxi-dev libxrender-dev libxcb1-dev libxcb-cursor-dev libxcb-glx0-dev`
  - `sudo apt-get install libxcb-keysyms1-dev libxcb-image0-dev libxcb-shm0-dev libxcb-icccm4-dev libxcb-sync-dev libxcb-xfixes0-dev libxcb-shape0-dev libxcb-randr0-dev libxcb-render-util0-dev libxcb-util-dev libxcb-xinerama0-dev libxcb-xkb-dev libxkbcommon-dev libxkbcommon-x11-dev`
  - `sudo apt-get install libxrender1 libxcb-render-util0 libxcb-shape0 libxcb-randr0 libxcb-xfixes0 libxcb-xkb1 libxcb-sync1 libxcb-shm0  libxcb-icccm4 libxcb-keysyms1-dev libxcb-image0  libxcb-util1 libxcb-cursor0 libxkbcommon-tools libxkbcommon-x11-0 libxkbcommon0 libfontconfig libfreetype6 libxext6 libx11-6 libxcb1 libx11-xcb1 libsm6 libice6 libglib2.0-0 libglib2.0-bin`
  - `mkdir qt-build`
  - `cd qt-build`
  - get qt sources from https://download.qt.io/official_releases/qt/6.6/6.6.1/single/: ```https://download.qt.io/official_releases/qt/6.6/6.6.1/single/qt-everywhere-src-6.6.1.tar.xz```
  - untar the sources: `tar xf...` takes a while in silence...
- Do Qt build:
	- Go to directory: `cd qt-everywhere-src-6.6.1`  
  - ```./configure -qt-zlib```
  - ```cmake --build . --parallel```
  - ```sudo cmake --install .``` will install to /usr/local/Qt-6.6.1 (need this for qmake) but could also set path to point into the build directory...
  - ```apt-get install qtchooser```
  - ```qtchooser -install qt6 /usr/local/Qt-6.6.1/bin/qmake```
  - ```export QT_SELECT=qt6``` (you need to do this after each login or make it permanent in .bashrc)
 
## Do libgit Build 
or install `sudo apt-get install libgit2-dev` and skip steps below (but remember to change lib paths for Fritzing build and install the package when deploying)
   - `wget https://github.com/libgit2/libgit2/archive/refs/tags/v1.7.1.tar.gz`
   - untar
   - change to libgit dir
   - `mkdir build && cd build`
   - `cmake ..`
   - `cmake --build . --parallel`
 
 ## Do spiceng Build
Or install shared lib devel package (https://packages.ubuntu.com/search?keywords=ngspice ngspice-dev) and skip steps below (but remember to change lib paths for Fritzing build, remove copy instructions from release script, and install the package when deploying). When installing the distro package, for Ubuntu (22.04) do a `ln -s /usr/lib/x86_64-linux-gnu/libngspice.so.0 libngspice.so` in the `lib` subdir of your Fritzing installation. This actually is the preferred way.

- get sources from https://ngspice.sourceforge.io/download.html
- untar
- rename dir: `mv ngspice-42 ngspice-40` (because of expected folder name by detect script)
- ```sudo apt-get install libxaw7-dev```
- change to unzip dir
- ```./configure --with-ngshared```
- ```make -j```

 ## get/compile quazip
   - `sudo apt-get install zlib1g-dev libbz2-dev`
   - `wget https://github.com/stachenov/quazip/archive/refs/tags/v1.4.tar.gz`
   - untar
   - rename to expected dir name e.g. `mv quazip-1.4 quazip-6.6.1-1.4`, made of Qt version number and expected version of quazip (1.4)
   - cmake needs to be called with path to qt6 files: `cmake -S . -B ./ -D QUAZIP_QT_MAJOR_VERSION=6 -DCMAKE_PREFIX_PATH="/usr/local/Qt-6.6.1/lib/cmake"`
   - `cmake --build ./ --parallel`
     
## do clipper1 lib build
   - get it from sourceforge: `https://sourceforge.net/projects/polyclipping/files/latest/download`
   - `mkdir Clipper1`, UPPERCASE!
   - `mv clipper_ver6.4.2.zip Clipper1`
   - `cd Clipper1`
   - unzip
   - `cd cpp`
   - `mkdir build-dir`
   - `cd build-dir`
   - `cmake ..`
   - `make -j`
   - optional system-wide installation: `sudo make install`, files will be in /usr/local/include/polyclipping and /usr/local/lib
     
## build svgpp lib v1.3.0 (as of writing, matches version expected by Fritzing)
   - `git clone https://github.com/svgpp/svgpp.git`
   - `sudo apt-get install libxml2-dev`
   - `mv svgpp svgpp-1.3.0`
   - `cd svgpp-1.3.0`
   - `mkdir build-dir`
   - `cd build-dir`
   - `cmake -D BOOST_ROOT=../../boost_1_84_0 ../src` (perfectly happy generating GNU makefile)
   - `make -j` (go fast)

## Prepare Fritzing build
  - ```git clone https://github.com/fritzing/fritzing-app.git```
	- optional: ```git clone https://github.com/fritzing/fritzing-parts```
	- change compile script (phoenix.pro):
		- allow for later versions of qt
   	- ```nano ./fritzing-app/phoenix.pro```
   	- change QT_MOST to 6.6.10 (or whatever is sufficiently high)
  - change boost detect script (will only accept up to 81)
    - ```nano ./fritzing-app/pri/boostdetect.pri```
   	- find `BOOSTS = 81`
   	- change to `BOOSTS = 84`
  - fix ngspice detection (will not set correct include dir)
   	- include must point to `../ngspice-40/src/include`but instead only points to `ngspice-40/include`
   	- `nano ./pri/spicedetect.pri`
   	- find `INCLUDEPATH += $$NGSPICEPATH/include`, change to `INCLUDEPATH += $$NGSPICEPATH/src/include`
  - fix clipper detect script:
   	- `nano ./pri/clipper1detect.pri`
   	- change ```CLIPPER1 = $$absolute_path($$PWD/../../Clipper1/6.4.2)``` to be ``` CLIPPER1 = $$absolute_path($$PWD/../../Clipper1)``` (no slash!)
   	- change `LIBS += -L$$absolute_path($${CLIPPER1}/lib) -lpolyclipping` to be ```LIBS += -L$$absolute_path($${CLIPPER1}/cpp/build-dir) -lpolyclipping``` 
   	- change ```INCLUDEPATH += $$absolute_path($${CLIPPER1}/include/polyclipping)``` to be ```INCLUDEPATH += $$absolute_path($${CLIPPER1}/cpp)```
  - fix quazip detect script:
   	- `nano pri/quazipdetect.pri`
   	- change ```QUAZIP_INCLUDE_PATH=$$QUAZIP_PATH/include/QuaZip-Qt6-$$QUAZIP_VERSION```to be ```QUAZIP_INCLUDE_PATH=$$QUAZIP_PATH```
   	- change ```LIBS += -L $$QUAZIP_LIB_PATH -lquazip1-qt$$QT_MAJOR_VERSION``` to be ```LIBS += -L $$QUAZIP_LIB_PATH/quazip -lquazip1-qt$$QT_MAJOR_VERSION -lbz2 -lz```
- Test build
  - `qmake`
  - `make`
- Everything ok? proceed... if not: fix
- `make clean`
- `rm Makefile*`



# Statically linked, Qt-zlib
Well, not entirely static build. Qt, Clipping and SpiceNG will be linked statically, Quazip, svgpp and Git2 will be linked dynamically. Main reasons are: The required version of Qt is still well ahead of most distributions, there are no binaries for the Clipping library. As for SpiceNG: there are binary builds in most distributions but with lower version numbers (Ubuntu 22.04: v36). Quazip must match our version of Qt, so no luck here, either. We can't rely on binaries for most distributions. The main reason for dynamically linking this is linking issues. Same for Git2. But the Git2 library is available in most distirbutions. Same for svgpp-dev.

This leads to an executable linked against (and included) libspice, but you must still provide the .so (because of the way the library is included by Fritzing). So this is somehow instructive but in the end useless..

Qt Shadow build: Keep build artifacts (and resulting binaries) out of the source tree.  

## Do Qt build
- `cd ~/qt-build`
- `mkdir qt-static`
- `cd qt-static`
- ` ../qt-everywhere-src-6.6.1/configure -static -qt-zlib -prefix /opt/Qt6.6.1`
- optional: check config.result for appropriate configuration (X11 / Wayland platform plugins, qt-zlib = yes, shared libraries = no)
- ```cmake --build . --parallel```, libs will be in `~/qt-build/qt-static/qtbase/lib`
- ```sudo cmake --install .``` will install to `/opt/Qt6.6.1`
- ```apt-get install qtchooser```
- ```qtchooser -install qt6 /opt/Qt6.6.1/bin/qmake```
- ```export QT_SELECT=qt6``` (you need to do this after each login or make it permanent in .bashrc)

## Satisfy libgit2 requirement
Install libgit2-dev and libgit2

## Do spiceng Build (static)
- get source, untar (version 42)
- `sudo apt-get install libxaw7-dev automake libtool`
- no: `./configure -enable-static`
- edit `configure.ac`
	- line # 481: replace `AC_SUBST([STATIC], [-shared])` with `AC_SUBST([STATIC], [-static])` as per https://sourceforge.net/p/ngspice/discussion/127605/thread/2216ffc5/
 - edit `~ngspice/src/Makefile.am`
 	- line 634: change `` to static
 	- line 636 change to static and include lib ver
  	- after change, looks like
	```
 	libngspice_la_CFLAGS = -static

	libngspice_la_LDFLAGS =  -static -version-info @LIB_VERSION@
 	```
 - `./compile_linux_shared.sh 64`
 - result will be found in `~/ngspice-42/releasesh/src/.libs` (`libngspice.a`), approx. 18MB, don't worry about failed install (can't write to system dirs without sufficient privileges, obviously...)

Please note that he library gets linked statically (well, no, we're telling the linker to do it but it gets thrown out) but Fritzing still complains about not having access to the shared version of it (in retrospect completely correct because the static lib got thrown out during linking and it insists on loading a shared version: ldd doesn't show any symbols for SPICE, so it must have been included statically. But wth... There still is one thing left to check: Simulation src in Fritzing code). See below comments in FAQ section.

## Do Clipping Library Build (static)
- get it from sourceforge: `https://sourceforge.net/projects/polyclipping/files/latest/download`
- `mkdir Clipper1`, UPPERCASE!
- `mv clipper_ver6.4.2.zip Clipper1`
- `cd Clipper1`
- unzip
- `cd cpp`
- change `cpp/CMakeLists.txt`:
	- find
 		```
   		SET(BUILD_SHARED_LIBS ON CACHE BOOL
    		"Build shared libraries (.dll/.so) instead of static ones (.lib/.a)")
		```  
 	- change to:
  - 		```
   		SET(BUILD_SHARED_LIBS OFF CACHE BOOL
    		"Don't Build shared libraries (.dll/.so) instead of static ones (.lib/.a)")
		```  
- `mkdir build-dir`
- `cd build-dir`
- `cmake ..`
- `make`
- Indicator of success: watch for ```[100%] Linking CXX static library libpolyclipping.a```
- library file will be in `~/Clipper1/cpp/build-dir`

## Do Quazip Build
For dynamic linking, see other instructions.  
See [here for static build](https://github.com/DanielBarie/FritzingBuild/blob/main/README.md#quazip-static-build).

## Install svgpp
`sudo apt-get install libsvgpp-dev`

## Prepare Fritzing build
- ```git clone https://github.com/fritzing/fritzing-app.git```
	- optional: ```git clone https://github.com/fritzing/fritzing-parts```
	- change compile script (phoenix.pro):
		- allow for later versions of qt
   	- ```nano ./fritzing-app/phoenix.pro```
   	- change QT_MOST to 6.6.10 (or whatever is sufficiently high)
- change boost detect script (will only accept up to 81)
    - ```nano ./fritzing-app/pri/boostdetect.pri```
   	- find `BOOSTS = 81`
   	- change to `BOOSTS = 84`
- fix ngspice detection (will not set correct include dir)
   	- include must point to `../ngspice-42/src/include`but instead points to `ngspice-40/include`
   	- `nano ./pri/spicedetect.pri`
   	- find `NGSPICEPATH = ../../ngspice-40`, change to `NGSPICEPATH = ../../ngspice-42`
   	- find `INCLUDEPATH += $$NGSPICEPATH/include`, change to `INCLUDEPATH += $$NGSPICEPATH/src/include`
   	- add (e.g. below NGSPICEPATH directive): ```LIBS += -L$$absolute_path($${NGSPICEPATH}/releasesh/src/.libs) -lngspice``` 
- fix clipper detect script:
   	- `nano ./pri/clipper1detect.pri`
   	- change ```CLIPPER1 = $$absolute_path($$PWD/../../Clipper1/6.4.2)``` to be ``` CLIPPER1 = $$absolute_path($$PWD/../../Clipper1)``` (no slash!)
   	- change `LIBS += -L$$absolute_path($${CLIPPER1}/lib) -lpolyclipping` to be ```LIBS += -L$$absolute_path($${CLIPPER1}/cpp/build-dir) -lpolyclipping``` 
   	- change ```INCLUDEPATH += $$absolute_path($${CLIPPER1}/include/polyclipping)``` to be ```INCLUDEPATH += $$absolute_path($${CLIPPER1}/cpp)```
- fix quazip detect script:
   	- `nano pri/quazipdetect.pri`
   	- change ```QUAZIP_INCLUDE_PATH=$$QUAZIP_PATH/include/QuaZip-Qt6-$$QUAZIP_VERSION```to be ```QUAZIP_INCLUDE_PATH=$$QUAZIP_PATH```
   	- change `QUAZIP_LIB_PATH=$$QUAZIP_PATH/lib` to be `QUAZIP_LIB_PATH=$$QUAZIP_PATH`
   	- change ```LIBS += -L $$QUAZIP_LIB_PATH -lquazip1-qt$$QT_MAJOR_VERSION``` to be ```LIBS += -L $$QUAZIP_LIB_PATH/build-dir/quazip -lquazip1-qt$$QT_MAJOR_VERSION```
- fix libgit detect script:
	- `nano pri/libgit2detect.pri`
 	- comment out conditional for including static lib, just set `LIBGITSTATIC = true`
 	- change `LIBGIT2LIB = $$LIBGITPATH/lib` to be `LIBGIT2LIB = $$LIBGITPATH/build`
- edit project file for platform plugin inclusion:
	- `nano phoenix.pro`
 	- in unix!macx section, below `LIBS += -lz` add new line `QTPLUGIN.platforms = qminimal qxcb qoffscreen qwayland qvnc`
  	- see: https://doc.qt.io/qt-6/qpa.html
- Test build
  - `qmake`
  - `make`
- Everything ok? proceed... if not: fix
  - `make clean`
  - `rm Makefile*`


# Build Release (dynamic linking)
Build releasable compressed file containing (to be done) all required dependencies.
The release script will clone the parts repo and include it.

- `cd tools/linux_release_script`
- `export QT_QPA_PLATFORM=minimal` `offscreen` also does.
- `nano release.sh`
- remove number of parallel jobs limitation (i.e. change `-j16` to `-j`)
- `./release.sh 1.0.2b_develop_private`, we're including the keyword `develop` so we don't have to worry about the repo not being in a clean state. Modify version string to fit your needs.
- `tar -cjf  ./1.0.2b_develop_private.tar.bz2 fritzing-1.0.2b_develop_private.linux.AMD64`: Won't create compressed file because we're not running Travis-CI.
- There's a modified version of the release script in this repo which copies over some (see spiceng) dependencies. Please be aware that the paths to these dependencies are hard-coded for Qt 6.6.1 (and above build paths (qt-build)).

# Deploy Release
Minimum required dependencies / packages if not statically linked.
- libgit2:
	- for Ubuntu, see https://packages.ubuntu.com/search?searchon=sourcenames&keywords=libgit2 for required package
	- Jammy (22.04 LTS): `sudo apt-get install libgit2-1.1`
- libngspice:
	- https://packages.ubuntu.com/search?keywords=ngspice
 	- Jammy (22.04 LTS): `sudo apt-get install libngspice0`

# FAQ
## Complains about missing libgit2.so.1.7 (or any other shared library)
Error message: ```<our installation dir>/lib/Fritzing: error while loading shared libraries: libgit2.so.1.7: cannot open shared object file: No such file or directory```.   
Go do a 
- `cd <our installation dir>/lib`
- `ldd Fritzing`
You'll probably see something like this:
```
linux-vdso.so.1 (0x00007ffc071de000)
	libz.so.1 => /lib/x86_64-linux-gnu/libz.so.1 (0x00007f1fa19cd000)
	libgit2.so.1.7 => not found
	libquazip1-qt6.so.1.4.0 => not found
	libpolyclipping.so.22 => not found
	libQt6PrintSupport.so.6 => not found
	libQt6SvgWidgets.so.6 => not found
	libQt6Widgets.so.6 => not found
	libQt6Svg.so.6 => not found
	libQt6Gui.so.6 => not found
	libQt6Network.so.6 => not found
	libQt6SerialPort.so.6 => not found
	libQt6Sql.so.6 => not found
	libQt6Xml.so.6 => not found
	libQt6Core5Compat.so.6 => not found
	libQt6Core.so.6 => not found
	libstdc++.so.6 => /lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f1fa0a00000)
	libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f1fa0d19000)
	libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f1fa19a7000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f1fa0600000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f1fa19fa000)
```
A ton of libraries "not found". Sure, they have been exluded from linking because the release script is for dynamic linking. We need to fix this...
Edit `phoenix.pro`:
- add line `CONFIG += static`
- change unix `QMAKE_CXXFLAGS` to add `-static` 

Doesn't change much, huh?

The reason can be found when looking at the output of the release script, it fails (pretty much silently): 
```
/Fritzing: error while loading shared libraries: libquazip1-qt6.so.1.4.0: cannot open shared object file: No such file or directory
Failed at 115: ./Fritzing -db "${release_folder}/fritzing-parts/parts.db" -pp "${release_folder}/fritzing-parts" -f "${release_folder}"
```
Do an ldd to see:
```
$fritzing-1.0.2b_develop_private.linux.AMD64/lib$ ldd Fritzing
  linux-vdso.so.1 (0x00007ffd83dbe000)
  libz.so.1 => /lib/x86_64-linux-gnu/libz.so.1 (0x00007fa318dc6000)
  libgit2.so.1.7 => /home/daniel/libgit2-1.7.1/build/libgit2.so.1.7 (0x00007fa318031000)
  libquazip1-qt6.so.1.4.0 => not found
  libpolyclipping.so.22 => not found
  libQt6PrintSupport.so.6 => not found
  libQt6SvgWidgets.so.6 => not found
  libQt6Widgets.so.6 => not found
  libQt6Svg.so.6 => not found
  libQt6Gui.so.6 => not found
  libQt6Network.so.6 => not found
  libQt6SerialPort.so.6 => not found
  libQt6Sql.so.6 => not found
  libQt6Xml.so.6 => not found
  libQt6Core5Compat.so.6 => not found
  libQt6Core.so.6 => not found
  libstdc++.so.6 => /lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007fa317e00000)
  libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007fa317d19000)
  libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007fa318da0000)
  libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fa317a00000)
  libssl.so.3 => /lib/x86_64-linux-gnu/libssl.so.3 (0x00007fa317c75000)
  libcrypto.so.3 => /lib/x86_64-linux-gnu/libcrypto.so.3 (0x00007fa317400000)
  /lib64/ld-linux-x86-64.so.2 (0x00007fa318df3000)
```
So we now have some libraries which are linked statically, some dynamically?
Well, that's how it happens: https://stackoverflow.com/questions/1361229/using-a-static-library-in-qt-creator

There's two possible ways out of this:
- create statically linkable libraries. While easy for the smaller depedencies (quazip etc...) I don't feel like doing it all over for Qt. See: https://doc.qt.io/archives/qt-4.8/deployment-x11.html
- package libraries with app: Copy .so files into the lib dir of the packaged app. Or use https://github.com/probonopd/linuxdeployqt see https://wiki.qt.io/Auto-deploying_your_Qt_Application

### Packaging Libraries with the app
see modified release.sh script.
But: building the parts database needs a GUI? Well, we'll have to run this with a GUI, then.

Qt platform plugins are located at `/usr/local/Qt-6.6.1/plugins/platforms` after building qt. But no wayland, no xcb? check build logs...

copy over sql plugin
copy over image plugins
make sure to have wayland plugin built for ubuntu

export QT_QPA_PLATFORM=minimal works for building the database...

Build database:  ./Fritzing -db "./fritzing-parts/parts.db" -pp "./fritzing-parts" -f "./"

Running the app doesn't work, even with platform plugins copied to correct directory (i.e. /lib/platforms), keeps complaining `Could not load the Qt platform plugin "wayland" in "" ` or whatever was specified.

## Still not loading?
Maybe there's some unmet dependencies for the platform plugin you're trying to use.
Do `export QT_DEBUG_PLUGINS=1` and try running again.
When running the exe, we'll see some more output:
```
qt.qpa.plugin: From 6.5.0, xcb-cursor0 or libxcb-cursor0 is needed to load the Qt xcb platform plugin.
qt.qpa.plugin: Could not load the Qt platform plugin "xcb" in "" even though it was found.
```
Checking for the libraries, they are installed? Must be a problem of the library path, it gets modified in Fritzing.sh. Trying to modify the library path to make sure the (installed) libs get used: ```export LD_LIBRARY_PATH="/usr/lib/x86_64-linux-gnu/:~/fritzing-app/tools/linux_release_script/fritzing-1.0.2b_develop_private.linux.AMD64/lib"```  doesn't change a thing.
Wait, that was a red herring... Having activated plugin debug yields some more information: `libqxcb.so: (libQt6XcbQpa.so.6: Kann die Shared-Object-Datei nicht öffnen: Datei oder Verzeichnis nicht gefunden)"`
Check dependencies:
- `cd ~/qt-build/qt-everywhere-src-6.6.1/qtbase/plugins/platforms`
- ` ldd libqxcb.so | grep qt-build`
- copy over any file listed

## Loading but crashing?
Look for any undefined symbols...
```nm libQt6Network.so | grep z_inflateInit``` will give ```U z_inflateInit2_@Qt_6``` indicating there's an undefined (lowercase u: internal) external (uppercase U) symbol.
Check config.summary for Using System zlib = yes.

Hm. 
```
ldd libQt6Network.so
	linux-vdso.so.1 (0x00007ffce15ca000)
	libQt6Core.so.6 => /home/daniel/fritzing-app/tools/linux_release_script/fritzing-1.0.2b_develop_private.linux.AMD64/lib/libQt6Core.so.6 (0x00007f21a4000000)
	libstdc++.so.6 => /lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f21a3c00000)
	libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f21a47e7000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f21a3800000)
	libicui18n.so.70 => /lib/x86_64-linux-gnu/libicui18n.so.70 (0x00007f21a3400000)
	libicuuc.so.70 => /lib/x86_64-linux-gnu/libicuuc.so.70 (0x00007f21a3205000)
	libz.so.1 => /lib/x86_64-linux-gnu/libz.so.1 (0x00007f21a47c9000)
	libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f21a3f19000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f21a49cf000)
	libicudata.so.70 => /lib/x86_64-linux-gnu/libicudata.so.70 (0x00007f21a1400000)
```
Loads `libz.so.1`. Let's see if theres an appropriate symbol:
```
nm -D /lib/x86_64-linux-gnu/libz.so.1
....
000000000000cd30 T inflateInit2_
...
```
but no `z_inflateInit2_`. 

Well, let's see the version:
```
dpkg -l | grep zlib
ii  zlib1g:amd64                               1:1.2.11.dfsg-2ubuntu9.2                amd64        compression library - runtime
ii  zlib1g-dev:amd64                           1:1.2.11.dfsg-2ubuntu9.2                amd64        compression library - development
```
Hm, 1.2.11. Let's see the latest source: https://github.com/madler/zlib/blob/develop/zconf.h
```
#  define inflateInit2_         z_inflateInit2_
```
Check against version used by Qt 6.6: https://doc.qt.io/qt-6/qtcore-attribution-zlib.html
```
Data Compression Library (zlib), version 1.3
```
Here we are... So there's some possible solutions:
- Maybe change to Qt internal zlib. I don't feel like doing this. But it works. Did it.
- Compile later version of zlib and include in package. Make sure to include the z_ prefix 


## Static Qt build with internal zlib
Leads to success regarding missing `z_...` symbols.

'./configure -static -prefix ~/qt-build//qt-everywhere-src-6.6.1 -qt-zlib'
```
nm ./qtbase/lib/libQt6BundledZLIB.a
...
0000000000000420 T z_inflateInit_
0000000000000330 T z_inflateInit2_
...
```
Hm. 
```
 ldd  ./qtbase/lib/libQt6Network.so.6.6.1
        linux-vdso.so.1 (0x00007ffd47fe8000)
        libQt6Core.so.6 => not found
        libbrotlidec.so.1 => /lib/x86_64-linux-gnu/libbrotlidec.so.1 (0x00007f463371a000)
        libstdc++.so.6 => /lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f4633400000)
        libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f46336fa000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f4633000000)
        libbrotlicommon.so.1 => /lib/x86_64-linux-gnu/libbrotlicommon.so.1 (0x00007f46336d5000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f4633319000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f46338f1000)
```
So when building with internal zlib, the z_ prefix will be used and the network lib will be happily including the internal zlib.

## The heck, z_... 
See zlib header file:
Why would someone do that?
```
#define ZCONF_H

/*
 * If you *really* need a unique prefix for all types and library functions,
 * compile with -DZ_PREFIX. The "standard" zlib should be compiled without it.
 * Even better than compiling with -DZ_PREFIX would be to use configure to set
 * this permanently in zconf.h using "./configure --zprefix".
 */
#ifdef Z_PREFIX     /* may be set to #if 1 by ./configure */
#  define Z_PREFIX_SET
```

## Issues when trying to statically link libgit2 and quazip
```
/usr/bin/ld: /home/daniel/libgit2-1.7.1/build/libgit2.a(regexp.c.o): warning: relocation against `pcre_free' in read-only section `.text'
/usr/bin/ld: /home/daniel/libgit2-1.7.1/build/libgit2.a(regexp.c.o): in function `git_regexp_compile':
/home/daniel/libgit2-1.7.1/src/util/regexp.c:20: undefined reference to `pcre_compile'
/usr/bin/ld: /home/daniel/libgit2-1.7.1/build/libgit2.a(regexp.c.o): in function `git_regexp_dispose':
/home/daniel/libgit2-1.7.1/src/util/regexp.c:30: undefined reference to `pcre_free'
/usr/bin/ld: /home/daniel/libgit2-1.7.1/build/libgit2.a(regexp.c.o): in function `git_regexp_match':
/home/daniel/libgit2-1.7.1/src/util/regexp.c:37: undefined reference to `pcre_exec'
/usr/bin/ld: /home/daniel/libgit2-1.7.1/build/libgit2.a(regexp.c.o): in function `git_regexp_search':
/home/daniel/libgit2-1.7.1/src/util/regexp.c:55: undefined reference to `pcre_exec'
/usr/bin/ld: /home/daniel/quazip-6.6.1-1.4/build-dir/quazip/libquazip1-qt6.a(unzip.c.o): in function `unzReadCurrentFile':
unzip.c:(.text.unzReadCurrentFile+0x311): undefined reference to `BZ2_bzDecompress'
/usr/bin/ld: /home/daniel/quazip-6.6.1-1.4/build-dir/quazip/libquazip1-qt6.a(unzip.c.o): in function `unzCloseCurrentFile':
unzip.c:(.text.unzCloseCurrentFile+0xbd): undefined reference to `BZ2_bzDecompressEnd'
/usr/bin/ld: /home/daniel/quazip-6.6.1-1.4/build-dir/quazip/libquazip1-qt6.a(unzip.c.o): in function `unzOpenCurrentFile3':
unzip.c:(.text.unzOpenCurrentFile3+0x6bd): undefined reference to `BZ2_bzDecompressInit'
/usr/bin/ld: /home/daniel/quazip-6.6.1-1.4/build-dir/quazip/libquazip1-qt6.a(zip.c.o): in function `zipWriteInFileInZip':
zip.c:(.text.zipWriteInFileInZip+0x1f1): undefined reference to `BZ2_bzCompress'
/usr/bin/ld: /home/daniel/quazip-6.6.1-1.4/build-dir/quazip/libquazip1-qt6.a(zip.c.o): in function `zipCloseFileInZipRaw64':
zip.c:(.text.zipCloseFileInZipRaw64+0x3b0): undefined reference to `BZ2_bzCompress'
/usr/bin/ld: zip.c:(.text.zipCloseFileInZipRaw64+0x415): undefined reference to `BZ2_bzCompressEnd'
/usr/bin/ld: zip.c:(.text.zipCloseFileInZipRaw64+0x4c3): undefined reference to `BZ2_bzCompressEnd'
/usr/bin/ld: zip.c:(.text.zipCloseFileInZipRaw64+0x667): undefined reference to `BZ2_bzCompressEnd'
/usr/bin/ld: /home/daniel/quazip-6.6.1-1.4/build-dir/quazip/libquazip1-qt6.a(zip.c.o): in function `zipOpenNewFileInZip4_64':
zip.c:(.text.zipOpenNewFileInZip4_64+0xe28): undefined reference to `BZ2_bzCompressInit'
```
Which indicates an issue with the order of linking.
We have
```
g++ -Wl,-O1 -Wl,-rpath,/home/daniel/quazip-6.6.1-1.4 -Wl,-rpath,/home/daniel/Clipper1/lib -o Fritzing  release/zlibdummy.o release/commands.o release/debugdialog.o release/fapplication.o release/fsplashscreen.o release/fsvgrenderer.o release/itemdrag.o release/layerattributes.o release/main.o release/processeventblocker.o release/sketchtoolbutton.o release/viewgeometry.o release/viewlayer.o release/waitpushundostack.o release/project_properties.o release/fdockwidget.o release/fritzingwindow.o release/mainwindow.o release/mainwindow_export.o release/mainwindow_menu.o release/mainwindow_dock.o release/sketchareawidget.o release/FProbeDropByModuleID.o release/FProbeKeyPressEvents.o release/getspice.o release/partsbinpalettewidget.o release/partsbinview.o release/partsbinlistview.o release/partsbiniconview.o release/graphicsflowlayout.o release/svgiconwidget.o release/partsbincommands.o release/searchlineedit.o release/binmanager.o release/stacktabbar.o release/stacktabwidget.o release/pemainwindow.o release/pemetadataview.o release/pecommands.o release/peconnectorsview.o release/pesvgview.o release/petoolview.o release/peutils.o release/pegraphicsitem.o release/kicadmoduledialog.o release/hashpopulatewidget.o release/sqlitereferencemodel.o release/svgfilesplitter.o release/svgpathparser.o release/svgpathgrammar.o release/svgpathlexer.o release/svgpathrunner.o release/svg2gerber.o release/svgflattener.o release/gerbergenerator.o release/groundplanegenerator.o release/groundplanegeneratorold.o release/x2svg.o release/kicad2svg.o release/kicadmodule2svg.o release/kicadschematic2svg.o release/gedaelement2svg.o release/gedaelementparser.o release/gedaelementgrammar.o release/gedaelementlexer.o release/svgtext.o release/aboutbox.o release/firsttimehelpdialog.o release/tipsandtricks.o release/modfiledialog.o release/updatedialog.o release/version.o release/versionchecker.o release/partschecker.o release/fritzing2eagle.o release/ftooltip.o release/autoclosemessagebox.o release/bendpointaction.o release/bezier.o release/bezierdisplay.o release/clickablelabel.o release/cursormaster.o release/expandinglabel.o release/fileprogressdialog.o release/flineedit.o release/fmessagebox.o release/focusoutcombobox.o release/fsizegrip.o release/lockmanager.o release/misc.o release/resizehandle.o release/folderutils.o release/graphicsutils.o release/graphutils.o release/ratsnestcolors.o release/schematicrectconstants.o release/s2s.o release/textutils.o release/zoomslider.o release/FMessageLogProbe.o release/uploadpair.o release/layerpalette.o release/breadboard.o release/capacitor.o release/clipablewire.o release/dip.o release/groundplane.o release/hole.o release/itembase.o release/jumperitem.o release/layerkinpaletteitem.o release/led.o release/logoitem.o release/moduleidnames.o release/mysterypart.o release/note.o release/pad.o release/paletteitem.o release/paletteitembase.o release/partfactory.o release/partlabel.o release/perfboard.o release/pinheader.o release/propertydef.o release/resistor.o release/resizableboard.o release/ruler.o release/schematicframe.o release/schematicsubpart.o release/screwterminal.o release/stripboard.o release/symbolpaletteitem.o release/tracewire.o release/via.o release/virtualwire.o release/wire.o release/FProbeR1PosPCB.o release/FProbeRPartLabel.o release/FProbeSwitchPackage.o release/autorouter.o release/autorouteprogressdialog.o release/autoroutersettingsdialog.o release/checker.o release/Rect.o release/GuillotineBinPack.o release/mazerouter.o release/zoomcontrols.o release/drc.o release/prefsdialog.o release/exportparametersdialog.o release/pinlabeldialog.o release/groundfillseeddialog.o release/quotedialog.o release/recoverydialog.o release/setcolordialog.o release/translatorlistmodel.o release/fabuploaddialog.o release/fabuploadprogress.o release/networkhelper.o release/fabuploadprogressprobe.o release/ipc_d_356.o release/debugconnectorsprobe.o release/bus.o release/busshared.o release/connector.o release/connectoritem.o release/nonconnectoritem.o release/connectorshared.o release/ercdata.o release/svgidlayer.o release/debugconnectors.o release/htmlinfoview.o release/scalediconframe.o release/modelbase.o release/modelpart.o release/modelpartshared.o release/palettemodel.o release/sketchmodel.o release/renderthing.o release/fgraphicsscene.o release/breadboardsketchwidget.o release/infographicsview.o release/pcbsketchwidget.o release/schematicsketchwidget.o release/sketchwidget.o release/welcomeview.o release/zoomablegraphicsview.o release/highlighter.o release/programtab.o release/programwindow.o release/syntaxer.o release/trienode.o release/console.o release/consolewindow.o release/consolesettings.o release/platform.o release/platformarduino.o release/platformpicaxe.o release/platformlaunchpad.o release/FTesting.o release/FProbe.o release/FTestingServer.o release/FProbeStartSimulator.o release/simulator.o release/ngspice_simulator.o release/cppversion.o release/fritzing_plugin_import.o release/qrc_phoenixresources.o release/moc_commands.o release/moc_debugdialog.o release/moc_fapplication.o release/moc_fsplashscreen.o release/moc_fsvgrenderer.o release/moc_itemdrag.o release/moc_sketchtoolbutton.o release/moc_viewlayer.o release/moc_waitpushundostack.o release/moc_fdockwidget.o release/moc_fritzingwindow.o release/moc_mainwindow.o release/moc_sketchareawidget.o release/moc_FProbeDropByModuleID.o release/moc_FProbeKeyPressEvents.o release/moc_partsbinpalettewidget.o release/moc_partsbinlistview.o release/moc_partsbiniconview.o release/moc_svgiconwidget.o release/moc_searchlineedit.o release/moc_binmanager.o release/moc_stacktabbar.o release/moc_stacktabwidget.o release/moc_pemainwindow.o release/moc_pemetadataview.o release/moc_peconnectorsview.o release/moc_pesvgview.o release/moc_petoolview.o release/moc_pegraphicsitem.o release/moc_kicadmoduledialog.o release/moc_hashpopulatewidget.o release/moc_baseremovebutton.o release/moc_sqlitereferencemodel.o release/moc_referencemodel.o release/moc_svgfilesplitter.o release/moc_svgpathrunner.o release/moc_svg2gerber.o release/moc_svgflattener.o release/moc_groundplanegenerator.o release/moc_groundplanegeneratorold.o release/moc_aboutbox.o release/moc_firsttimehelpdialog.o release/moc_tipsandtricks.o release/moc_modfiledialog.o release/moc_updatedialog.o release/moc_versionchecker.o release/moc_autoclosemessagebox.o release/moc_bendpointaction.o release/moc_boundedregexpvalidator.o release/moc_clickablelabel.o release/moc_cursormaster.o release/moc_expandinglabel.o release/moc_familypropertycombobox.o release/moc_fileprogressdialog.o release/moc_flineedit.o release/moc_fmessagebox.o release/moc_focusoutcombobox.o release/moc_fsizegrip.o release/moc_lockmanager.o release/moc_resizehandle.o release/moc_s2s.o release/moc_zoomslider.o release/moc_layerpalette.o release/moc_breadboard.o release/moc_capacitor.o release/moc_clipablewire.o release/moc_dip.o release/moc_groundplane.o release/moc_hole.o release/moc_itembase.o release/moc_jumperitem.o release/moc_layerkinpaletteitem.o release/moc_led.o release/moc_logoitem.o release/moc_mysterypart.o release/moc_note.o release/moc_pad.o release/moc_paletteitem.o release/moc_paletteitembase.o release/moc_partlabel.o release/moc_perfboard.o release/moc_pinheader.o release/moc_resistor.o release/moc_resizableboard.o release/moc_ruler.o release/moc_schematicframe.o release/moc_schematicsubpart.o release/moc_screwterminal.o release/moc_stripboard.o release/moc_symbolpaletteitem.o release/moc_tracewire.o release/moc_via.o release/moc_virtualwire.o release/moc_wire.o release/moc_autorouter.o release/moc_autorouteprogressdialog.o release/moc_autoroutersettingsdialog.o release/moc_mazerouter.o release/moc_zoomcontrols.o release/moc_drc.o release/moc_prefsdialog.o release/moc_exportparametersdialog.o release/moc_pinlabeldialog.o release/moc_groundfillseeddialog.o release/moc_quotedialog.o release/moc_recoverydialog.o release/moc_setcolordialog.o release/moc_translatorlistmodel.o release/moc_fabuploaddialog.o release/moc_fabuploadprogress.o release/moc_fabuploadprogressprobe.o release/moc_debugconnectorsprobe.o release/moc_bus.o release/moc_connector.o release/moc_connectoritem.o release/moc_nonconnectoritem.o release/moc_connectorshared.o release/moc_debugconnectors.o release/moc_htmlinfoview.o release/moc_scalediconframe.o release/moc_modelbase.o release/moc_modelpart.o release/moc_modelpartshared.o release/moc_palettemodel.o release/moc_sketchmodel.o release/moc_fgraphicsscene.o release/moc_breadboardsketchwidget.o release/moc_infographicsview.o release/moc_pcbsketchwidget.o release/moc_schematicsketchwidget.o release/moc_sketchwidget.o release/moc_welcomeview.o release/moc_zoomablegraphicsview.o release/moc_highlighter.o release/moc_programtab.o release/moc_programwindow.o release/moc_syntaxer.o release/moc_console.o release/moc_consolewindow.o release/moc_consolesettings.o release/moc_platform.o release/moc_platformarduino.o release/moc_platformpicaxe.o release/moc_platformlaunchpad.o release/moc_FTesting.o release/moc_FTestingServer.o release/moc_FProbeStartSimulator.o release/moc_simulator.o   -lz /home/daniel/libgit2-1.7.1/build/libgit2.a -L/home/daniel/ngspice-42/releasesh/src/.libs -lngspice -L /home/daniel/quazip-6.6.1-1.4/build-dir/quazip -L/home/daniel/quazip-6.6.1-1.4 -lquazip1-qt6 -L/home/daniel/Clipper1/cpp/build-dir -lpolyclipping -lssl -lcrypto /opt/Qt6.6.1/lib/objects-Release/Gui_resources_1/.rcc/qrc_qpdf.cpp.o /opt/Qt6.6.1/lib/objects-Release/Gui_resources_2/.rcc/qrc_gui_shaders.cpp.o /opt/Qt6.6.1/plugins/platforms/libqxcb.a /opt/Qt6.6.1/plugins/xcbglintegrations/libqxcb-egl-integration.a /opt/Qt6.6.1/plugins/xcbglintegrations/libqxcb-glx-integration.a /opt/Qt6.6.1/lib/libQt6XcbQpa.a -lxkbcommon-x11 -lxcb-cursor -lxcb-icccm -lxcb-image -lxcb-keysyms -lxcb-randr -lxcb-render-util -lxcb-shm -lxcb-sync -lxcb-xfixes -lxcb-render -lxcb-shape -lxcb-xkb -lSM -lICE -lxcb-glx /opt/Qt6.6.1/plugins/iconengines/libqsvgicon.a /opt/Qt6.6.1/plugins/imageformats/libqgif.a /opt/Qt6.6.1/plugins/imageformats/libqicns.a /opt/Qt6.6.1/plugins/imageformats/libqico.a /opt/Qt6.6.1/plugins/imageformats/libqjpeg.a /opt/Qt6.6.1/lib/libQt6BundledLibjpeg.a /opt/Qt6.6.1/plugins/imageformats/libqsvg.a /opt/Qt6.6.1/plugins/imageformats/libqtga.a /opt/Qt6.6.1/plugins/imageformats/libqtiff.a /opt/Qt6.6.1/plugins/imageformats/libqwbmp.a /opt/Qt6.6.1/plugins/imageformats/libqwebp.a /opt/Qt6.6.1/lib/objects-Release/EglFSDeviceIntegrationPrivate_resources_1/.rcc/qrc_cursor.cpp.o /opt/Qt6.6.1/plugins/egldeviceintegrations/libqeglfs-emu-integration.a /opt/Qt6.6.1/plugins/egldeviceintegrations/libqeglfs-kms-egldevice-integration.a /opt/Qt6.6.1/lib/libQt6EglFsKmsSupport.a /opt/Qt6.6.1/lib/libQt6KmsSupport.a -ldrm /opt/Qt6.6.1/plugins/egldeviceintegrations/libqeglfs-x11-integration.a /opt/Qt6.6.1/lib/libQt6EglFSDeviceIntegration.a /opt/Qt6.6.1/lib/libQt6FbSupport.a /opt/Qt6.6.1/lib/libQt6InputSupport.a /opt/Qt6.6.1/lib/libQt6DeviceDiscoverySupport.a /opt/Qt6.6.1/lib/libQt6OpenGL.a -lX11-xcb -lxcb /opt/Qt6.6.1/plugins/networkinformation/libqnetworkmanager.a /opt/Qt6.6.1/plugins/tls/libqopensslbackend.a /opt/Qt6.6.1/plugins/sqldrivers/libqsqlite.a /opt/Qt6.6.1/lib/objects-Release/PrintSupport_resources_1/.rcc/qrc_qprintdialog.cpp.o /opt/Qt6.6.1/lib/objects-Release/PrintSupport_resources_2/.rcc/qrc_qprintdialog1.cpp.o /opt/Qt6.6.1/lib/objects-Release/Widgets_resources_1/.rcc/qrc_qstyle.cpp.o /opt/Qt6.6.1/lib/objects-Release/Widgets_resources_2/.rcc/qrc_qstyle1.cpp.o /opt/Qt6.6.1/lib/objects-Release/Widgets_resources_3/.rcc/qrc_qmessagebox.cpp.o /opt/Qt6.6.1/lib/libQt6PrintSupport.a /opt/Qt6.6.1/lib/libQt6SvgWidgets.a /opt/Qt6.6.1/lib/libQt6Widgets.a /opt/Qt6.6.1/lib/libQt6Svg.a /opt/Qt6.6.1/lib/libQt6Gui.a -lEGL /opt/Qt6.6.1/lib/libQt6BundledLibpng.a /opt/Qt6.6.1/lib/libQt6BundledHarfbuzz.a -lfreetype -lfontconfig -lX11 /opt/Qt6.6.1/lib/libQt6DBus.a -lxkbcommon /opt/Qt6.6.1/lib/libQt6Concurrent.a /opt/Qt6.6.1/lib/libQt6Network.a -lbrotlidec -lresolv /opt/Qt6.6.1/lib/libQt6SerialPort.a -ludev /opt/Qt6.6.1/lib/libQt6Sql.a /opt/Qt6.6.1/lib/libQt6Xml.a /opt/Qt6.6.1/lib/libQt6Core5Compat.a /opt/Qt6.6.1/lib/libQt6Core.a /opt/Qt6.6.1/lib/libQt6BundledZLIB.a -lm /opt/Qt6.6.1/lib/libQt6BundledPcre2.a -ldl -lrt -lpthread -lGLX -lOpenGL
```
This specifies libgit2 before `/opt/Qt6.6.1/lib/libQt6BundledPcre2.a`. Can't be the reason (correct order, required symbols first, then lib providing these)... Fix order of linking. Can be seen in the Makefile, there's a ton of Qt PRIs, then there are the fritzing PRIs, then Qt PRI's... Haven't had the time to figure this out, yet.

Quick and dirty fix:
- edit phoenix.pro
	- add `QMAKE_LFLAGS += -Wl,--start-group` at the top of the file. Will instruct the linker to sort out circular references, it will complain about a missing `--end-group` but will add that automatically at the end.

Proper fix would be re-odering. Now there's:
```
-L /home/daniel/quazip-6.6.1-1.4/build-dir/quazip -lbz2 -L/home/daniel/quazip-6.6.1-1.4 -lquazip1-qt6 
```
Should be (-lbz2 after quazip because quazip depends on bz2:
```
-L /home/daniel/quazip-6.6.1-1.4/build-dir/quazip -L/home/daniel/quazip-6.6.1-1.4 -lquazip1-qt6  -lbz2
```

## Platform Plugin Errors when building Fritzing Release
The release script will build the parts database. For this, Fritzing will be launched with `-db` parameter. Since your (at least mine) development machine probably doesn't have a graphical user interface installed (or available when you're building over ssh / remote shell), the platform plugin is set to be `minimal` or `offscreen` (in our version of the release script). This of course needs to be linked into the binary (when doing a static build) or available in the `platforms` directory (dynamic linking or static when having built Qt with default parameters, which will be pulled automatically from your build machine config).
```
qt.qpa.plugin: Could not find the Qt platform plugin "minimal" in ""
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.

Available platform plugins are: xcb.

Aborted (core dumped)
```  

Solutions are to  

 - either do a Qt build with platform parameters specified explicitly (https://forum.qt.io/topic/105059/static-qt-build-with-multiple-qpa-plugins/3), we do this with our custom `phoenix.pro`
 - or provide the necessary plugins in the `platforms` directory of the release (same as for dynamic linking)

## libgit2 Build (static)
either do wget or git pull of  https://github.com/libgit2/libgit2/pull/6471
wget way below
- `cd ~`
- `wget https://github.com/libgit2/libgit2/archive/refs/tags/v1.7.1.tar.gz`
- `tar xzvf v1.7.1.tar.gz`
- `cd libgit2-1.7.1/`
- change `CMakeLists.txt` to generate static lib
	- change `option(BUILD_SHARED_LIBS       "Build Shared Library (OFF for Static)"                  ON)` to `option(BUILD_SHARED_LIBS       "Build Shared Library (OFF for Static)"                  OFF)`
 - `mkdir build && cd build`
 - `cmake ..`
 - `cmake --build . --parallel`
 - optional: check for `libgit2.a` being present in build dir `ls -lsah libgit2.a`
 - lib will be in `~/libgit2-1.7.1/build
 - edit `pri/libgit2detect.pri`
 	- in unix branch, static case, change `LIBS += $$LIBGIT2LIB/libgit2.a -lssl -lcrypto` to be `LIBS += $$LIBGIT2LIB/libgit2.a -lssl -lcrypto -lpcre`
  - apply quick and dirty fix (see above) to `phoenix.pro`

## Quazip static Build
- `sudo apt-get install zlib1g-dev libbz2-dev`
- `wget https://github.com/stachenov/quazip/archive/refs/tags/v1.4.tar.gz`
- untar
- rename to expected dir name e.g. `mv quazip-1.4 quazip-6.6.1-1.4`, made of Qt version number and expected version of quazip (1.4)
- `cd quazip-6.6.1-1.4`
- edit `CMakeLists.txt`
	- find (in the else case, not EMSCRIPTEN) `option(BUILD_SHARED_LIBS "" ON)`
 	- change to `option(BUILD_SHARED_LIBS "" OFF)`
  	- while you're at it, you may also disable the install (next line)
- `cd build-dir`
- cmake needs to be called with path to qt6 files: `cmake -S .. -B ./ -D QUAZIP_QT_MAJOR_VERSION=6 -DCMAKE_PREFIX_PATH="/opt/Qt6.6.1/lib/cmake"`
- `cmake --build .`
- indicator of success: look for ```[100%] Linking CXX static library libquazip1-qt6.a```
- library (`libquazip1-qt6.a`) will be in `~/quazip-6.6.1-1.4/build-dir/quazip`
- edit `pri/quazipdetect.pri`
	- add `LIBS += -lbz2` below `LIBS += -L $$QUAZIP_LIB_PATH/build-dir/quazip -lquazip1-qt$$QT_MAJOR_VERSION`

## Static Quazip Build complaining about missing BZ_ symbols:
We obviously forgot to specify the bz2 lib when linking.
Find and include it:
- `dpkg -l | grep bz2` (find package name, if we previously installed it)
- if empty: install: `sudo apt-get install libbz2-1.0`
- `dpkg -L libbz2-dev | grep bz2` for location
- include location when linking (Ubuntu 22.04): add `-L /usr/lib/x86_64-linux-gnu -l bz2` in quazipdetect.pri
- https://stackoverflow.com/questions/12718126/how-do-i-enforce-the-order-of-qmake-library-dependencies
- see above for adding fix for dependencies (issues when statically linking...)

## Static SVGPP Library Build 
Build svgpp lib v1.3.0 (as of writing, matches version expected by Fritzing)
- `git clone https://github.com/svgpp/svgpp.git`
- `sudo apt-get install libxml2-dev`
- `mv svgpp svgpp-1.3.0`
- `cd svgpp-1.3.0`
- `mkdir build-dir`
- `cd build-dir`
- `cmake -D BOOST_ROOT=../../boost_1_84_0 ../src` (perfectly happy generating GNU makefile)
- `make -j` (go fast)

## Release Build works, but stops at building database (no output)?
You're probably building on a machine without GUI or via a console connection. 
There probably is some sort of error in the parts repository. With the current git version 1.0.2b there's an error in a part.  
This will cause an error message to be displayed.  

![grafik](https://github.com/DanielBarie/FritzingBuild/assets/73287620/9016ddfc-0347-4d56-8b29-d1dbf967e04a)   

That only works when a GUI is available. We've set the platform to be `offscreen`. So we're now in a situation where there is an error message we can't see. 
Solution: Get the command line used to launch the database build (`ps -ef`, copy it some place). Kill the process, launch it with a working graphical display. You may then acknowledge the error, the database build will continue. Tar/bzip2 manually.

## What are the dependencies of the static build (X11)?
Only standard libraries, no Qt or SpiceNG.
```
linux-vdso.so.1 (0x00007ffd103f8000)
libz.so.1 => /lib/x86_64-linux-gnu/libz.so.1 (0x00007f2120d8a000)
libpcre.so.3 => /lib/x86_64-linux-gnu/libpcre.so.3 (0x00007f2120d14000)
libbz2.so.1.0 => /lib/x86_64-linux-gnu/libbz2.so.1.0 (0x00007f2120d01000)
libssl.so.3 => /lib/x86_64-linux-gnu/libssl.so.3 (0x00007f211e35c000)
libcrypto.so.3 => /lib/x86_64-linux-gnu/libcrypto.so.3 (0x00007f211de00000)
libxkbcommon-x11.so.0 => /lib/x86_64-linux-gnu/libxkbcommon-x11.so.0 (0x00007f2120cf4000)
libxcb-cursor.so.0 => /lib/x86_64-linux-gnu/libxcb-cursor.so.0 (0x00007f211da00000)
libxcb-icccm.so.4 => /lib/x86_64-linux-gnu/libxcb-icccm.so.4 (0x00007f2120ced000)
libxcb-image.so.0 => /lib/x86_64-linux-gnu/libxcb-image.so.0 (0x00007f2120ce7000)
libxcb-keysyms.so.1 => /lib/x86_64-linux-gnu/libxcb-keysyms.so.1 (0x00007f2120ce2000)
libxcb-randr.so.0 => /lib/x86_64-linux-gnu/libxcb-randr.so.0 (0x00007f2120ccf000)
libxcb-render-util.so.0 => /lib/x86_64-linux-gnu/libxcb-render-util.so.0 (0x00007f2120cc6000)
libxcb-shm.so.0 => /lib/x86_64-linux-gnu/libxcb-shm.so.0 (0x00007f2120cc1000)
libxcb-sync.so.1 => /lib/x86_64-linux-gnu/libxcb-sync.so.1 (0x00007f211e352000)
libxcb-xfixes.so.0 => /lib/x86_64-linux-gnu/libxcb-xfixes.so.0 (0x00007f211e348000)
libxcb-render.so.0 => /lib/x86_64-linux-gnu/libxcb-render.so.0 (0x00007f211e339000)
libxcb-shape.so.0 => /lib/x86_64-linux-gnu/libxcb-shape.so.0 (0x00007f211e334000)
libxcb-xkb.so.1 => /lib/x86_64-linux-gnu/libxcb-xkb.so.1 (0x00007f211e316000)
libSM.so.6 => /lib/x86_64-linux-gnu/libSM.so.6 (0x00007f211e30b000)
libICE.so.6 => /lib/x86_64-linux-gnu/libICE.so.6 (0x00007f211e2ee000)
libxcb-glx.so.0 => /lib/x86_64-linux-gnu/libxcb-glx.so.0 (0x00007f211e2d1000)
libdrm.so.2 => /lib/x86_64-linux-gnu/libdrm.so.2 (0x00007f211e2bb000)
libX11-xcb.so.1 => /lib/x86_64-linux-gnu/libX11-xcb.so.1 (0x00007f211e2b6000)
libxcb.so.1 => /lib/x86_64-linux-gnu/libxcb.so.1 (0x00007f211e28c000)
libEGL.so.1 => /lib/x86_64-linux-gnu/libEGL.so.1 (0x00007f211e279000)
libfreetype.so.6 => /lib/x86_64-linux-gnu/libfreetype.so.6 (0x00007f211dd38000)
libfontconfig.so.1 => /lib/x86_64-linux-gnu/libfontconfig.so.1 (0x00007f211dcee000)
libX11.so.6 => /lib/x86_64-linux-gnu/libX11.so.6 (0x00007f211d8c0000)
libxkbcommon.so.0 => /lib/x86_64-linux-gnu/libxkbcommon.so.0 (0x00007f211dca7000)
libbrotlidec.so.1 => /lib/x86_64-linux-gnu/libbrotlidec.so.1 (0x00007f211e269000)
libudev.so.1 => /lib/x86_64-linux-gnu/libudev.so.1 (0x00007f211dc7d000)
libGLX.so.0 => /lib/x86_64-linux-gnu/libGLX.so.0 (0x00007f211dc49000)
libOpenGL.so.0 => /lib/x86_64-linux-gnu/libOpenGL.so.0 (0x00007f211dc1d000)
libstdc++.so.6 => /lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f211d600000)
libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f211d519000)
libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f211e247000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f211d200000)
/lib64/ld-linux-x86-64.so.2 (0x00007f2120db7000)
libxcb-util.so.1 => /lib/x86_64-linux-gnu/libxcb-util.so.1 (0x00007f211dc14000)
libuuid.so.1 => /lib/x86_64-linux-gnu/libuuid.so.1 (0x00007f211dc0b000)
libbsd.so.0 => /lib/x86_64-linux-gnu/libbsd.so.0 (0x00007f211d8a8000)
libXau.so.6 => /lib/x86_64-linux-gnu/libXau.so.6 (0x00007f211d8a2000)
libXdmcp.so.6 => /lib/x86_64-linux-gnu/libXdmcp.so.6 (0x00007f211d89a000)
libGLdispatch.so.0 => /lib/x86_64-linux-gnu/libGLdispatch.so.0 (0x00007f211d461000)
libpng16.so.16 => /lib/x86_64-linux-gnu/libpng16.so.16 (0x00007f211d85f000)
libexpat.so.1 => /lib/x86_64-linux-gnu/libexpat.so.1 (0x00007f211d82e000)
libbrotlicommon.so.1 => /lib/x86_64-linux-gnu/libbrotlicommon.so.1 (0x00007f211d43e000)
libmd.so.0 => /lib/x86_64-linux-gnu/libmd.so.0 (0x00007f211d431000)
```

## The Simulator isn't available?
![grafik](https://github.com/DanielBarie/FritzingBuild/assets/73287620/2d124bdc-7271-451e-a7b2-21252793f6fa)

In Release builds, you need to activate it.
Go to Edit -> Settings -> Beta Features, set checkbox.
![grafik](https://github.com/DanielBarie/FritzingBuild/assets/73287620/3e9c4139-41ad-4271-81ab-a8c06a9b4954)  

You might want to set the Gerber checkbox as well.

## Spice, Spice, Spice..
libngspice gets compiled into a static library (`libngspice.a`) and successfully linked (statically) into Fritzing binary. But Fritzing still wants to load the shared version?

Check exports of compiled static library: 
`nm libngspice.a | grep ngSpice_Init` should give 
```
0000000000000cd0 T ngSpice_Init
0000000000000360 T ngSpice_Init_Evt
00000000000001b0 T ngSpice_Init_Sync
```
Good. 
But then:
`nm Fritzing | grep spice` does come up empty. Must have gone AWOL during linking.  
Maybe try -rdynamic flag when linking? Doesn't work. Lets try including the whole archive during linking. Modify `pri/spicedetect.pri` to have 
```
LIBS += -Wl,--whole-archive $$absolute_path($${NGSPICEPATH}/releasesh/src/.libs/libngspice.a) -Wl,--no-whole-archive
```
Linking throws a ton of errors related to openMP, e.g.:
``` 
usr/bin/ld: /home/daniel/ngspice-42/releasesh/src/.libs/libngspice.a(osdiload.o): in function `OSDIload._omp_fn.0':
osdiload.c:(.text+0x1a8): undefined reference to `GOMP_barrier
```
Which means we've successfully forced the library to be linked. (Still haven't found out why it gets thrown out in the first place.)  
Add `-fopen` flag in `phoenix.pro`:
```
QMAKE_CXXFLAGS += -O3 -fno-omit-frame-pointer -fopenmp
QMAKE_LFLAGS += -fopenmp
QMAKE_LFLAGS += -Wl,--start-group
``` 
Well, now we have an executable size of approx. 249MB. And 
```nm ./Fritzing | grep ngSpice```
is looking great:
```
0000000000966210 T ngSpice_AllEvtNodes
0000000000966100 T ngSpice_AllPlots
00000000009672e0 T ngSpice_AllVecs
0000000000967160 T ngSpice_Circ
0000000000967780 T ngSpice_Command
00000000009660f0 T ngSpice_CurPlot
0000000000966b40 T ngSpice_Init
00000000009661d0 T ngSpice_Init_Evt
0000000000966020 T ngSpice_Init_Sync
0000000000966220 T ngSpice_LockRealloc
0000000000966000 T ngSpice_running
00000000009673e0 T ngSpice_SetBkpt
0000000000966240 T ngSpice_UnlockRealloc
```

So why on earth don't these get used. Well, use the source, Luke. See: `src/simulation/ngspice_simulator.cpp`:
```
void NgSpiceSimulator::init() {
        if (m_isInitialized) return;

        m_library.setFileName("ngspice");
        m_library.load();

        QStringList libPaths = QStringList({ QCoreApplication::applicationDirPath()
                        })
                        // TODO Not sure if we can place the library there on macOS
                        + QStandardPaths::standardLocations(QStandardPaths::AppLocalDataLocation);

        if( !m_library.isLoaded() ) {         // fallback custom paths
        #ifdef Q_OS_LINUX
                const QString libName = "libngspice.so";
        #elif defined Q_OS_MAC
                const QString libName = "libngspice.0.dylib";
        #elif defined Q_OS_WIN
                const QString libName = "ngspice.dll";
        #endif
                DebugDialog::debug("Couldn't load ngspice " + m_library.errorString());
                for( const auto& path : libPaths ) {
                        QFileInfo library(QString(path + "/" + libName));
                        DebugDialog::debug("Try path " + library.absoluteFilePath());
                        if(!library.canonicalFilePath().isEmpty()) {
                                m_library.setFileName(library.canonicalFilePath());
                                m_library.load();
                                if( m_library.isLoaded() ) {
                                        break;
                                } else {
                                        DebugDialog::debug("Couldn't load ngspice " + m_library.errorString());
                                        throw std::runtime_error( "Error loading ngspice shared library" );            >
                                }
                        }
                }
        }

        if (!m_library.isLoaded()) {
                DebugDialog::debug("Could not find ngspice.");
                return;
        }
...
```
So Fritzing insists on loading the lib dynamically. Static compilation seems to be totally unreasonable. On the one hand I do understand these additional checks, on the other hand I don't.  
We now have two options:  

- patch the source to allow for static linking or
- go for dynamic linking with the source as is.

I don't really feel like patching the source since this will just end up being more work each time a new version of Fritzing is released. And most distros actually ship a version of `libngspice.so` (Ubuntu 22.04 is at v36). Well, go dynamic.

# Additional Reading:
Things to remember when compiling/linking C/C++ software  
https://gist.github.com/gubatron/32f82053596c24b6bec6?permalink_comment_id=2575013

And Pros and Cons of static and dynamic linking:  
https://stackoverflow.com/questions/26103966/how-can-i-statically-link-standard-library-to-my-c-program


