---
title: Tensorflow setup on Ubuntu 14.04, CUDA 8.0 and cuDNN 5.1
date: 2017-03-05 17:05:00
tags: [Machine Learning, Tools]
---

[TOC]

## Install Nvidia driver

Install required packages.

```bash
$ sudo apt-get update
$ sudo apt-get install build-essential
$ sudo apt-get autoremove
$ sudo apt-get install gfortran
```

Download driver from [Nvidia Unix Driver Archive](http://www.nvidia.com/object/unix.html).  For GTX 1070, I used `Latest Long Lived Branch version: 367.57`. The file name was `NVIDIA-Linux-x86_64-367.57.run`.

Uninstall current Nvidia drivers if exist.

```bash
$ sudo nvidia-uninstall
```

Blacklist `nouveau`.

```bash
$ sudo vim /etc/modprobe.d/blacklist-nouveau.conf
# add following lines
blacklist nouveau
blacklist lbm-nouveau
options nouveau modeset=0
alias nouveau off
alias lbm-nouveau off
```

<!-- more -->

Disable `kernel nouveau`.

```bash
$ echo options nouveau modeset=0 | sudo tee -a /etc/modprobe.d/nouveau-kms.conf
```

Reboot.

> Rebooting through AWS console is recommended if installing on EC-2.

```bash
$ update-initramfs -u
$ sudo apt-get install linux-headers-$(uname -r)
$ reboot
```

Press `Ctrl+Alt+F1` after reboot to login into virtual terminal, and then stop X server to install driver.

> NOTE: The driver installer requires gcc version ≥ 5. So please check with `which gcc` and make sure the soft link point to the correct gcc bin.

```bash
$ sudo service lightdm stop
$ cd <dir containing driver file>
$ sudo sh NVIDIA-Linux-x86_64-367.57.run
$ sudo reboot
```

Nvidia installer automatically installs the driver, and at the end it will ask you whether you want to save your new X configuration. Press Yes. After reboot, save Nvidia configuration in `/etc/X11/xorg.conf`.

```bash
$ sudo nvidia-xconfig
```

Test installation.

> run with `sudo` at the very first time after driver installed.

```bash
$ nvidia-smi
```

## Install CUDA toolkit

Download [CUDA toolkit](https://developer.nvidia.com/cuda-toolkit) *runfile(local)*. Extract the installer into driver and sample parts.

```bash
$ mkdir ./nvidia_intaller
$ sudo ./cuda_8.0.44_linux.run -extract=$(pwd)/nvidia_intaller
```

Blacklist `nouveau` and disable `kernel nouveau` as in the Nvidia driver installation guide, and then login to virtual terminal after reboot.

```bash
$ sudo service lightdm stop
$ cd ./nvidia_intaller
$ sudo ./cuda-linux64-rel-8.0.44.run
$ sudo ./cuda-samples-linux-8.0.44.run
```

Add environment variables.

```bash
$ export CUDA_HOME=/usr/local/cuda-8.0
$ export PATH=/usr/local/cuda-8.0/bin:$PATH
$ export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64
```

Reboot and run test.

```bash
$ cd /usr/local/cuda/samples/1_Utilities/deviceQuery
# make for the first time only
$ make all
$ ./deviceQuery
```

> Currently the `nvcc` depends on `gcc`≤5.2. With incompatible `gcc`, `nvcc` gives error during compiling with CUDA. 

>According to [CUDA incompatible with my gcc version](http://stackoverflow.com/questions/6622454/cuda-incompatible-with-my-gcc-version), it can be solved by adding a soft link. For example `sudo ln -s /usr/bin/gcc-4.8 /usr/local/cuda-8.0/bin/gcc`.

## Install cuDNN

Download cuDNN from [Nvidia cuDNN website](https://developer.nvidia.com/cudnn). Extract to some dir and do copy & replace with CUDA libraries.

```bash
$ tar xvzf cudnn-8.0-linux-x64-v5.1.tgz
$ cd cuda 
$ sudo cp -P include/cudnn.h /usr/local/cuda-8.0/include/ 
$ sudo cp -P lib64/* /usr/local/cuda-8.0/lib64/
```

## Install Tensorflow

### The New Way (with native pip)

```bash
$ sudo apt-get install python-pip python-dev
# CPU support
$ sudo pip install tensorflow
# GPU support
$ sudo pip install tensorflow-gpu
```

> Reference:
- [Installing Tensorflow](https://www.tensorflow.org/install/)



### The Old Way
> Reference: 
> - [Tensorflow Installation](http://www.nvidia.com/object/gpu-accelerated-applications-tensorflow-installation.html).
> - [Tensorflow v0.10 installed from scratch on Ubuntu 16.04, CUDA 8.0RC+Patch, cuDNN v5.1 with a 1080GTX](https://marcnu.github.io/2016-08-17/Tensorflow-v0.10-installed-from-scratch-Ubuntu-16.04-CUDA8.0RC-cuDNN5.1-1080GTX/)

Install Bazel.

```bash
$ sudo apt-get install software-properties-common swig 
$ sudo add-apt-repository ppa:webupd8team/java 
$ sudo apt-get update 
$ sudo apt-get install oracle-java8-installer 
$ echo "deb http://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list 
$ curl https://storage.googleapis.com/bazel-apt/doc/apt-key.pub.gpg | sudo apt-key add - 
$ sudo apt-get update 
$ sudo apt-get install bazel
```

Clone Tensorflow latest github version.

```bash
$ git clone https://github.com/tensorflow/tensorflow
$ cd tensorflow 
```

Run configure script.

> Use the `nvcc` referred `gcc` instead of system default.

```bash
$ ./configure 
Please specify the location of python. [Default is /usr/bin/python]: [enter]
Do you wish to build TensorFlow with Google Cloud Platform support? [y/N] n 
No Google Cloud Platform support will be enabled for TensorFlow 
Do you wish to build TensorFlow with GPU support? [y/N] y 
GPU support will be enabled for TensorFlow 
Please specify which gcc nvcc should use as the host compiler. [Default is /usr/bin/gcc]: /usr/local/cuda/bin/gcc
Please specify the Cuda SDK version you want to use, e.g. 7.0. [Leave empty to use system default]: 8.0 
Please specify the location where CUDA 8.0 toolkit is installed. Refer to README.md for more details. [Default is /usr/local/cuda]: [enter] 
Please specify the Cudnn version you want to use. [Leave empty to use system default]: [enter]
Please specify the location where cuDNN 5 library is installed. Refer to README.md for more details. [Default is /usr/local/cuda]: [enter] 
Please specify a list of comma-separated Cuda compute capabilities you want to build with. 
You can find the compute capability of your device at: https://developer.nvidia.com/cuda-gpus. 
Please note that each additional compute capability significantly increases your build time and binary size. 
[Default is: "3.5,5.2"]: 5.2,6.1 [see https://developer.nvidia.com/cuda-gpus] 
Setting up Cuda include 
Setting up Cuda lib64 
Setting up Cuda bin 
Setting up Cuda nvvm 
Setting up CUPTI include 
Setting up CUPTI lib64 
Configuration finished
```

Build Tensorflow python pip package.

```bash
bazel build -c opt --config=cuda //tensorflow/tools/pip_package:build_pip_package 
bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
```

Pip install.

```bash
$ sudo pip install --upgrade /tmp/tensorflow_pkg/tensorflow-*.whl
```

Test installation in Python.

```python
import tensorflow as tf
hello = tf.constant('Hello, TensorFlow!')
sess = tf.Session()
print(sess.run(hello)) # Hello, TensorFlow! 
a = tf.constant(10) 
b = tf.constant(32) 
print(sess.run(a + b)) # 42
```

## Theano Setup

> Reference: [Easy Installation of an Optimized Theano on Current Ubuntu](http://deeplearning.net/software/theano/install_ubuntu.html#install-ubuntu)

Install Theano.

```bash
$ sudo apt-get install python-numpy python-scipy python-dev python-pip python-nose g++ git libatlas3gf-base libatlas-dev
$ pip install --upgrade --no-deps git+git://github.com/Theano/Theano.git
```

Install blas.

```bash
$ apt-get install libblas-dev
```

Enable GPU.

```bash
$ vim ~/.theanorc
# add the following lines

[global]
floatX=float32
device=gpu
mode=FAST_RUN

[blas]
blas.ldflags=-lblas
```