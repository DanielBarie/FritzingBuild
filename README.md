# FritzingBuild
Building Fritzing with Ubuntu 22.04 LTS

# What?
Obviously building fritzing for teaching purposes. End goal is having a release and building a VM for students to explore electrical circuits using fritzing.

# Why?
There's instructions for building fritzing on their Github: https://github.com/fritzing/fritzing-app/wiki/1.-Building-Fritzing  
These do only refer to building from qt Studio, so you'll have to go through qt studio every time you might want to launch the app. Suboptimal for teaching purposes. Release build instructions (https://github.com/fritzing/fritzing-app/wiki/4.-Publishing-a-Release) are outdated with a link to setting up a build VM leading nowhere. The missing instructions are located at https://github.com/fritzing/fritzing-app/wiki/4.1-Via-Linux-on-a-virtual-box but refer to a very old Ubuntu release as build platform. 

There's another page called linux notes: https://github.com/fritzing/fritzing-app/wiki/1.3-Linux-notes

All that documentation is inconclusive, inconsistent and outdated.

So I decided to try this with a newer release of Ubuntu, here we go.

# How ?
## Get build environment up and running
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
- get Ubuntu 22.04 LTS
- create vm
  - 32 GB disk space
  - 16384 MB RAM
  - 4 cores
  - network
- install Ubuntu
  - minimal install
- post-install:
  - be root (or pre-pend sudo to each of the commands below) 
  - ```apt install openssh-server mc```
  - ```apt-get update```
  - ```apt-get upgrade```
  - ```apt-get install qemu-guest-agent```
  - ```apt-get install build-essential git cmake libssl-dev libudev-dev```
  - ```apt-get install qt6-base-dev qtchooser qt5-qmake qt6-base-dev-tools``` (because there's no qt5-default and qt5 is insufficient for building current versions of fritzing)
  - ```apt-get install libqt6serialport6-dev libqt6svg6-dev```
  - ```apt-get install libgit2-dev```
  - get boost math lib from [boost.org:](https://www.boost.org/users/download/)
    - ```wget https://boostorg.jfrog.io/artifactory/main/release/1.84.0/source/boost_1_84_0.tar.gz```
    - ```tar xzvf boost_1_84_0.tar.gz```
  - set path: ```export PATH=/usr/lib/x86_64-linux-gnu/qt6/bin:$PATH```
  - clone repo: ```git clone https://github.com/fritzing/fritzing-app.git```
  - 
