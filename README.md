- [Intro](#intro)
- [Use Nvidia Driver](#use-nvidia-driver)
- [Installation Packages](#installation-packages)
  * [Necessary Packages](#necessary-packages)
    + [Packages Without Extra Work](#packages-without-extra-work)
    + [Packages With Extra Work](#packages-with-extra-work)
    + [Package Install Commands](#package-install-commands)
    + [Notes for Packages With Extra Work](#notes-for-packages-with-extra-work)
      - [pipx](#pipx)
      - [virtualenv](#virtualenv)
      - [asp](#asp)
      - [cuda & cuda-tools & cudnn](#cuda---cuda-tools---cudnn)
- [docker](#docker)
      - [nvidia-sdk](#nvidia-sdk)
      - [nvidia-container-toolkit](#nvidia-container-toolkit)
      - [OpenCV with CUDA, cuDNN](#opencv-with-cuda--cudnn)
  * [Flags that we want for OpenCV](#flags-that-we-want-for-opencv)
  * [Nice to Have](#nice-to-have)
  * [Check Python OpenCV Status](#check-python-opencv-status)
- [Issues ???](#issues----)

# Intro
These are my notes of trying to set up nVidia machine learning (ML) tools such as CUDA, cuDNN, opencv for Manjaro Linux (based on Arch) to run correctly.

# Use Nvidia Driver
Make sure you're using the Nvidia driver instead of `nouveau`
This guide shows how to do that:
- [Configure NVIDIA (non-free) settings and load them on Startup](https://wiki.manjaro.org/index.php/Configure_NVIDIA_(non-free)_settings_and_load_them_on_Startup)

1. Check Display Driver used
```bash
sudo inxi -G
```

2. Install NVIDIA Display Drivers
```bash
sudo mhwd -a pci nonfree 0300
```
3. Reboot the system
4. Check installed drivers
```
    ~/  sudo mhwd -li                                                                                     ✔  22s  
> Installed PCI configs:
--------------------------------------------------------------------------------
                  NAME               VERSION          FREEDRIVER           TYPE
--------------------------------------------------------------------------------
           video-linux            2018.05.04                true            PCI
          video-nvidia            2020.11.30               false            PCI
```
5. Check which driver is used again
```
    ~/  sudo inxi -G                                                                                              ✔ 
Graphics:  Device-1: NVIDIA GP104 [GeForce GTX 1070] driver: nvidia v: 465.31 
           Display: server: X.org 1.20.11 driver: loaded: nvidia resolution: <missing: xdpyinfo> 
           OpenGL: renderer: NVIDIA GeForce GTX 1070/PCIe/SSE2 v: 4.6.0 NVIDIA 465.31 
```

# Installation Packages
These are packages that I've installed in some way or another. 
Don't blindly install all these as that will not work.
Some will need to modifications or are just guidelines of what you're trying to get installed.

You should get familiar with:
- See [Arch Build System > Usage](https://wiki.archlinux.org/title/Arch_Build_System#Usage) for modifying official packages
- See [Arch User Repository > Acquire build files](https://wiki.archlinux.org/title/Arch_User_Repository#Acquire_build_files) for modifying AUR packages

## Necessary Packages
We're going to need most of these:

**Arch**
```
base-devel
clang
cmake
cuda
cuda-tools
cudnn
docker
docker-compose
jre-openjdk
llvm
opencl-nvidia
```

**AUR**
```
asp-git
nvidia-container-toolkit
nvidia-sdk
opencv-cuda
```
**Other**
```
pipx
virtualenv
```

### Packages Without Extra Work
You can safely install these packages without caring about order:
```
base-devel
clang
cmake
cuda
cuda-tools
cudnn
docker
docker-compose
jre-openjdk
llvm
opencl-nvidia
```

### Packages With Extra Work
These packages may have some sort of gotcha or extra thing you need to know about
```
asp
docker
nvidia-container-toolkit
nvidia-sdk
opencv-cuda
pipx
virtualenv
```

### Package Install Commands
See notes below for some of these as they may fail or require other things.
```bash
pacman -S \
    base-devel \
    clang \
    cmake \
    cuda \
    cuda-tools \
    cudnn \
    docker \
    docker-compose \
    jre-openjdk \
    llvm \
    opencl-nvidia
pamac install \
    asp-git \
    nvidia-container-toolkit \
    nvidia-sdk
```

### Notes for Packages With Extra Work
#### pipx
[pipx](https://pypa.github.io/pipx/installation/) is recommended for installing `virtualenv`.
No idea why, but what's the harm?
```bash
pip install pipx
```

#### virtualenv
[virtualenv](https://virtualenv.pypa.io/en/latest/installation.html) is good for using project level dependencies so we don't use global ones all the time which would result in a mental breakdown after a while. 
```bash
pipx install virtualenv
```

#### asp
[asp](https://archlinux.org/packages/extra/any/asp/) is used for downloading package instructions so that we can modify and build them locally.
You will need to become a bit familiar with https://wiki.archlinux.org/title/Arch_Build_System#Usage
Unfortunately, Manjaro does not have this in the `extra` repo, so we need to install [asp-git](https://aur.archlinux.org/packages/asp-git/)
```bash
pamac install asp-git
```

#### cuda & cuda-tools & cudnn
You can install these without issue, just make sure you do it early on
- https://archlinux.org/packages/community/x86_64/cuda/
- https://archlinux.org/packages/community/x86_64/cuda-tools/
- https://archlinux.org/packages/community/x86_64/cudnn/
```bash
pacman -S cuda cuda-tools cudnn
```

**NOTE:** At time of writing cudnn was out of date, the steps below show what I had to do to install latest package.
The package maintainer was super quick to update this once reported.
Thus, when I downloaded the package via `asp` I didn't need to make any changes myself.
If package is outdated:

1. Download config locally

This is where `asp` comes in.
```bash
asp export cudnn
```
2. Modify config
3. Make and install package
```
   ~/workspace/cudnn  makepkg                                          ✔ 
==> Making package: cudnn 8.2.1.32-1 (Fri 25 Jun 2021 12:57:24 PM EDT)
==> Checking runtime dependencies...
==> Checking buildtime dependencies...
==> Retrieving sources...
  -> Downloading cudnn-11.3-linux-x64-v8.2.1.32.tgz...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 1792M  100 1792M    0     0  30.8M      0  0:00:58  0:00:58 --:--:-- 33.2M
  -> Found NVIDIA_SLA+cuDNN_Supp_Feb2017_release.pdf
==> Validating source files with sha512sums...
    cudnn-11.3-linux-x64-v8.2.1.32.tgz ... Passed
    NVIDIA_SLA+cuDNN_Supp_Feb2017_release.pdf ... Passed
==> Extracting sources...
==> Entering fakeroot environment...
==> Starting package()...
==> Tidying install...
  -> Removing libtool files...
  -> Purging unwanted files...
  -> Compressing man and info pages...
==> Checking for packaging issues...
==> Creating package "cudnn"...
  -> Generating .PKGINFO file...
  -> Generating .BUILDINFO file...
  -> Generating .MTREE file...
  -> Compressing package...
==> Leaving fakeroot environment.
==> Finished making: cudnn 8.2.1.32-1 (Fri 25 Jun 2021 12:59:11 PM EDT)
    ~/workspace/cudnn  makepkg --install                                                                              ✔  1m 47s  
==> WARNING: A package has already been built, installing existing package...
==> Installing package cudnn with pacman -U...
[sudo] password for alex: 
loading packages...
resolving dependencies...
looking for conflicting packages...

Packages (1) cudnn-8.2.1.32-1

Total Installed Size:  3826.94 MiB

:: Proceed with installation? [Y/n] y
(1/1) checking keys in keyring                                                     [###############################################] 100%
(1/1) checking package integrity                                                   [###############################################] 100%
(1/1) loading package files                                                        [###############################################] 100%
(1/1) checking for file conflicts                                                  [###############################################] 100%
(1/1) checking available disk space                                                [###############################################] 100%
:: Processing package changes...
(1/1) installing cudnn                                                             [###############################################] 100%
:: Running post-transaction hooks...
(1/1) Arming ConditionNeedsUpdate...
    ~/workspace/cudnn                                                                                                    ✔  22s  
```

# docker
```bash
sudo su
pacman -S docker
systemctl start docker.service
systemctl enable docker.service
groupadd docker
usermod -aG docker $USER
Ctrl+D
docker run hello-world
```

#### nvidia-sdk
- https://aur.archlinux.org/packages/nvidia-sdk/

This requires that you download `Video_Codec_SDK_11.1.5.zip` or similar from https://developer.nvidia.com/nvidia-video-codec-sdk/download and place it in the necessary location.

1. Try to install the package so it creates necessary folders, this is expected to fail.
```bash
pamac install nvidia-sdk 
```
2. Copy files
```bash
cp ~/Downloads/Video_Codec_SDK_11.0.10.zip /var/tmp/pamac-build-$USER/nvidia-sdk/
```
3. Run install again, it should now succeed
```bash
pamac install nvidia-sdk 
```

#### nvidia-container-toolkit
```bash
pamac install
```
```
Warning about nvidia containers!

Systemd v247.2-2 introduced a unified cgroup change which has somewhat
broken nvidia-container's access to the handles in
/sys/fs/cgroup/devices.

If you are using Docker you will then need to explicitly allow access
to the nvidia devices like:

docker run ... --gpus all --device /dev/nvidia0 --device \
    /dev/nvidia-uvm --device /dev/nvidia-uvm-tools --device \
    /dev/nvidiactl ...

or by using a docker-compose which esposes the devices with:

devices:
  - /dev/nvidia0:/dev/nvidia0
  - /dev/nvidiactl:/dev/nvidiactl
  - /dev/nvidia-modeset:/dev/nvidia-modeset
  - /dev/nvidia-uvm:/dev/nvidia-uvm
  - /dev/nvidia-uvm-tools:/dev/nvidia-uvm-tools

See the link for more details:
https://github.com/NVIDIA/nvidia-docker/issues/1447#issuecomment-757034464
```

#### OpenCV with CUDA, cuDNN
This is by far the biggest PITA.

The [AUR opencv-cuda](https://aur.archlinux.org/packages/opencv-cuda/) (v4.5.X):
- no longer exists
- had CUDA support
- no cuDNN support
It also won't help too much since we want to install this as a python package in our virtualenv.
See [PKGBUILD](https://aur.archlinux.org/cgit/aur.git/tree/PKGBUILD?h=opencv-cuda)

The [Official opencv-cuda](https://archlinux.org/packages/extra/x86_64/opencv/):
- no CUDA support 
- no cuDNN support
See [PKGBUILD](https://github.com/archlinux/svntogit-packages/blob/packages/opencv/trunk/PKGBUILD)

Nevertheless, these packages are a good intro to what dependencies, tools, flags, and patches we will need to build opencv. 

Other useful resources were:
- [Figure out what ninja is](https://dmerej.info/blog/post/chuck-norris-part-1-cmake-ninja/)
- https://github.com/opencv/opencv-python
- https://gist.github.com/raulqf/f42c718a658cddc16f9df07ecc627be7
- https://answers.opencv.org/question/222056/fast-math-flags-enable_fast_math-and-cuda_fast_math/
- https://github.com/NeerajGulia/python-opencv-cuda


##### Arch Build Package
```bash
git clone https://aur.archlinux.org/opencv-cuda.git
asp export opencv
cp opencv/*.patch opencv-cuda
```
I [sorted](https://sortmylist.com/) and [compared](https://www.diffchecker.com/diff) the flags for `opencv` and `opencv-cuda`.
Then added the following to the modified `opencv-cuda` PKGBUILD.
```
OPENCV_ENABLE_NONFREE=ON
WITH_VULKAN=ON 
BUILD_opencv_python2=OFF
BUILD_opencv_python3=ON
WITH_CUDNN=ON
OPENCV_DNN_CUDA=ON
CUDA_ARCH_BIN=8.6
ENABLE_FAST_MATH=ON
```
Note that the`CUDA_ARCH_BIN` version of is based on my GPU, see the full compatibility list in https://developer.nvidia.com/cuda-gpus

| GPU                 | Compute Capability |
|---------------------|--------------------|
| 30XX                | 8.6                |
| 20XX/1650/Titan RTX | 7.5                |
| TITAN V             | 7.0                |
| 10XX/Titan V/Xp/X   | 6.1                |


##### BASH

**OUTDATED:**
The info below is outdated since the AUR package was removed. These commands were relevant for [v4.5.2](https://aur.archlinux.org/cgit/aur.git/tree/PKGBUILD?h=opencv-cuda&id=0e5af8ca4a708994fee7a050d4f34d54505eb719)

There were the shell commands I ran to test the process
```bash
pkgver=4.5.2
srcdir="$PWD"
wget https://github.com/opencv/opencv/archive/$pkgver/opencv-$pkgver.tar.gz
wget https://github.com/opencv/opencv_contrib/archive/$pkgver/opencv_contrib-$pkgver.tar.gz
cp ../opencv/opencv-openexr3.patch .
tar zxvf opencv-$pkgver.tar.gz
tar zxvf opencv_contrib-$pkgver.tar.gz
patch -d "opencv-$pkgver" -p1 < opencv-openexr3.patch
sed -i 's|nvcuvid.h|nvidia-sdk/nvcuvid.h|' opencv_contrib-$pkgver/modules/cud*/src/*.hpp
find "opencv-$pkgver" -name '*.cpp' -exec sed -i s/dgels_/LAPACK_dgels/g {} \;


mkdir -p build
cd build
# Make sure you activate your virtualenv and install correct version requirements such as numpy==1.19.5
_pythonpath=`python -c "from sysconfig import get_path; print(get_path('platlib'))"`
echo $_pythonpath

cmake ../opencv-$pkgver \
-GNinja \
-DWITH_OPENCL=ON \
-DWITH_OPENGL=ON \
-DWITH_TBB=ON \
-DOpenGL_GL_PREFERENCE=GLVND \
-DBUILD_WITH_DEBUG_INFO=OFF \
-DBUILD_TESTS=OFF \
-DBUILD_PERF_TESTS=OFF \
-DBUILD_EXAMPLES=ON \
-DINSTALL_C_EXAMPLES=ON \
-DINSTALL_PYTHON_EXAMPLES=ON \
-DCMAKE_INSTALL_PREFIX=/usr \
-DCMAKE_INSTALL_LIBDIR=lib \
-DCPU_BASELINE_DISABLE=SSE3 \
-DCPU_BASELINE_REQUIRE=SSE2 \
-DWITH_NVCUVID=ON \
-DWITH_CUDA=ON \
-DCUDA_FAST_MATH=ON \
-DWITH_CUBLAS=ON \
-DOPENCV_ENABLE_NONFREE=ON \
-DWITH_VULKAN=ON \
-DBUILD_opencv_python2=OFF \
-DBUILD_opencv_python3=ON \
-DWITH_CUDNN=ON \
-DOPENCV_DNN_CUDA=ON \
-DCUDA_ARCH_BIN=6.1 \
-DENABLE_FAST_MATH=ON \
-DCUDA_HOST_COMPILER=/opt/cuda/bin/gcc \
-DOPENCV_EXTRA_MODULES_PATH="$srcdir/opencv_contrib-$pkgver/modules" \
-DOPENCV_SKIP_PYTHON_LOADER=ON \
-DEIGEN_INCLUDE_PATH=/usr/include/eigen3 \
-DOPENCV_PYTHON3_INSTALL_PATH=$_pythonpath \
-DLAPACK_LIBRARIES="/usr/lib/liblapack.so;/usr/lib/libblas.so;/usr/lib/libcblas.so" \
-DLAPACK_CBLAS_H="/usr/include/cblas.h" \
-DLAPACK_LAPACKE_H="/usr/include/lapacke.h" \
-DOPENCV_GENERATE_PKGCONFIG=ON
ninja
sudo ? ninja install
```

**Current Steps:**
???

##### opencv-python
This will very likely be easier, but I never got around to it. 
The idea is you clone the [opencv-python](https://github.com/opencv/opencv-python#manual-builds), export flags as shell variables, and then build the package.

## Nice to Have Packages
These are tools I personally use. You can ignore these if you want/don't use them.
```
fio
google-chrome
java-openjfx
jetbrains-toolbox
nodejs
npm
pinta
tmux
vlc
discord
```

## Check Python OpenCV Status
- https://stackoverflow.com/questions/61492452/how-to-check-if-opencv-is-using-gpu-or-not

Check if the virtualenv python works with opencv and cuda:
```
import cv2
print(cv2.getBuildInformation())
print(cv2.cuda.getCudaEnabledDeviceCount())
```

# Issues ???
With Tensorflow (TF) 2.5.0 and Numpy 1.21.X. 
Getting an error because of what appears to be numpy version.
```
2021-06-25 10:45:30.730037: I tensorflow/stream_executor/platform/default/dso_loader.cc:53] Successfully opened dynamic library libcudart.so.11.0
RuntimeError: module compiled against API version 0xe but this version of numpy is 0xd
Traceback (most recent call last):
  File "/home/alex/workspace/Hypergraphs-Image-Inpainting/test.py", line 6, in <module>
    import cv2
ImportError: numpy.core.multiarray failed to import
```
Similar to https://github.com/freqtrade/freqtrade/issues/4281

It seems TF is only compatible with 1.19.5
https://github.com/tensorflow/tensorflow/issues/50204

Going to try different Tensorflow version. 
Will try Tensorflow 2.2.3 since that's what was recommended for the package.
Since I'm using Python 3.9, TF 2.2.3 was never build with that Python version.
Will have to build TF 2.2.3 myself.

https://www.tensorflow.org/install/source

## Testing with [Hypergraphs-Image-Inpainting](https://github.com/GouravWadhwa/Hypergraphs-Image-Inpainting)
```bash
python test.py --dataset celeba-hq --pretrained_model_dir pretrained_models/ --checkpoint_prefix celeba_hq_256x256_random_mask --random_mask 1 --test_dir [Testing Folder Path]
python test.py --dataset celeba-hq --pretrained_model_dir pretrained_models/ --checkpoint_prefix celeba_hq_256x256_random_mask --random_mask 1 --test_file_path test1.png
```

