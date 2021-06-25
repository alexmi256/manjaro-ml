


-D WITH_CUDA=ON \
-D BUILD_opencv_cudacodec=OFF \
-D ENABLE_FAST_MATH=1 \
-D CUDA_FAST_MATH=1 \
-D WITH_CUBLAS=1 \
-D WITH_CUDNN=ON \
-D OPENCV_DNN_CUDA=ON \
-D CUDA_ARCH_BIN=6.1 \


Manjaro for Machine Learning (NVidia)


Install a whole bunch of build related packages
```
pacman -S base-devel cmake
```


Make sure you're using the Nvidia driver instead of nouveau
https://wiki.manjaro.org/index.php/Configure_NVIDIA_(non-free)_settings_and_load_them_on_Startup

Download the CUDA thing
Video_Codec_SDK_11.0.10.zip


```
asp OR asp-git for Manjaro
opencv-cuda
nvidia-sdk
cuda
cuda-tools
jre-openjdk
opencl-nvidia
clang
llvm
cudnn
```


Other tools:
```
docker
docker-compose
nodejs
npm
fio
tmux
java-openjfx
google-chrome
jetbrains-toolbox
vlc
pinta
```



# Install cudNN
https://archlinux.org/packages/community/x86_64/cudnn/
This package was out of date, so we will need to update it's PKGBUILD file to download the latest
This requires [asp](https://archlinux.org/packages/extra/any/asp/), see https://wiki.archlinux.org/title/Arch_Build_System#Usage
However `a`


pipx https://pypi.org/project/pipx/



https://aur.archlinux.org/cgit/aur.git/tree/PKGBUILD?h=opencv-cuda
Has opencv with cuda support

https://github.com/archlinux/svntogit-packages/blob/packages/opencv/trunk/PKGBUILD
Has opencv without cida support

Activate the python virtual environment
Install python dependencies


https://stackoverflow.com/questions/61492452/how-to-check-if-opencv-is-using-gpu-or-not
Check if the virtualenv python works with opencv and cuda:
```
import cv2
print(cv2.cuda.getBuildInformation())
print(cv2.cuda.getCudaEnabledDeviceCount())
```

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
Will try Tensorflow 2.2.3 since that's what was reccomended for the package.
Since I'm using Python 3.9, TF 2.2.3 was never build with that Python version.
Will have to build TF 2.2.3 myself.

https://www.tensorflow.org/install/source


python test.py --dataset celeba-hq --pretrained_model_dir pretrained_models/ --checkpoint_prefix celeba_hq_256x256_random_mask --random_mask 1 --test_dir [Testing Folder Path]


python test.py --dataset celeba-hq --pretrained_model_dir pretrained_models/ --checkpoint_prefix celeba_hq_256x256_random_mask --random_mask 1 --test-file-path test1.png

