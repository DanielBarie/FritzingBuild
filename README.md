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
  - ```sudo apt-get install libqt6serialport6-dev libqt6svg6-dev```
  - ```apt-get install libgit2-dev```
  - get boost math lib from [boost.org:](https://www.boost.org/users/download/)
    - ```wget https://boostorg.jfrog.io/artifactory/main/release/1.84.0/source/boost_1_84_0.tar.gz```
    - ```tar xzvf boost_1_84_0.tar.gz```
  - set path: ```export PATH=/usr/lib/x86_64-linux-gnu/qt5/bin:$PATH```
  - clone repo: ```git clone https://github.com/fritzing/fritzing-app.git```
  - 
