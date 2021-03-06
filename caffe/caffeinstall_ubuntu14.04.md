---
title: Caffe install file ubuntu14.04
layout: post
---

原文出自[bearpaw的glist](https://gist.github.com/bearpaw/c38ef18ec45ba6548ec0)

# Caffe + Ubuntu 14.04 64bit + CUDA 6.5 配置说明

本步骤能实现用Intel核芯显卡来进行显示， 用NVIDIA GPU进行计算。

## 1. 安装开发所需的依赖包
安装开发所需要的一些基本包

  ```sh
  sudo apt-get install build-essential  # basic requirement
  sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libboost-all-dev libhdf5-serial-dev libgflags-dev libgoogle-glog-dev liblmdb-dev protobuf-compiler #required by caffe
  ```

## 2. 安装CUDA及驱动
### 2.1 准备工作
在关闭桌面管理 lightdm 的情况下安装驱动似乎可以实现Intel 核芯显卡 来显示 + NVIDIA 显卡来计算。具体步骤如下：

1. 首先在BIOS设置里选择用Intel显卡来显示或作为主要显示设备
2. 进入Ubuntu， 按 ctrl+alt+F1 进入tty， 登录tty后输入如下命令
  
   ```sh
   sudo service lightdm stop
   ```
该命令会关闭lightdm。如果你使用 gdm或者其他的desktop manager, 请在安装NVIDIA驱动前关闭他。

### 2.2 下载deb包及安装CUDA
使用deb包安装CUDA及驱动能省去很多麻烦(参见[CUDA Starting Guide](http://developer.download.nvidia.com/compute/cuda/6_5/rel/docs/CUDA_Getting_Started_Linux.pdf))。下载对应于你系统的[CUDA deb包](https://developer.nvidia.com/cuda-downloads), 然后用下列命令添加软件源
```sh
 sudo dpkg -i cuda-repo-<distro>_<version>_<architecture>.deb
 sudo apt-get update
```
然后用下列命令安装CUDA
```sh
 sudo apt-get install cuda
```
安装完成后 reboot.
```sh
sudo reboot
```

### 2.3 安装cuDNN
**(03-25: 今天下最新的caffe回来发现编译不过啊一直CUDNN报错浪费了我几个小时没搞定! 后来才发现caffe15小时前的更新开始使用cudnn v2, 但是官网上并没有明显提示!!! 坑爹啊!)**
cuDNN能加速caffe中conv及pooling的计算。首先下载cuDNN, 然后执行下列命令解压并安装
```sh
tar -zxvf cudnn-6.5-linux-x64-v2.tgz
cd cudnn-6.5-linux-x64-v2
sudo cp lib* /usr/local/cuda/lib64/
sudo cp cudnn.h /usr/local/cuda/include/
```
更新软链接
```sh
cd /usr/local/cuda/lib64/
sudo rm -rf libcudnn.so libcudnn.so.6.5
sudo ln -s libcudnn.so.6.5.48 libcudnn.so.6.5
sudo ln -s libcudnn.so.6.5 libcudnn.so
```

### 2.4 设置环境变量
安装完成后需要在`/etc/profile`中添加环境变量, 在文件最后添加:
```sh
PATH=/usr/local/cuda/bin:$PATH
export PATH
```
保存后, 执行下列命令, 使环境变量立即生效
```
source /etc/profile
```
同时需要添加lib库路径： 在 `/etc/ld.so.conf.d/`加入文件 `cuda.conf`, 内容如下
```sh
/usr/local/cuda/lib64
```
保存后，执行下列命令使之立刻生效
```sh
sudo ldconfig
```
 

## 3. 安装CUDA SAMPLE
进入`/usr/local/cuda/samples`, 执行下列命令来build samples
```sh
sudo make all -j8
```
整个过程大概10分钟左右, 全部编译完成后， 进入 `samples/bin/x86_64/linux/release`, 运行deviceQuery
```sh
./deviceQuery
```
如果出现显卡信息， 则驱动及显卡安装成功：

```
./deviceQuery Starting...

 CUDA Device Query (Runtime API) version (CUDART static linking)

Detected 1 CUDA Capable device(s)

Device 0: "GeForce GTX 670"
  CUDA Driver Version / Runtime Version          6.5 / 6.5
  CUDA Capability Major/Minor version number:    3.0
  Total amount of global memory:                 4095 MBytes (4294246400 bytes)
  ( 7) Multiprocessors, (192) CUDA Cores/MP:     1344 CUDA Cores
  GPU Clock rate:                                1098 MHz (1.10 GHz)
  Memory Clock rate:                             3105 Mhz
  Memory Bus Width:                              256-bit
  L2 Cache Size:                                 524288 bytes
  Maximum Texture Dimension Size (x,y,z)         1D=(65536), 2D=(65536, 65536), 3D=(4096, 4096, 4096)
  Maximum Layered 1D Texture Size, (num) layers  1D=(16384), 2048 layers
  Maximum Layered 2D Texture Size, (num) layers  2D=(16384, 16384), 2048 layers
  Total amount of constant memory:               65536 bytes
  Total amount of shared memory per block:       49152 bytes
  Total number of registers available per block: 65536
  Warp size:                                     32
  Maximum number of threads per multiprocessor:  2048
  Maximum number of threads per block:           1024
  Max dimension size of a thread block (x,y,z): (1024, 1024, 64)
  Max dimension size of a grid size    (x,y,z): (2147483647, 65535, 65535)
  Maximum memory pitch:                          2147483647 bytes
  Texture alignment:                             512 bytes
  Concurrent copy and kernel execution:          Yes with 1 copy engine(s)
  Run time limit on kernels:                     Yes
  Integrated GPU sharing Host Memory:            No
  Support host page-locked memory mapping:       Yes
  Alignment requirement for Surfaces:            Yes
  Device has ECC support:                        Disabled
  Device supports Unified Addressing (UVA):      Yes
  Device PCI Bus ID / PCI location ID:           1 / 0
  Compute Mode:
     < Default (multiple host threads can use ::cudaSetDevice() with device simultaneously) >

deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 6.5, CUDA Runtime Version = 6.5, NumDevs = 1, Device0 = GeForce GTX 670
Result = PASS
```

## 4. 安装Intel MKL 或Atlas
如果没有Intel MKL， 可以用下列命令安装免费的atlas
```sh
sudo apt-get install libatlas-base-dev
```

如果有mkl安装包，首先解压安装包，下面有一个install_GUI.sh文件， 执行该文件，会出现图形安装界面，根据说明一步一步执行即可。

*注意*： 安装完成后需要添加library路径, 创建`/etc/ld.so.conf.d/intel_mkl.conf`文件， 在文件中添加内容
```
/opt/intel/lib
/opt/intel/mkl/lib/intel64
```
注意把路径替换成自己的安装路径。 编辑完后执行
```sh
sudo ldconfig
```

### 5. 安装OpenCV (Optional， 如果运行caffe时opencv报错， 可以重新按照此步骤安装)
虽然我们已经安装了`libopencv-dev `, 但该库似乎会导致[libtiff的相关问题](http://www.cnblogs.com/platero/p/4141063.html), 所以我们需要从源代码build 自己的版本。这个尽量不要手动安装.

#### 安装2.4.10 (推荐)
1. 下载[安装脚本](https://github.com/bearpaw/Install-OpenCV)
2. 进入目录 `Install-OpenCV/Ubuntu/2.4`
3. 执行脚本
   ```sh
   sudo ./opencv2_4_10.sh
   ```

#### 安装2.4.9 (deprecated)
Github上有人已经写好了完整的[安装脚本](https://github.com/jayrambhia/Install-OpenCV), 能自动安装所有dependencies. 下载该脚本，进入`Ubuntu/2.4` 目录, 给所有shell脚本加上可执行权限
```sh
chmod +x *.sh
```
修改脚本`opencv2_4_X.sh`, 在cmake中加入参数
```
-D BUILD_TIFF=ON
```
然后安装（当前为2.4.9）
```sh
sudo ./opencv2_4_9.sh
```
脚本会自动安装依赖项，下载安装包，编译并安装OpenCV。整个过程大概半小时左右。 

注意，安装`2.4.9`时中途可能会报错
```
opencv-2.4.9/modules/gpu/src/nvidia/core/NCVPixelOperations.hpp(51): error: a storage class is not allowed in an explicit specialization
```
解决方法[在此](http://code.opencv.org/issues/3814)  下载 [NCVPixelOperations.hpp](http://code.opencv.org/projects/opencv/repository/revisions/feb74b125d7923c0bc11054b66863e1e9f753141/raw/modules/gpu/src/nvidia/core/NCVPixelOperations.hpp)， 替换掉opencv2.4.9内的文件， *并注释掉`opencv2_4_9.sh`中下载opencv包的代码`, 重新执行`sudo ./opencv2_4_9.sh`.

# 6. 安装Caffe所需要的Python环境
## 6.1 安装anaconda包
[在此](http://continuum.io/downloads)下载最新的安装包, 用默认设置安装在用户目录下。

## 6.2 安装python依赖库
打开新的终端, 用`which python`和`which pip`确定使用的是anaconda提供的python环境，然后进入`caffe_root/python`, 执行下列命令
```
for req in $(cat requirements.txt); do pip install $req; done
```

## 6.3 修正Anaconda存在的bug
加入在编译或者运行caffe时遇到这样的错误
```
/usr/lib/x86_64-linux-gnu/libx264.so.142:undefined reference to ' 
```
那么请删除掉`anaconda/lib`中的`libm.*`. 参考[this issue](https://github.com/BVLC/caffe/issues/985#issuecomment-53533769)

实际编译caffe的时候还碰到anaconda和系统的libreadline冲突的状况，需要conda remove readline （感谢@jastarex ）.

## 6.4 添加Anaconda Library Path
这里需要注意，在运行Caffe时，可能会报一些找不到`libxxx.so`的错误，而用 `locate libxxx.so`命令发现已经安装在anaconda中，这时首先想到的是在`/etc/ld.so.conf.d/` 下面将 `your_anaconda_path/lib` 加入 `LD_LIBRARY_PATH`中。 但是*这样做可能导致登出后无法再进入桌面！！！*原因（猜测）可能是anaconda的lib中有些内容于系统自带的lib产生冲突。

正确的做法是：为了不让系统在启动时就将anaconda/lib加入系统库目录，可以在用户自己的`~/.bashrc` 中添加library path， 比如我就在最后添加了两行
```
# add library path
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:your_anaconda_path/lib"
```
开启另一个终端后即生效，并且重启后能够顺利加载lightdm, 进入桌面环境。

# 7. 安装MATLAB
Caffe提供了MATLAB接口， 有需要用MATLAB的同学可以额外安装MATLAB。 安装教程请自行搜索。 

安装完成后[添加图标]( http://www.linuxidc.com/Linux/2011-01/31632.htm)
```sh
sudo vi /usr/share/applications/Matlab.desktop
```
输入以下内容

```
[Desktop Entry]
Type=Application
Name=Matlab
GenericName=Matlab R2013b
Comment=Matlab:The Language of Technical Computing
Exec=sh /usr/local/MATLAB/R2013b/bin/matlab -desktop
Icon=/usr/local/MATLAB/Matlab.png
Terminal=false
Categories=Development;Matlab;
```
 

(I use the R2013b patched package. First you should uncompress the .iso file. Then use sudo cp to copy the patch file)

# 8. 编译Caffe
## 8.1 编译主程序
终于完成了所有环境的配置，可以愉快的编译Caffe了！ 进入caffe根目录， 首先复制一份`Makefile.config`, 然后修改里面的内容，主要需要修改的参数包括

* CPU_ONLY 是否只使用CPU模式，没有GPU没安装CUDA的同学可以打开这个选项
* BLAS (使用intel mkl还是atlas)
* MATLAB_DIR 如果需要使用MATLAB wrapper的同学需要指定matlab的安装路径, 如我的路径为 `/usr/local/MATLAB/R2013b` (注意该目录下需要包含bin文件夹，bin文件夹里应该包含mex二进制程序)
* DEBUG 是否使用debug模式，打开此选项则可以在eclipse或者NSight中debug程序

完成设置后， 开始编译
```sh
make all -j4
make test
make runtest
```
注意 `-j4` 是指使用几个线程来同时编译， 可以加快速度， j后面的数字可以根据CPU core的个数来决定， 我的CPU使4核， 所以-j4.


## 8.2 编译Matlab wrapper
执行如下命令
```
make matcaffe
```
然后就可以跑官方的matlab demo啦。

## 8.3 编译Python wrapper
```
 make pycaffe 
```
然后基本就全部安装完拉.

接下来大家尽情地跑demo吧～