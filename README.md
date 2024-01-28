# FritzingBuild
Building Fritzing with Ubuntu 22.04 LTS

Please don't ask for binaries.

Quite an expensive process figuring this out, about €1000 when taking my hourly wage. At least I've dusted off my software build skills. Repeated builds will of course be cheaper...
![grafik](https://github.com/DanielBarie/FritzingBuild/assets/73287620/e97830fd-37e5-4151-91bf-8deac4182c02)
![grafik](https://github.com/DanielBarie/FritzingBuild/assets/73287620/6673ce88-e0bf-4592-9234-d20469ed429b)


# What?
Obviously building Fritzing for teaching purposes. The goal is having a release and building a VM for students to explore electrical circuits and their simulation using Fritzing.

We'll build with the latest (as of writing) version of Qt which is 6.6.1. For the brave: Download any version greater than 6.5.2 (minimum requirement for Fritzing) at https://download.qt.io/archive/qt/ Please be advised to change paths accordingly.

Compilation will give some deprecation warnings regarding Qt functions. Will work anyway, we'll maybe have to re-visit this if Qt remove/change these functions.

# Why?
- There's instructions for building fritzing on their Github: https://github.com/fritzing/fritzing-app/wiki/1.-Building-Fritzing
- These do only refer to building from Qt Studio, so you'll have to go through Qt studio every time you might want to launch the app. Suboptimal for teaching purposes.
- Release build instructions (https://github.com/fritzing/fritzing-app/wiki/4.-Publishing-a-Release) are outdated with a link to setting up a build VM leading nowhere. The missing instructions are located at https://github.com/fritzing/fritzing-app/wiki/4.1-Via-Linux-on-a-virtual-box but refer to a very old Ubuntu release as build platform.
- There's another page called linux notes: https://github.com/fritzing/fritzing-app/wiki/1.3-Linux-notes

All that documentation is inconclusive, inconsistent and outdated. This just is a huge pain.

So I decided to try this with a newer release of Ubuntu, here we go.

# Alternatives?
I went for Ubuntu because of the LTS release and because I wrongly assumed having an easier build process. Boy, was I mistaken. Don't choose 22.04 LTS, because Qt is 6.2.4. Fritzing 1.0.2 depends on Qt > 6.5.2. Don't choose Ubuntu 23.10 because, guess what, it's running ```Using Qt version 6.4.2 in /usr/lib/x86_64-linux-gnu```  
When focusing on LTS, we might as well take Debian Bookworm (five years from mid 2023 on). But this also uses Qt 6.4.2. So having the correct version of Qt is the main issue...

# How ?
We'll set up a sufficiently beefy VM and get going in there. No containerized building.

# To Do
- re-structure (steps in correct ordering, first, we need to build qt, then quazip)
- maybe create a snap or flatpack?
- fix simulator (include lib):
  ```
  Running command(remcirc):
  qt.core.library: "ngspice" cannot load: Cannot load library ngspice: (/lib/x86_64-linux-gnu/ngspice: Kann die Datei-Daten nicht lesen: Ist ein Verzeichnis)
  "Couldn't load ngspice Cannot load library ngspice: (/lib/x86_64-linux-gnu/ngspice: Kann die Datei-Daten nicht lesen: Ist ein Verzeichnis)"
  "Try path /home/daniel/fritzing-app/tools/linux_release_script/fritzing-1.0.2b_develop_private.linux.AMD64/lib/libngspice.so"
  "Try path /home/daniel/.local/share/Fritzing/Fritzing/libngspice.so"
  "Try path /usr/share/gnome/Fritzing/Fritzing/libngspice.so"
  "Try path /usr/local/share/Fritzing/Fritzing/libngspice.so"
  "Try path /usr/share/Fritzing/Fritzing/libngspice.so"
  "Try path /var/lib/snapd/desktop/Fritzing/Fritzing/libngspice.so"
  "Could not find ngspice."
  Running m_simulator->command('reset'):
  ```

# Common Steps for Ubuntu 22.04 Build Environment
Set up the basic build environment (VM, dependencies) for building Fritzing.  
Having completed that, you may choose to build Fritzing linked either dynamically or statically. The main difference is the Qt build.  

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
  - ```apt-get install ```
- prepare for build, assuming you're happy to build below your user home dir
	- ```sudo apt-get install build-essential git cmake libssl-dev libudev-dev libglu1-mesa-dev freeglut3-dev mesa-common-dev libdrm-dev libgles2-mesa-dev pkg-config```
- get and untar Boost:
	- ```wget https://boostorg.jfrog.io/artifactory/main/release/1.84.0/source/boost_1_84_0.tar.gz```
 	- ```tar xzvf boost_1_84_0.tar.gz```


# This is where the common part stops
Below steps are different for shared libs / dynamic linking and static linkin.

  
# Dynamically linked, Qt-zlib
## Prepare and Do Qt build
  - note: Be careful when re-configuring. There may be artifacts from previous builds. See https://stackoverflow.com/questions/6067271/qt-when-building-qt-from-source-how-do-i-clean-old-configure-configurations
  - note: Documentation: https://doc.qt.io/qt-6/configure-options.html section "Reconfiguring Existing Builds"
  - note: Do also try removing 'CMakeCache.txt' from the build directory
  - note: Care about modules?  `/usr/local/Qt-6.6.1/bin/qt-configure-module`
  - note: Go static (see below)? 
  - `sudo apt-get install libwayland-dev  libwayland-egl1-mesa libwayland-server0 libgles2-mesa-dev libxkbcommon-dev`
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
 
## do libgit build 
or install `sudo apt-get install libgit2-dev` and skip steps below (but remember to change lib paths for Fritzing build and install the package when deploying)
   - `wget https://github.com/libgit2/libgit2/archive/refs/tags/v1.7.1.tar.gz`
   - untar
   - change to libgit dir
   - `mkdir build && cd build`
   - `cmake ..`
   - `cmake --build . --parallel`
 ## do spiceng build 
or install shared lib devel package (https://packages.ubuntu.com/search?keywords=ngspice ngspice-dev) and skip steps below (but remember to change lib paths for Fritzing build and install the package when deploying)
   - get sources from https://ngspice.sourceforge.io/download.html
   - unzip
   - ```apt-get install libxaw7-dev```
   - change to unzip dir
   - ```./configure```
   - ```make```
   - rename directory to `ngspice-40`
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
   	- change ```LIBS += -L $$QUAZIP_LIB_PATH -lquazip1-qt$$QT_MAJOR_VERSION``` to be ```LIBS += -L $$QUAZIP_LIB_PATH/quazip -lquazip1-qt$$QT_MAJOR_VERSION```
- Test build
  - `qmake`
  - `make`
- Everything ok? proceed... if not: fix
  - `make clean`
  - `rm Makefile*`

# Statically linked, Qt-zlib
Well, not entirely static build. Qt, Clipping and SpiceNG will be linked statically, Quazip, svgpp and Git2 will be linked dynamically. Main reasons are: The required version of Qt is still well ahead of most distributions, there are no binaries for the Clipping library. As for SpiceNG: there are binary builds in most distributions but with lower version numbers (Ubuntu 22.04: v36). Quazip must match our version of Qt, so no luck here, either. We can't rely on binaries for most distributions. The main reason for dynamically linking this is linking issues. Same for Git2. But the Git2 library is available in most distirbutions. Same for svgpp-dev.

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
- get source, unzip (version 42)
- `sudo apt-get install libxaw7-dev`
- no: `./configure -enable-static`
- edit `configure.ac`
	- line # 481: replace `AC_SUBST([STATIC], [-shared])` with `AC_SUBST([STATIC], [-static])` as per https://sourceforge.net/p/ngspice/discussion/127605/thread/2216ffc5/
 - edit `~ngspice/src/makefile.am`
 	- line 634: change `` to static
 	- line 636 change to static and include lib ver
 - `./compile_linux_shared.sh 64`
 - result will be found in `~/ngspice-42/releasesh/src/.libs` (`libngspice.a`), approx. 18MB, don't worry about failed install (can't write to system dirs without sufficient privileges, obviously...)

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
for dynamic linking, see other instructions

## Install svgpp
`sudo apt-get install dvgpp-dev`

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

## Platform Plugin Errors when building Fritzing Release
The release script will build the parts database. For this, Fritzing will be launched with `-db` parameter. Since your (at least mine) development machine probably doesn't have a graphical user interface installed (or available when you're building over ssh / remote shell), the platform plugin is set to be `minimal` or `offscreen` (in our version of the release script). This of course needs to be linked into the binary (when doing a static build) or available in the `platforms` directory (dynamic linking or static when having built Qt with default parameters, which will be pulled automatically from your build machine config).
```
qt.qpa.plugin: Could not find the Qt platform plugin "minimal" in ""
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.

Available platform plugins are: xcb.

Aborted (core dumped)
```  

Solutions are to  

 - either do a Qt build with platform parameters specified explicitly (https://forum.qt.io/topic/105059/static-qt-build-with-multiple-qpa-plugins/3) 
 - or provide the necessary plugins in the `platforms` directory of the release

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
	- find (in the else case, not EMSCRIPTEN) `option(BUILD_SHARED_LIBS "" OFF)`
 	- change to `option(BUILD_SHARED_LIBS "" ON)`
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

# Additional Reading:
Things to remember when compiling/linking C/C++ software  
https://gist.github.com/gubatron/32f82053596c24b6bec6?permalink_comment_id=2575013

And Pros and Cons of static and dynamic linking:  
https://stackoverflow.com/questions/26103966/how-can-i-statically-link-standard-library-to-my-c-program


