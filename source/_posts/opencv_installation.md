---
title: COMPILE and INSTALL Opencv-python on Centos6.5
date: 2016-06-23 14:27:19
tags:
    - opencv
    - anaconda
---
<Excerpt in index | 首页摘要>
Compile and Install Opencv-python, with **Extra** modules.
<!-- more -->

For Ubuntu: Click [Here](http://docs.opencv.org/master/d7/d9f/tutorial_linux_install.html)

######  User: root
## System Preparation
  - Install Dependencies
  ```
  yum install gcc cmake git gtk2-devel libdc1394-devel \
  libv41-devel pkgconfig ffmpeg-devel gstreamer-plugins-base-devel \
  libpng-devel libjpeg-turbo-devel jasper-devel openexr-devel \
  libtiff-devel libwebp-devel tbb-devel eigen3-devel -y
```

## Install Anaconda
  1. Download `Anaconda.<version>.sh` from [ANACONDA Offical Site](https://www.continuum.io/downloads)
  2. Install Anaconda
    - `bash Anaconda.<version>.sh`
      - ** PREFIX=/usr/local/anaconda **
  3. Set PATH: `export PATH=/usr/local/anaconda/bin:$PATH`

## COMPILE and INSTALL opencv-python
[Opencv Offical Tutorial](http://docs.opencv.org/3.1.0/dd/dd5/tutorial_py_setup_in_fedora.html)
  1. Download opencv src to `/usr/local/src/opencv`
    - `cd /usr/local/src`
    - `git clone https://github.com/Itseez/opencv.git`
  2. Download opencv_contrib to `/usr/local/src/opencv_contrib`
    - `cd /usr/local/src`
    - `git clone https://github.com/Itseez/opencv_contrib`
  3. make directory for compile
    - `mkdir build`
    - `cd build`
  4. Compile opencv
    - Configuring
    ```
      cmake -D CMAKE_BUILD_TYPE=RELEASE \
      -D OPENCV_EXTRA_MODULES_PATH=/usr/local/src/opencv_contrib/modules \
      -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D PYTHON_INCLUDE_DIR=/usr/local/anaconda/include/python2.7/ \
      -D PYTHON_LIBRARY=/usr/local/anaconda/lib/libpython2.7.so \
      -DPYTHON2_PACKAGES_PATH=/usr/local/anaconda/lib/python2.7/site-packages \
      -DBUILD_SHARED_LIBS=OFF \
      -D BUILD_DOCS=ON \
      -D BUILD_TESTS=OFF \
      -D BUILD_PERF_TESTS=OFF \
      -D BUILD_EXAMPLES=OFF ..
      ```
    - Optional Flags
      - `-D BUILD_opencv_python3=OFF`

  5. Installing
    - `make && make install`
  6. Check if `cv2.so` is in Python packages directory
    - `ls /usr/local/anaconda/lib/python2.7/site-packages/cv2.so`
    - If no `cv2.so` there, copy from PYTHON_PACKAGES_PATH:
      - `cp /usr/local/lib/python2.7/site-packages/cv2.so /usr/local/anaconda/lib/python2.7/site-packages`
  7. Validate opencv-python
    - `python -c "import cv2; print cv2.__version__"`
