# PyTorchLocalBuild
Dockerfile to build Pytorch 1.9 with Cuda 11.4.

## Background
I need to use PyTorch 1.9 to study 「BERT による自然言語処理入門」. And I want to run the sample codes in my local machine (RTX3060 12GB). However official pytorch 1.9 (1.9.1+cu111) is build for Cuda 11.1 and I could not find NVidia driver with Cuda11.1 anymore but 11.4. Therefore I have to build Pytorch 1.9 with Cuda 11.4 by myself.

## Prerequisite

* Linux
* Docker
* GPU that supports Cuda 11.4

## Preparation

Install NVidia Driver.

```
sudo apt-get purge '*nvidia*'
sudo apt install nvidia-driver-470
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## How to build docker image

1. Build container
    ```
    cd docker
    CMAKE_VERSION=3.25.1
    docker  build -f torch1.9-build.Dockerfile \
            --build-arg CMAKE_VERSION=${CMAKE_VERSION} \
            --tag=torch-build:0.0.1 .
2. Clone Pytorch source repository
    ```
    cd SOMEWHERE
    git clone https://github.com/pytorch/pytorch.git
    cd pytorch
    git checkout refs/tags/v1.9.1
    git submodule update --init --recursive
    ```
3. Edit source to fix build error.
    ```
    nano caffe2/utils/math_gpu.cu
        # then add "#include <thrust/host_vector.h>" in the beginning of the file
    ```
4. Run the docker container.
    ```
    cd .. # current is SOMEWHERE
    docker run -it --rm --gpus all -v $(pwd):/mnt torch-build:0.0.1
    ```
5. Build source code
    ```
    MAX_JOBS=8 USE_DISTRIBUTED=1 USE_MKLDNN=0 USE_CUDA=1 \
    BUILD_TEST=0 USE_FBGEMM=0 USE_NNPACK=0 USE_QNNPACK=0 \
    USE_XNNPACK=0 BUILD_CUSTOM_PROTOBUF=ON \
    python setup.py bdist_wheel
    ```
    This took about 30 min in my machine. Note that USE_DISTRIBUTED=1 is necessary to avoid "pytorch_lightning 'torch._c' is not a package" error if you use pytorch_lightning.

6. Copy whl to host machine

    In your host machine, copy the whl to your favored location.
    ```
    cp SOMEWHERE/pytorch/dist/torch-1.9.0a0+gitunknown-cp38-cp38-linux_x86_64.whl YOUR_DESTINATION
    ```
