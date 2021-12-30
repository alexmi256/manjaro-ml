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
        * [Arch Build Package](#arch-build-package)
        * [opencv-python](#opencv-python)
  * [Nice to Have Packages](#nice-to-have-packages)
  * [Check Python OpenCV Status](#check-python-opencv-status)
    + [extra/python-opencv & extra/opencv-cuda](#extra-python-opencv---extra-opencv-cuda)
  * [GitHub opencv/opencv-python](#github-opencv-opencv-python)
- [Issues](#issues)
  * [Building GitHub Python Wheel](#building-github-python-wheel)
    + [LAPACK](#lapack)
    + [NVCUVID](#nvcuvid)
    + [VTK](#vtk)
  * [Numpy API version (RuntimeError: module compiled against API version 0xe but this version of numpy is 0xd)](#numpy-api-version--runtimeerror--module-compiled-against-api-version-0xe-but-this-version-of-numpy-is-0xd-)
  * [Testing with [Hypergraphs-Image-Inpainting](https://github.com/GouravWadhwa/Hypergraphs-Image-Inpainting)](#testing-with--hypergraphs-image-inpainting--https---githubcom-gouravwadhwa-hypergraphs-image-inpainting-)

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
    fmt \
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
cp ~/Downloads/Video_Codec_SDK_11.1.5.zip /var/tmp/pamac-build-$USER/nvidia-sdk/
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

The [extras/opencv](https://archlinux.org/packages/extra/x86_64/opencv/):
- no CUDA support 
- no cuDNN support
See [PKGBUILD](https://github.com/archlinux/svntogit-packages/blob/packages/opencv/trunk/PKGBUILD)

The new package to use seems to be [extras/opencv-cuda](https://archlinux.org/packages/extra/x86_64/opencv-cuda/):
- has CUDA
- has CUDNN

Nevertheless, these packages are a good intro to what dependencies, tools, flags, and patches we will need to build opencv. 

Other useful resources were:
- [Figure out what ninja is](https://dmerej.info/blog/post/chuck-norris-part-1-cmake-ninja/)
- https://github.com/opencv/opencv-python
- https://gist.github.com/raulqf/f42c718a658cddc16f9df07ecc627be7
- https://answers.opencv.org/question/222056/fast-math-flags-enable_fast_math-and-cuda_fast_math/
- https://github.com/NeerajGulia/python-opencv-cuda


##### Arch Build Package
See commit changes for what I vaguely did previously, but it seems like now you can just install `opencv-cuda` and `python-opencv` via `pacman`.

When I did this manually in the past, I [sorted](https://sortmylist.com/) and [compared](https://www.diffchecker.com/diff) the flags for `opencv` and `opencv-cuda`.
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


##### opencv-python
This is more convenient when dealing with virtual environments
The idea is you clone the [opencv-python](https://github.com/opencv/opencv-python#manual-builds), export flags as shell variables, and then build the package.
```bash
virtualenv venv
source venv/bin/activate
pip install mpi4py
git clone --recursive https://github.com/opencv/opencv-python.git
export ENABLE_CONTRIB=1
export ENABLE_HEADLESS=0
export CMAKE_ARGS="-DCUDA_ARCH_BIN=8.6 -DCUDA_HOST_COMPILER=/opt/cuda/bin/gcc -DCUDA_nvcuvid_LIBRARY=/usr/lib/libnvcuvid.so -DCUDA_FAST_MATH=ON -DEIGEN_INCLUDE_PATH=/usr/include/eigen3 -DENABLE_FAST_MATH=ON -DLAPACK_CBLAS_H=/usr/include/cblas.h -DLAPACK_LAPACKE_H=/usr/include/lapacke.h -DLAPACK_LIBRARIES=/usr/lib/liblapack.so;/usr/lib/libblas.so;/usr/lib/libcblas.so -DOPENCV_DNN_CUDA=ON -DOPENCV_ENABLE_NONFREE=ON -DWITH_CUBLAS=ON -DWITH_CUDA=ON -DWITH_CUDNN=ON -DWITH_CUFFT=ON -DWITH_LAPACK=ON -DWITH_NVCUVID=ON -DWITH_OPENCL=ON -DWITH_OPENGL=ON -DWITH_QT=ON -DWITH_TBB=ON -DWITH_VULKAN=ON"
wget https://raw.githubusercontent.com/archlinux/svntogit-packages/packages/opencv/trunk/vtk9.patch
patch -d opencv-python/opencv -p1 < vtk9.patch
pip wheel . --verbose
```

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

### extra/python-opencv & extra/opencv-cuda
**NOTE:** You may need to update Numpy to the latest version in order to run this in case of a `RuntimeError: module compiled against API version 0xe but this version of numpy is 0xd` error.
This is unfortunate because the TensorFlow v2.5.X is not compatible with latest numpy version v1.21.5 

```
General configuration for OpenCV 4.5.4 =====================================
  Version control:               unknown

  Extra modules:
    Location (extra):            /build/opencv/src/opencv_contrib-4.5.4/modules
    Version control (extra):     unknown

  Platform:
    Timestamp:                   2021-11-25T17:50:32Z
    Host:                        Linux 5.15.3-arch1-1 x86_64
    CMake:                       3.22.0
    CMake generator:             Unix Makefiles
    CMake build tool:            /usr/bin/make
    Configuration:               Release

  CPU/HW features:
    Baseline:                    SSE SSE2
      requested:                 SSE3
      required:                  SSE2
      disabled:                  SSE3
    Dispatched code generation:  SSE4_1 SSE4_2 FP16 AVX AVX2 AVX512_SKX
      requested:                 SSE4_1 SSE4_2 AVX FP16 AVX2 AVX512_SKX
      SSE4_1 (15 files):         + SSE3 SSSE3 SSE4_1
      SSE4_2 (1 files):          + SSE3 SSSE3 SSE4_1 POPCNT SSE4_2
      FP16 (0 files):            + SSE3 SSSE3 SSE4_1 POPCNT SSE4_2 FP16 AVX
      AVX (4 files):             + SSE3 SSSE3 SSE4_1 POPCNT SSE4_2 AVX
      AVX2 (30 files):           + SSE3 SSSE3 SSE4_1 POPCNT SSE4_2 FP16 FMA3 AVX AVX2
      AVX512_SKX (5 files):      + SSE3 SSSE3 SSE4_1 POPCNT SSE4_2 FP16 FMA3 AVX AVX2 AVX_512F AVX512_COMMON AVX512_SKX

  C/C++:
    Built as dynamic libs?:      YES
    C++ standard:                11
    C++ Compiler:                /usr/bin/c++  (ver 11.1.0)
    C++ flags (Release):         -D_FORTIFY_SOURCE=2 -march=x86-64 -mtune=generic -O2 -pipe -fno-plt   -fsigned-char -W -Wall -Werror=return-type -Werror=non-virtual-dtor -Werror=address -Werror=sequence-point -Wformat -Werror=format-security -Wmissing-declarations -Wundef -Winit-self -Wpointer-arith -Wshadow -Wsign-promo -Wuninitialized -Wsuggest-override -Wno-delete-non-virtual-dtor -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -msse -msse2 -fvisibility=hidden -fvisibility-inlines-hidden -O3 -DNDEBUG  -DNDEBUG
    C++ flags (Debug):           -D_FORTIFY_SOURCE=2 -march=x86-64 -mtune=generic -O2 -pipe -fno-plt   -fsigned-char -W -Wall -Werror=return-type -Werror=non-virtual-dtor -Werror=address -Werror=sequence-point -Wformat -Werror=format-security -Wmissing-declarations -Wundef -Winit-self -Wpointer-arith -Wshadow -Wsign-promo -Wuninitialized -Wsuggest-override -Wno-delete-non-virtual-dtor -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -msse -msse2 -fvisibility=hidden -fvisibility-inlines-hidden -g  -DDEBUG -D_DEBUG
    C Compiler:                  /usr/bin/cc
    C flags (Release):           -D_FORTIFY_SOURCE=2 -march=x86-64 -mtune=generic -O2 -pipe -fno-plt   -fsigned-char -W -Wall -Werror=return-type -Werror=address -Werror=sequence-point -Wformat -Werror=format-security -Wmissing-declarations -Wmissing-prototypes -Wstrict-prototypes -Wundef -Winit-self -Wpointer-arith -Wshadow -Wuninitialized -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -msse -msse2 -fvisibility=hidden -O3 -DNDEBUG  -DNDEBUG
    C flags (Debug):             -D_FORTIFY_SOURCE=2 -march=x86-64 -mtune=generic -O2 -pipe -fno-plt   -fsigned-char -W -Wall -Werror=return-type -Werror=address -Werror=sequence-point -Wformat -Werror=format-security -Wmissing-declarations -Wmissing-prototypes -Wstrict-prototypes -Wundef -Winit-self -Wpointer-arith -Wshadow -Wuninitialized -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -msse -msse2 -fvisibility=hidden -g  -DDEBUG -D_DEBUG
    Linker flags (Release):      -Wl,--exclude-libs,libippicv.a -Wl,--exclude-libs,libippiw.a -Wl,-O1,--sort-common,--as-needed,-z,relro,-z,now  -Wl,--gc-sections -Wl,--as-needed  
    Linker flags (Debug):        -Wl,--exclude-libs,libippicv.a -Wl,--exclude-libs,libippiw.a -Wl,-O1,--sort-common,--as-needed,-z,relro,-z,now  -Wl,--gc-sections -Wl,--as-needed  
    ccache:                      NO
    Precompiled headers:         NO
    Extra dependencies:          m pthread /lib64/libGL.so /lib64/libGLU.so cudart_static dl rt nppc nppial nppicc nppidei nppif nppig nppim nppist nppisu nppitc npps cublas cudnn cufft -L/opt/cuda/lib64 -L/lib64
    3rdparty dependencies:

  OpenCV modules:
    To be built:                 alphamat aruco barcode bgsegm bioinspired calib3d ccalib core cudaarithm cudabgsegm cudacodec cudafeatures2d cudafilters cudaimgproc cudalegacy cudaobjdetect cudaoptflow cudastereo cudawarping cudev cvv datasets dnn dnn_objdetect dnn_superres dpm face features2d flann freetype fuzzy gapi hdf hfs highgui img_hash imgcodecs imgproc intensity_transform java line_descriptor mcc ml objdetect optflow phase_unwrapping photo plot python3 quality rapid reg rgbd saliency shape stereo stitching structured_light superres surface_matching text tracking video videoio videostab viz wechat_qrcode xfeatures2d ximgproc xobjdetect xphoto
    Disabled:                    world
    Disabled by dependency:      -
    Unavailable:                 julia matlab ovis python2 sfm ts
    Applications:                examples apps
    Documentation:               NO
    Non-free algorithms:         YES

  GUI:                           QT5
    QT:                          YES (ver 5.15.2 )
      QT OpenGL support:         YES (Qt5::OpenGL 5.15.2)
    GTK+:                        NO
    OpenGL support:              YES (/lib64/libGL.so /lib64/libGLU.so)
    VTK support:                 YES (ver 9.1.0)

  Media I/O: 
    ZLib:                        /lib64/libz.so (ver 1.2.11)
    JPEG:                        /lib64/libjpeg.so (ver 80)
    WEBP:                        /lib64/libwebp.so (ver encoder: 0x020f)
    PNG:                         /lib64/libpng.so (ver 1.6.37)
    TIFF:                        /lib64/libtiff.so (ver 42 / 4.3.0)
    JPEG 2000:                   OpenJPEG (ver 2.4.0)
    OpenEXR:                     OpenEXR::OpenEXR (ver 3.1.3)
    HDR:                         YES
    SUNRASTER:                   YES
    PXM:                         YES
    PFM:                         YES

  Video I/O:
    DC1394:                      YES (2.2.6)
    FFMPEG:                      YES
      avcodec:                   YES (58.134.100)
      avformat:                  YES (58.76.100)
      avutil:                    YES (56.70.100)
      swscale:                   YES (5.9.100)
      avresample:                NO
    GStreamer:                   YES (1.18.5)
    v4l/v4l2:                    YES (linux/videodev2.h)

  Parallel framework:            TBB (ver 2021.4 interface 12040)

  Trace:                         YES (with Intel ITT)

  Other third-party libraries:
    Intel IPP:                   2020.0.0 Gold [2020.0.0]
           at:                   /build/opencv/src/build-cuda/3rdparty/ippicv/ippicv_lnx/icv
    Intel IPP IW:                sources (2020.0.0)
              at:                /build/opencv/src/build-cuda/3rdparty/ippicv/ippicv_lnx/iw
    VA:                          YES
    Lapack:                      YES (/usr/lib/liblapack.so /usr/lib/libblas.so /usr/lib/libcblas.so)
    Eigen:                       YES (ver 3.4.0)
    Custom HAL:                  NO
    Protobuf:                    /lib64/libprotobuf.so (3.17.3)

  NVIDIA CUDA:                   YES (ver 11.5, CUFFT CUBLAS)
    NVIDIA GPU arch:             35 37 50 52 60 61 70 75 80 86
    NVIDIA PTX archs:

  cuDNN:                         YES (ver 8.3.0)

  Vulkan:                        YES
    Include path:                /build/opencv/src/opencv-4.5.4/3rdparty/include
    Link libraries:              Dynamic load

  OpenCL:                        YES (INTELVA)
    Include path:                /build/opencv/src/opencv-4.5.4/3rdparty/include/opencl/1.2
    Link libraries:              Dynamic load

  Python 3:
    Interpreter:                 /usr/bin/python3 (ver 3.9.9)
    Libraries:                   /lib64/libpython3.9.so (ver 3.9.9)
    numpy:                       /usr/lib/python3.9/site-packages/numpy/core/include (ver 1.21.3)
    install path:                lib/python3.9/site-packages

  Python (for build):            /usr/bin/python3

  Java:                          
    ant:                         /bin/ant (ver 1.10.11)
    JNI:                         /usr/lib/jvm/default/include /usr/lib/jvm/default/include/linux /usr/lib/jvm/default/include
    Java wrappers:               YES
    Java tests:                  NO

  Install to:                    /usr
-----------------------------------------------------------------
```

## GitHub opencv/opencv-python
```
General configuration for OpenCV 4.5.5 =====================================
  Version control:               4.5.5-dirty

  Extra modules:
    Location (extra):            /home/alex/workspace/build-github-opencv-python/opencv-python/opencv_contrib/modules
    Version control (extra):     4.5.5

  Platform:
    Timestamp:                   2021-12-30T22:03:35Z
    Host:                        Linux 5.10.84-1-MANJARO x86_64
    CMake:                       3.22.1
    CMake generator:             Ninja
    CMake build tool:            /usr/bin/ninja
    Configuration:               Release

  CPU/HW features:
    Baseline:                    SSE SSE2 SSE3
      requested:                 SSE3
    Dispatched code generation:  SSE4_1 SSE4_2 FP16 AVX AVX2 AVX512_SKX
      requested:                 SSE4_1 SSE4_2 AVX FP16 AVX2 AVX512_SKX
      SSE4_1 (16 files):         + SSSE3 SSE4_1
      SSE4_2 (1 files):          + SSSE3 SSE4_1 POPCNT SSE4_2
      FP16 (0 files):            + SSSE3 SSE4_1 POPCNT SSE4_2 FP16 AVX
      AVX (4 files):             + SSSE3 SSE4_1 POPCNT SSE4_2 AVX
      AVX2 (31 files):           + SSSE3 SSE4_1 POPCNT SSE4_2 FP16 FMA3 AVX AVX2
      AVX512_SKX (5 files):      + SSSE3 SSE4_1 POPCNT SSE4_2 FP16 FMA3 AVX AVX2 AVX_512F AVX512_COMMON AVX512_SKX

  C/C++:
    Built as dynamic libs?:      NO
    C++ standard:                11
    C++ Compiler:                /usr/bin/c++  (ver 11.1.0)
    C++ flags (Release):         -fsigned-char -ffast-math -W -Wall -Werror=return-type -Werror=non-virtual-dtor -Werror=address -Werror=sequence-point -Wformat -Werror=format-security -Wmissing-declarations -Wundef -Winit-self -Wpointer-arith -Wshadow -Wsign-promo -Wuninitialized -Wsuggest-override -Wno-delete-non-virtual-dtor -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -msse -msse2 -msse3 -fvisibility=hidden -fvisibility-inlines-hidden -O3 -DNDEBUG  -DNDEBUG
    C++ flags (Debug):           -fsigned-char -ffast-math -W -Wall -Werror=return-type -Werror=non-virtual-dtor -Werror=address -Werror=sequence-point -Wformat -Werror=format-security -Wmissing-declarations -Wundef -Winit-self -Wpointer-arith -Wshadow -Wsign-promo -Wuninitialized -Wsuggest-override -Wno-delete-non-virtual-dtor -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -msse -msse2 -msse3 -fvisibility=hidden -fvisibility-inlines-hidden -g  -O0 -DDEBUG -D_DEBUG
    C Compiler:                  /usr/bin/cc
    C flags (Release):           -fsigned-char -ffast-math -W -Wall -Werror=return-type -Werror=address -Werror=sequence-point -Wformat -Werror=format-security -Wmissing-declarations -Wmissing-prototypes -Wstrict-prototypes -Wundef -Winit-self -Wpointer-arith -Wshadow -Wuninitialized -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -msse -msse2 -msse3 -fvisibility=hidden -O3 -DNDEBUG  -DNDEBUG
    C flags (Debug):             -fsigned-char -ffast-math -W -Wall -Werror=return-type -Werror=address -Werror=sequence-point -Wformat -Werror=format-security -Wmissing-declarations -Wmissing-prototypes -Wstrict-prototypes -Wundef -Winit-self -Wpointer-arith -Wshadow -Wuninitialized -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -msse -msse2 -msse3 -fvisibility=hidden -g  -O0 -DDEBUG -D_DEBUG
    Linker flags (Release):      -Wl,--exclude-libs,libippicv.a -Wl,--exclude-libs,libippiw.a   -Wl,--gc-sections -Wl,--as-needed  
    Linker flags (Debug):        -Wl,--exclude-libs,libippicv.a -Wl,--exclude-libs,libippiw.a   -Wl,--gc-sections -Wl,--as-needed  
    ccache:                      NO
    Precompiled headers:         NO
    Extra dependencies:          va va-drm /usr/lib/liblapack.so /usr/lib/libblas.so /usr/lib/libcblas.so Qt5::Test Qt5::Concurrent Qt5::OpenGL /lib64/libjpeg.so /lib64/libwebp.so /lib64/libpng.so /lib64/libtiff.so openjp2 Qt5::Core Qt5::Gui Qt5::Widgets /lib64/libhdf5.so /lib64/libsz.so /lib64/libz.so /lib64/libdl.so /lib64/libm.so /lib64/libGL.so /lib64/libGLU.so VTK::FiltersExtraction VTK::FiltersSources VTK::FiltersTexture VTK::IOExport VTK::IOGeometry VTK::IOPLY VTK::InteractionStyle VTK::RenderingCore VTK::RenderingLOD VTK::RenderingOpenGL2 Iconv::Iconv m pthread cudart_static dl rt nppc nppial nppicc nppidei nppif nppig nppim nppist nppisu nppitc npps cublas cudnn cufft -L/opt/cuda/lib64 -L/lib64
    3rdparty dependencies:       libprotobuf ade ittnotify IlmImf quirc ippiw ippicv

  OpenCV modules:
    To be built:                 alphamat aruco barcode bgsegm bioinspired calib3d ccalib core cudaarithm cudabgsegm cudacodec cudafeatures2d cudafilters cudaimgproc cudalegacy cudaobjdetect cudaoptflow cudastereo cudawarping cudev cvv datasets dnn dnn_objdetect dnn_superres dpm face features2d flann fuzzy gapi hdf hfs highgui img_hash imgcodecs imgproc intensity_transform line_descriptor mcc ml objdetect optflow phase_unwrapping photo plot python3 quality rapid reg rgbd saliency shape stereo stitching structured_light superres surface_matching text tracking video videoio videostab viz wechat_qrcode xfeatures2d ximgproc xobjdetect xphoto
    Disabled:                    freetype world
    Disabled by dependency:      -
    Unavailable:                 java julia matlab ovis python2 sfm ts
    Applications:                -
    Documentation:               NO
    Non-free algorithms:         YES

  GUI:                           QT5
    QT:                          YES (ver 5.15.2 )
      QT OpenGL support:         YES (Qt5::OpenGL 5.15.2)
    GTK+:                        YES (ver 3.24.30)
      GThread :                  YES (ver 2.70.1)
      GtkGlExt:                  NO
    OpenGL support:              YES (/lib64/libGL.so /lib64/libGLU.so)
    VTK support:                 YES (ver 9.1.0)

  Media I/O: 
    ZLib:                        /lib64/libz.so (ver 1.2.11)
    JPEG:                        /lib64/libjpeg.so (ver 80)
    WEBP:                        /lib64/libwebp.so (ver encoder: 0x020f)
    PNG:                         /lib64/libpng.so (ver 1.6.37)
    TIFF:                        /lib64/libtiff.so (ver 42 / 4.3.0)
    JPEG 2000:                   OpenJPEG (ver 2.4.0)
    OpenEXR:                     build (ver 2.3.0)
    HDR:                         YES
    SUNRASTER:                   YES
    PXM:                         YES
    PFM:                         YES

  Video I/O:
    DC1394:                      YES (2.2.6)
    FFMPEG:                      YES
      avcodec:                   YES (58.134.100)
      avformat:                  YES (58.76.100)
      avutil:                    YES (56.70.100)
      swscale:                   YES (5.9.100)
      avresample:                NO
    GStreamer:                   YES (1.18.5)
    v4l/v4l2:                    YES (linux/videodev2.h)

  Parallel framework:            TBB (ver 2021.4 interface 12040)

  Trace:                         YES (with Intel ITT)

  Other third-party libraries:
    Intel IPP:                   2020.0.0 Gold [2020.0.0]
           at:                   /home/alex/workspace/build-github-opencv-python/opencv-python/_skbuild/linux-x86_64-3.9/cmake-build/3rdparty/ippicv/ippicv_lnx/icv
    Intel IPP IW:                sources (2020.0.0)
              at:                /home/alex/workspace/build-github-opencv-python/opencv-python/_skbuild/linux-x86_64-3.9/cmake-build/3rdparty/ippicv/ippicv_lnx/iw
    VA:                          YES
    Lapack:                      YES (/usr/lib/liblapack.so /usr/lib/libblas.so /usr/lib/libcblas.so)
    Eigen:                       YES (ver 3.4.0)
    Custom HAL:                  NO
    Protobuf:                    build (3.19.1)

  NVIDIA CUDA:                   YES (ver 11.5, CUFFT CUBLAS FAST_MATH)
    NVIDIA GPU arch:             86
    NVIDIA PTX archs:

  cuDNN:                         YES (ver 8.3.0)

  Vulkan:                        YES
    Include path:                /home/alex/workspace/build-github-opencv-python/opencv-python/opencv/3rdparty/include
    Link libraries:              Dynamic load

  OpenCL:                        YES (INTELVA)
    Include path:                /home/alex/workspace/build-github-opencv-python/opencv-python/opencv/3rdparty/include/opencl/1.2
    Link libraries:              Dynamic load

  Python 3:
    Interpreter:                 /home/alex/workspace/build-github-opencv-python/venv/bin/python (ver 3.9.9)
    Libraries:                   /usr/lib/libpython3.9.so (ver 3.9.9)
    numpy:                       /tmp/pip-build-env-0ceolcnq/overlay/lib/python3.9/site-packages/numpy/core/include (ver 1.19.3)
    install path:                python/cv2/python-3

  Python (for build):            /home/alex/workspace/build-github-opencv-python/venv/bin/python

  Java:                          
    ant:                         NO
    JNI:                         /usr/lib/jvm/default/include /usr/lib/jvm/default/include/linux /usr/lib/jvm/default/include
    Java wrappers:               NO
    Java tests:                  NO

  Install to:                    /home/alex/workspace/build-github-opencv-python/opencv-python/_skbuild/linux-x86_64-3.9/cmake-install
-----------------------------------------------------------------
```

# Issues
## Building GitHub Python Wheel
### LAPACK
The AUR opencv package used to require a patch for LAPACK
https://aur.archlinux.org/cgit/aur.git/tree/opencv-lapack-3.10.patch?h=opencv-git
I didn't bother with this now as the extras PKGBUILD does not do this

### NVCUVID
This seems like something old thing that no one cares about anymore
- https://github.com/opencv/opencv/issues/11220
- https://github.com/opencv/opencv/issues/10201

However it seems we might be able to enable it with `CUDA_nvcuvid_LIBRARY` and `WITH_NVCUVID=ON`
Not knowing which lib to point it to, we can try and find a couple...
```bash
sudo find /usr/ -name "*nvcuvid*"
/usr/lib/libnvcuvid.so
/usr/lib/libnvcuvid.so.495.44
/usr/lib/libnvcuvid.so.1
/usr/include/nvidia-sdk/nvcuvid.h
/usr/include/ffnvcodec/dynlink_nvcuvid.h
/usr/lib32/libnvcuvid.so
/usr/lib32/libnvcuvid.so.495.44
/usr/lib32/libnvcuvid.so.1
ls -la /usr/lib/libnvcuvid*
lrwxrwxrwx 1 root root      15 Nov 21 13:08 /usr/lib/libnvcuvid.so -> libnvcuvid.so.1
lrwxrwxrwx 1 root root      20 Nov 21 13:08 /usr/lib/libnvcuvid.so.1 -> libnvcuvid.so.495.44
-rwxr-xr-x 1 root root 5274408 Nov 21 13:08 /usr/lib/libnvcuvid.so.495.44
```

### VTK
The build step complained about VTK
```
Traceback (most recent call last):
    File "<string>", line 1, in <module>
  ModuleNotFoundError: No module named 'mpi4py'
  -- Found PkgConfig: /usr/bin/pkg-config (found version "1.8.0")
  -- VTK is not found. Please set -DVTK_DIR in CMake to VTK build directory, or to VTK install subdirectory with VTKConfig.cmake file
```
No idea why this would be, vtk seems to be installed, but official repo does have a patch for it.
I tried applying it and rebuilding:
```bash
wget https://raw.githubusercontent.com/archlinux/svntogit-packages/packages/opencv/trunk/vtk9.patch
patch -d opencv-python/opencv -p1 < vtk9.patch
```
This seemed to fix things
## Numpy API version (RuntimeError: module compiled against API version 0xe but this version of numpy is 0xd)
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

