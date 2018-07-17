---
layout: post
title: Building and working with RIDI: Robust IMU Double Integration for Localisation!
---


As a part of my first project at CMU I was asked to verify the perks of using the implementaion of Yang et al. [RIDI: Robust IMU Double Integration](https://arxiv.org/abs/1712.09004) for robust localisation. The method might look mathematically too involved as we look at the paper for the very first time, however, the work is revolutionary in the aspect that it uses IMU data, which in comparison to vision based inputs comsume negligible battery and resources and can be processed faster as well! 

So there we see, a game changing revolutionary idea coming into play using classical machine learning and simplified data.

As I saw there weren't specific blog posts written to work around with this repository I decided to write one, for anyone who is trying to formulate thsi work in future :) 

First of all you need to install the following dependencies:

+ **C++ Dependencies**
  1. Eigen3(version > 3.3)
  2. Glog
  3. Gflags
  4. Ceres-Solver 
  5. OpenCV (version>3)
  6. OpenMesh
+ **Python Dependencies**
  1. numpy (version > 1.13)
  2. numpy-quaternion
  3. scipy
  4. opencv python
  5. plyfile
  
## Step by Step Installation Guide for all the dependencies!

### C++ Dependencies

1. *Installing Glog, Gflags, CMake, BLAS & LAPACK*

Referring to the instructions stated on Ceres-Solver Documentation the following commands need to be executed,: 
```
# CMake
sudo apt-get install cmake
# google-glog + gflags
sudo apt-get install libgoogle-glog-dev
# BLAS & LAPACK
sudo apt-get install libatlas-base-dev
# SuiteSparse and CXSparse (optional)
# - If you want to build Ceres as a *static* library (the default)
#   you can use the SuiteSparse package in the main Ubuntu package
#   repository:
sudo apt-get install libsuitesparse-dev
# - However, if you want to build Ceres as a *shared* library, you must
#   add the following PPA:
sudo add-apt-repository ppa:bzindovic/suitesparse-bugfix-1319687
sudo apt-get update
sudo apt-get install libsuitesparse-dev
```
Please note: I removed the command to download eigen3 as using `sudo apt-get install libeigen3-dev`
as of now installs Eigen 3.2.92 To work along with this repo we specifically need a version greater than or equal to 3.3.0, so we can't use that method to install eigen3. We need to manually install and compile Eigen3!

2. **Installing Eigen3:** You need to check which version of eigen if at all you have one is installed in yoru system. To check the version you can use `pkg-config --modversion eigen3`

If the returned value is more than or equal to 3.3.0 then you are through. Else, you need to find where eigen is located and delete it. This is better as we are trying to avoid version conflict in future by having multiple eigen versions installed in our system. To do so I would suggest the following: 

```
pkg-config --cflags eigen3 #gives you location of eigen in your computer 
sudo rm -r /usr/local/include/eigen3 # is that is the location of eigen3, else type whatever locations you have for eigen.
```

Systems involving to many workspaces and virtaul environments  we might wanna use `sudo apt-get remove libeigen3-dev`

After removing the current versions of eigen in your system, you need to download the latest stable release of Eigen3 from [here](http://eigen.tuxfamily.org/index.php?title=Main_Page) and follow the following commands: 

```
tar xvf eigen-eigen-5a0156e40feb.tar.gz
cd eigen-eigen-5a0156e40feb
mkdir build
cd build 
cmake ..
make all
make install
```
Once through check your eigen version using `pkg-config --modversion eigen3`
Congratulations! You have successfully installed eigen3 in your system!

3. **Installing Ceres-Solver**: Ceres documentation works pretty well. You can have a look at it here. We hve already installed the required dependencies. we finally need to run the following commands to build Ceres-Solver: 

```
git clone https://ceres-solver.googlesource.com/ceres-solver
cd ceres-solver 
mkdir ceres-bin
cd ceres-bin
cmake ..
make all
sudo make install #using sudo as you might need permission to write files!
```
That installs ceres solver in our system! 

4.**Installing OpenCV(> 3)**
Download a stable release of OpenCV from [here](https://opencv.org/releases.html). Once through, extract the files and then:
```
cd opencv-3.3
mkdir build
cd build
cmake ..
make all
make install
```
It's better to use make install, as to remove the current version you could use `make uninstall`

5. **Installing OpenMesh**
Download the latest stable release of OpenMesh from [here](https://www.openmesh.org/download/). After extracting the files use the following commands: 

```
cd OpenMesh-7.1
mkdir build
cd build
cmake ..
make all
make install
```
If you are lucky, OpenMesh will install with the above commands. However, there is a possibility that you might have to use the flag variables that have been described in the [documentation](https://www.openmesh.org/media/Documentations/OpenMesh-Doc-Latest/a03923.html) judiciously to make it work.

One problem that I had faced was it was linking c++98 for some reason so I had to explicitly specify that in the cmake using `cmake .. -DCMAKE_CXX_FLAGS=-std=c++11`

*Please do refer to the flags that can be passed to cmake very judiciously in case you are not able to build OpenMesh!*  




  

