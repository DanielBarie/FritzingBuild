# FritzingBuild
Building Fritzing with Ubuntu 22.04 LTS

Quite an expensive process figuring this out, about €1000 when taking my hourly wage. At least I've dusted off my software build skills. Repeated builds will of course be cheaper...

# What?
Obviously building Fritzing for teaching purposes. The goal is having a release and building a VM for students to explore electrical circuits and their simulation using Fritzing.

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

# To Do
- maybe create a snap or flatpack?

## Get build environment up and running
Caveat: Ubuntu 22.04 is at node.js 12.22.9 so too low for qt to build some components (nodejs > 14 required, doesn't matter for us)
- get Ubuntu 22.04 LTS
- create vm (qt build will take approx 7 minutes)
  - 128 GB disk space (SSD)
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
  - ```apt-get install build-essential git cmake libssl-dev libudev-dev```
- prepare for build, assuming you're happy to build below your user home dir
 - ```apt-get install libglu1-mesa-dev freeglut3-dev mesa-common-dev```
 - ```apt-get install libdrm-dev libgles2-mesa-dev```
 - ```apt-get install pkg-config```
 - do qt build:- 
   - note: Be careful when re-configuring. There may be artifacts from previous builds. See https://stackoverflow.com/questions/6067271/qt-when-building-qt-from-source-how-do-i-clean-old-configure-configurations
   - note: Do also try removing 'CMakeCache.txt' from the build directory
   - note: care about modules?  `/usr/local/Qt-6.6.1/bin/qt-configure-module`
   - `sudo apt-get install libwayland-dev  libwayland-egl1-mesa libwayland-server0 libgles2-mesa-dev libxkbcommon-dev`
   - `sudo apt-get install libfontconfig1-dev libfreetype6-dev libx11-dev libx11-xcb-dev libxext-dev libxfixes-dev libxi-dev libxrender-dev libxcb1-dev libxcb-cursor-dev libxcb-glx0-dev`
   - `sudo apt-get install libxcb-keysyms1-dev libxcb-image0-dev libxcb-shm0-dev libxcb-icccm4-dev libxcb-sync-dev libxcb-xfixes0-dev libxcb-shape0-dev libxcb-randr0-dev libxcb-render-util0-dev libxcb-util-dev libxcb-xinerama0-dev libxcb-xkb-dev libxkbcommon-dev libxkbcommon-x11-dev`
   - `sudo apt-get install libxrender1 libxcb-render-util0 libxcb-shape0 libxcb-randr0 libxcb-xfixes0 libxcb-xkb1 libxcb-sync1 libxcb-shm0  libxcb-icccm4 libxcb-keysyms1-dev libxcb-image0  libxcb-util1 libxcb-cursor0 libxkbcommon-tools libxkbcommon-x11-0 libxcbcommon0 libfontconfig0 libfreetype6 libxext6 libx11-6 libxcb1 libx11-xcb1 libsm6 libice6 libglib2.0-0 libglib2.0-bin`
   - get qt sources from https://download.qt.io/official_releases/qt/6.6/6.6.1/single/: ```https://download.qt.io/official_releases/qt/6.6/6.6.1/single/qt-everywhere-src-6.6.1.tar.xz```
   - untar the sources: `tar xf...` takes a while in silence...
   - go to directory
   - ```./configure```
   - ```cmake --build . --parallel```
   - ```sudo cmake --install .```will install to /usr/local/Qt-6.6.1
 - do libgit build:
   - `wget https://github.com/libgit2/libgit2/archive/refs/tags/v1.7.1.tar.gz`
   - untar
   - change to libgit dir
   - `mkdir build && cd build`
   - `cmake ..`
   - `cmake --build . --parallel`
 - do spiceng build
   - get sources from https://ngspice.sourceforge.io/download.html
   - unzip
   - ```apt-get install libxaw7-dev```
   - change to unzip dir
   - ```./configure```
   - ```make```
   - rename directory to `ngspice-40`
 - get/compile quazip
   - `sudo apt-get install zlib1g-dev libbz2-dev`
   - `wget https://github.com/stachenov/quazip/archive/refs/tags/v1.4.tar.gz`
   - untar
   - rename to expected dir name e.g. `mv quazip-1.4 quazip-6.6.1-1.4`, made of Qt version number and expected version of quazip (1.4)
   - cmake needs to be called with path to qt6 files: `cmake -S . -B ./ -D QUAZIP_QT_MAJOR_VERSION=6 -DCMAKE_PREFIX_PATH="/usr/local/Qt-6.6.1/lib/cmake"`
   - `cmake --build ./ --parallel`
 - do clipper1 lib build
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
 - build svgpp lib v1.3.0 (as of writing, matches version expected by Fritzing)
   - `git clone https://github.com/svgpp/svgpp.git`
   - `sudo apt-get install libxml2-dev`
   - `mv svgpp svgpp-1.3.0`
   - `cd svgpp-1.3.0`
   - `mkdir build-dir`
   - `cd build-dir`
   - `cmake -D BOOST_ROOT=../../boost_1_84_0 ../src` (perfectly happy generating GNU makefile)
   - `make -j` (go fast)
   - 
 - ```apt-get install qtchooser```
 - ```qtchooser -install qt6 /usr/local/Qt-6.6.1/bin/qmake```
 - ```export QT_SELECT=qt6``` (you need to do this after each login or make it permanent in .bashrc)
 - ```wget https://boostorg.jfrog.io/artifactory/main/release/1.84.0/source/boost_1_84_0.tar.gz```
 - ```tar xzvf boost_1_84_0.tar.gz```
 - ```git clone https://github.com/fritzing/fritzing-app.git```
 - ```git clone https://github.com/fritzing/fritzing-parts```
 - change compile script (phoenix.pro):
   - allow for later versions of qt
   - ```nano ./fritzing-app/phoenix.pro```
   - change QT_MOST to 6.6.10 (or whatever is sufficiently high)
 - change boost detect script (will only accept up to 81)
   - ```nano ./fritzing-app/pri/boostdetect.pri```
   - find 81
   - change to 84
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
   - change ```LIBS += -L $$QUAZIP_LIB_PATH -lquazip1-qt$$QT_MAJOR_VERSION``` to be ```LIBS += -L $$QUAZIP_LIB_PATH/quazip -lquazip1-qt$$QT_MAJOR_VERSION```
 - test build
   - `qmake`
   - `make`
   - everything ok? proceed... if not: fix
   - `make clean`
   - `rm Makefile*`

## Build Release
Build releasable compressed file containing all required dependencies.
The release script will clone the parts repo and include it.

- `cd tools/linux_release_script`
- `nano release.sh`
- remove number of parallel jobs limitation (i.e. change `-j16` to `-j`)
- `./release.sh 1.0.2b_develop_private`, we're including the keyword `develop` so we don't have to worry about the repo not being in a clean state. Modify version string to fit your needs.
- `tar -cjf  ./1.0.2b_develop_private.tar.bz2 fritzing-1.0.2b_develop_private.linux.AMD64`: Won't create compressed file because we're not running Travis-CI.

## Deploy Release

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

epxort QT_QPA_PLATFORM=minimal works for building the database...

Build database:  ./Fritzing -db "./fritzing-parts/parts.db" -pp "./fritzing-parts" -f "./"


Don't choose 23.10 because, guess what, it's running ```Using Qt version 6.4.2 in /usr/lib/x86_64-linux-gnu```   
But since we're at it... 
Ubuntu 23.10.01  

- get 23.10.01
- create vm
 - 32 GB disk space
 - 262144 MB RAM (64 GB will be sufficient for building qt, usage rarely tops 32GB)
 - 32 cores, the more, the merrier... but with 32 cores building qt proceeds at an ok speed with 100% load most of the time
 - network
- install Ubuntu
 - standard install
- post-install, basic VM config:
 - be root (or pre-pend sudo to each of the commands below) 
 - ```apt install openssh-server mc```
 - ```apt-get update```
 - ```apt-get upgrade```
 - ```apt-get install qemu-guest-agent```
- prepare for build, assuming you're happy to build below your user home dir
 - get qt sources from https://download.qt.io/official_releases/qt/6.6/6.6.1/single/
 - untar the sources: `tar xf...` takes a while in silence...
 - move to build dir (as described https://doc.qt.io/qt-6/linux-building.html)
 - go to build directory, subdir for qt source
 - ```./configure```
 - ```cmake --build . --parallel```, those cores looking pretty busy...
 - 


   
  - ```apt-get install build-essential git cmake libssl-dev libudev-dev```
  - ```apt-get install qt6-base-dev qtchooser qmake6 qt6-base-dev-tools```
  - ```sudo apt-get install libqt6serialport6-dev libqt6svg6-dev libgit2-dev```
  - fix qt chooser bug: https://askubuntu.com/questions/1460242/ubuntu-22-04-with-qt6-qmake-could-not-find-a-qt-installation-of
   - ```qtchooser -install qt6 $(which qmake6)```
   - ```export QT_SELECT=qt6```

Don't choose 22.04 LTS, because qt6 is 6.2.4. Fritzing 1.0.2 depends on qt 6.5.2

  - ```apt-get install qt6-base-dev qtchooser qt5-qmake qt6-base-dev-tools``` (because there's no qt5-default and qt5 is insufficient for building current versions of fritzing)
  - ```apt-get install libqt6serialport6-dev libqt6svg6-dev```
  - ```apt-get install libgit2-dev```
  - get boost math lib from [boost.org:](https://www.boost.org/users/download/)
    - ```wget https://boostorg.jfrog.io/artifactory/main/release/1.84.0/source/boost_1_84_0.tar.gz```
    - ```tar xzvf boost_1_84_0.tar.gz```
  - set path: ```export PATH=/usr/lib/x86_64-linux-gnu/qt6/bin:$PATH```
  - clone repo: ```git clone https://github.com/fritzing/fritzing-app.git```

 - do clipper2 lib build
   - `git clone https://github.com/AngusJohnson/Clipper2.git`
   - `wget https://github.com/google/googletest/archive/refs/tags/v1.14.0.zip`
   - unzip
   - `cd CPP`
   - `mkdir /Tests/googletest`
   - `cp -R ../../googletest-1.14.0/* ./Tests/googletest/`
   - `mkdir build-dir`
   - `cd build-dir`
   - `cmake ..`
   - `make`
   - `sudo make install` (will go to /usr/local/lib and /usr/local/include)
 
