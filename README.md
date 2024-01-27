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
  - ```apt-get install build-essential git cmake libssl-dev libudev-dev```
- prepare for build, assuming you're happy to build below your user home dir
	- ```apt-get install libglu1-mesa-dev freeglut3-dev mesa-common-dev libdrm-dev libgles2-mesa-dev pkg-config```
- get and untar Boost:
	- ```wget https://boostorg.jfrog.io/artifactory/main/release/1.84.0/source/boost_1_84_0.tar.gz```
 	- ```tar xzvf boost_1_84_0.tar.gz``` 	
- do libgit build or install `sudo apt-get install libgit2-dev` and skip steps below (but remember to install the package when deploying)
   - `wget https://github.com/libgit2/libgit2/archive/refs/tags/v1.7.1.tar.gz`
   - untar
   - change to libgit dir
   - `mkdir build && cd build`
   - `cmake ..`
   - `cmake --build . --parallel`
 - do spiceng build or install shared lib devel package (https://packages.ubuntu.com/search?keywords=ngspice ngspice-dev) and skip steps below
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
- prepare Qt build
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
  
# Dynamically linked, Qt-zlib
- Do Qt build:
	- Go to directory: `cd qt-everywhere-src-6.6.1`  
  - ```./configure -qt-zlib```
  - ```cmake --build . --parallel```
  - ```sudo cmake --install .``` will install to /usr/local/Qt-6.6.1 (need this for qmake) but could also set path to point into the build directory...
  - ```apt-get install qtchooser```
  - ```qtchooser -install qt6 /usr/local/Qt-6.6.1/bin/qmake```
  - ```export QT_SELECT=qt6``` (you need to do this after each login or make it permanent in .bashrc)
- Prepare Fritzing build
  - ```git clone https://github.com/fritzing/fritzing-app.git```
	- optional: ```git clone https://github.com/fritzing/fritzing-parts```
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
- Test build
  - `qmake`
  - `make`
- Everything ok? proceed... if not: fix
  - `make clean`
  - `rm Makefile*`

# Statically linked, Qt-zlib
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

## Do libgit2 build (static)
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

## Do spiceng build (static)
- `sudo apt-get install libxaw7-dev`
- no: `./configure -enable-static`
- edit `configure.ac`
	- line # 481: replace `AC_SUBST([STATIC], [-shared])` with `AC_SUBST([STATIC], [-static])` as per https://sourceforge.net/p/ngspice/discussion/127605/thread/2216ffc5/
 - line 634: change `` to static
 - line 636 change to static and include lib ver
- `make -j`

# Build Release
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

## The heck. 
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

# Additional Reading:
Things to remember when compiling/linking C/C++ software  
https://gist.github.com/gubatron/32f82053596c24b6bec6?permalink_comment_id=2575013

And Pros and Cons of static and dynamic linking:  
https://stackoverflow.com/questions/26103966/how-can-i-statically-link-standard-library-to-my-c-program


