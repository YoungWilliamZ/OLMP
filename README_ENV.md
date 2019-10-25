> Notice: 
> 1. This is an instruction for `caffe` of OLMP version base on https://caffe.berkeleyvision.org/installation.html.
> 2. You can compile the `caffe` by yourself in your own machine or directly use the compiled version within docker images. We recommand you to use the docker to run.

# TEST ENVIRONMENT

- OS: Ubuntu 18.04.3 LTS (Bionic Beaver)
- CPU: Intel(R) Xeon(R) CPU E5-2690 v4 @ 2.60GHz
- GPU: TITAN V

# Dependence For GPU user

- [nvidia driver](https://github.com/NVIDIA/nvidia-docker/wiki/Frequently-Asked-Questions#how-do-i-install-the-nvidia-driver)
- [nvidia-docker](https://github.com/NVIDIA/nvidia-docker)

**Node:** You can find the dependencies and install instruction in the above links.

# The image base on

- [nvidia/cuda](https://hub.docker.com/r/nvidia/cuda/)

# How to run

There are two ways provided for you to set up the OLMP environment:

## Run OLMP With `Docker`

### 0. Install Dependencies

#### Install CUDA for Nvidia GPU user

Follow the instruction: https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#ubuntu-installation

Or Goolge a suitable way for your Nvidia-GPU.

#### Install Nvidia-docker for Nvidia GPU user

```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
sudo systemctl restart docker
```

### 1. Get the docker image: 

For GPU user,

```
docker pull youngwilliam/olmp:gpu_python3.6
```

or

```
# In the OLMP source code dir
sudo docker build . -t youngwilliam/olmp:gpu_python3.6
```

For CPU-ONLY user,

```
docker pull youngwilliam/olmp:cpu_python3.6
```

or

```
# Uncomment `# CPU_ONLY := 1` in Makefile.config, ie, change line 8 `# CPU_ONLY := 1` to `CPU_ONLY := 1`
# In the OLMP source code dir
sudo docker build . -t youngwilliam/olmp:cpu_python3.6
```

### 2. Create a new folder for parameter files

```
mkdir -p parameter
cd parameter

# Use vim to create or edit your parameter file, or use the other tools you like.
vim param1.json
```

### 3. Run

#### Run OLMP in the docker directly
##### For GPU user

```
sudo docker run -it -v `pwd`/parameter:/parameter --gpus all youngwilliam/olmp:gpu_python3.6 python3 exp_lenet300100_3.py -d 29 -c /parameter/param1.json
```

##### For CPU user

```
sudo docker run -it -v `pwd`/parameter:/parameter youngwilliam/olmp:cpu_python3.6 python3 exp_lenet300100_3_CPU.py -d 29 -c /parameter/param1.json
```

#### Or you can run a container instead of creating a new contianer every time.

##### For GPU user,

```
sudo docker run -itd -v `pwd`/parameter:/parameter  --gpus all --name OLMP_GPU youngwilliam/olmp:gpu_python3.6 bash
```

Get into the container

```
sudo docker exec -it OLMP_GPU bash
```

Run the OLMP with parameter

```
python3 exp_lenet300100_3.py -d 29 -c /parameter/param1.json
```

##### For CPU-Only user,

```
sudo docker run -itd -v `pwd`/parameter:/parameter --name OLMP_CPU youngwilliam/olmp:cpu_python3.6 bash
```

Get into the container

```
sudo docker exec -it OLMP_CPU bash
```

Run the OLMP with parameter

```
python3 exp_lenet300100_3_CPU.py -d 29 -c /parameter/param1.json
```

**Node:** You can change the parameter files in the `parameter/` dir in you host computer, and the parameter files in docker container will change as well. So you don't need to stop the container.

## Compile the `caffe` of the OLMP version and run OLMP

> We take Ubuntu as example, and you can complie in other distributions of Linux.

### Install Dependencies

#### Install CUDA for Nvidia GPU user

Follow the instruction: https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#ubuntu-installation

Or Goolge a suitable way for your Nvidia-GPU.

#### Install Other dependencies

```
sudo apt-get update
sudo apt-get install -y --no-install-recommends \
    build-essential \
    cmake \
    git \
    wget \
    gcc-5 \
    g++-5 \
    libatlas-base-dev \
    libboost-all-dev \
    libgflags-dev \
    libgoogle-glog-dev \
    libhdf5-serial-dev \
    libleveldb-dev \
    liblmdb-dev \
    libopencv-dev \
    libprotobuf-dev \
    libsnappy-dev \
    protobuf-compiler \
    python3 \
    python3-dev \
    python-numpy \
    python3-pip \
    python3-setuptools \
    python-scipy \
    python3-opencv
```

### Install Python reqiured packages

```
# In the OLMP source code dir
cd python && pip3 install -r requirements.txt && pip3 install easydict
```

### Compile 

> Notice: if you are Not Nvidia GPU user, uncomment `# CPU_ONLY := 1` in `Makefile.config`, ie, change line 8 `# CPU_ONLY := 1` to `CPU_ONLY := 1`

```
# In the OLMP source code dir
make clean && make -j"$(nproc)" all && make -j"$(nproc)" pycaffe
```

### Set the environment variable

> If you restart or reopen a termial, you should do the following again. Or you can set the variable in your shell config file, like `~/.bashrc`

```
# In the OLMP source code dir
export CAFFE_ROOT=`pwd`
export PYCAFFE_ROOT=$CAFFE_ROOT/python
export PYTHONPATH=$PYCAFFE_ROOT:$PYTHONPATH
export PATH=$CAFFE_ROOT/build/tools:$PYCAFFE_ROOT:$PATH
echo "$CAFFE_ROOT/build/lib" >> /etc/ld.so.conf.d/caffe.conf && ldconfig
```

### Run


Run the OLMP with parameter

#### For CPU user

```
python3 exp_lenet300100_3_CPU.py -d 29 -c parameter/param1.json
```

#### For GPU user

```
python3 exp_lenet300100_3.py -d 29 -c parameter/param1.json
```

# Result

After run, you will get result like this:

```
...
I1021 12:22:16.430500     1 solver.cpp:242]     Train net output #0: accuracy = 1
I1021 12:22:16.430508     1 solver.cpp:242]     Train net output #1: loss = 0.0139253 (* 1 = 0.0139253 loss)
I1021 12:22:16.430516     1 solver.cpp:521] Iteration 30000, lr = 0.00353553
Compression:14.568056390361182, Accuracy:1.0
random seed:961449
Time:0.6409
fit:[0.13419976745058326]
0.8658002325494167
```

The `0.8658002325494167` in the last line is the result.