---
layout: post
title: 'Docker Study-gcc'
date: 2019-07-18
author: eavan
color: rgb(255,210,32)
cover: ''
tags: docker
---

# Docker Study

## Docker里的c/c++编译

&emsp; 这里根据项目需要在docker内进行c++文件的编译，因此需要先行配置环境。

&emsp; 由于ubuntu的docker大约200MB，根据网络资料，可采取更为精简化、轻量级的debian版本和Alpine版本。Alpine操作系统是一个面向安全的轻型 Linux 发行版。，不同于通常 Linux 发行版，Alpine 采用了 musl libc 和 busybox 以减小系统的体积和运行时资源消耗，但功能上比busybox又完善的多，因此得到开源社区越来越多的青睐。在保持瘦身的同时，Alpine 还提供了自己的包管理工具 apk，可以通过[https://pkgs.alpinelinux.org/packages](https://pkgs.alpinelinux.org/packages) 网站上查询包信息，也可以直接通过 apk 命令直接查询和安装各种软件，且大小仅有5M。

&emsp; 目前 Docker 官方已开始推荐使用 Alpine 替代之前的 Ubuntu 做为基础镜像环境。这样会带来多个好处。包括镜像下载速度加快，镜像安全性提高，主机之间的切换更方便，占用更少磁盘空间等。
```
REPOSITORY          TAG           IMAGE ID          VIRTUAL SIZE
alpine              latest        4e38e38c8ce0      4.799 MB
debian              latest        4d6ce913b130      84.98 MB
ubuntu              latest        b39b81afc8ca      188.3 MB
centos              latest        8efe422e6104      210 MB
```
&emsp; 用户可以直接使用docker run alpine 命令下载。
&emsp; 检验是否可用：
`sudo docker run alpine echo '123'` 
&emsp; 进入容器
`sudo docker run -it alpine /bin/sh`
&emsp; 安装gcc:`apk add gcc`
&emsp; 安装g++:`apk add g++`,使用`g++ -v`检测是否安装成功。

## docker里的opencv编译问题

### Alpine

&emsp; 这里先使用了其他开发者开发的docker容器，为基于alpine的python opencv的容器，大约800MB，对于我们所用的来说，多了一些python的部件，但可以使用，存在本地image里，名字为test。  
&emsp; dockerfile网址：[https://gitlab.com/ucair/alpine-opencv/blob/master/Dockerfile](https://gitlab.com/ucair/alpine-opencv/blob/master/Dockerfile)  
&emsp; 进入docker容器后（sudo docker run --name test -it -v /home/psy/mydocker/mygcc/other:/mygcc 85aa44e19d49 /bin/sh），检测是否可用选择[https://www.cnblogs.com/xiangfeidemengzhu/p/7657887.html](https://www.cnblogs.com/xiangfeidemengzhu/p/7657887.html) 进行检测。这种运行方法为cmake方法，其中测试图片放在当前文件夹内，如果正常，则显示如图：
![1]({{ "/assets/docker/opencv结果.PNG"|absolute_url}})
&emsp; 其他方法可见：[https://blog.csdn.net/m0_37357063/article/details/84191669](https://blog.csdn.net/m0_37357063/article/details/84191669) 
 
Dockerfile test版如下：（目前只能通过cmake编译，牵扯到libgtk2.0-dev/pkg-config等的图像显示还有问题）

```
FROM eavan/mygcc:base

ENV OPENCV_VERSION 4.1.0

RUN echo "@testing http://dl-cdn.alpinelinux.org/alpine/edge/testing" >> \
    /etc/apk/repositories;

WORKDIR /mycpp/
VOLUME ["/mycpp"]
RUN apk add --update --no-cache \
    cmake build-base linux-headers \
    # image formats
    libjpeg-turbo-dev libpng-dev tiff-dev \
    #if need pkgconfig
    #pkgconfig (gtk+2.0)

RUN wget https://github.com/opencv/opencv/archive/${OPENCV_VERSION}.zip && \
    unzip ${OPENCV_VERSION}.zip && \
    rm -rf ${OPENCV_VERSION}.zip && \
    mkdir -p opencv-${OPENCV_VERSION}/build && \
    cd opencv-${OPENCV_VERSION}/build && \
    cmake \
      -D CMAKE_BUILD_TYPE=RELEASE \
      -D WITH_TBB=ON \
      -D WITH_IPP=OFF \
      -D BUILD_TESTS=OFF \
      -D BUILD_DOCS=OFF \
      -D BUILD_PERF_TESTS=OFF \
      -D BUILD_SHARED_LIBS=OFF \
      -D BUILD_opencv_apps=OFF \
      -D INSTALL_C_EXAMPLES=OFF \
      -D INSTALL_PYTHON_EXAMPLES=OFF \
#     -D OPENCV_GENERATE_PKGCONFIG=ON \ if need pkg-config 
      .. && \
    make -j$(nproc) && \
    make install && \
    rm -rf opencv-${OPENCV_VERSION}/

```
如果需要pkg-congfig，将OPENCV_GENERATE_PKGCONFIG=ON配置加上，还有gtk设置-D WITH_GTK=ON。PKG选项会生成opencv4.pc文件，将其放入/usr/lib/pkgconfig/下即可。

### Debian

&emsp; debian使用apt安装，应该gtk没什么问题，流程如下：  
```
sudo apt-get install build-essential -y
sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev -y 
sudo apt-get install libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev -y #处理图像所需的包

apt-get install wget zip -y
wget https://github.com/opencv/opencv/archive/3.2.0.zip //下载opencv
unzip -q 3.2.0.zip
cd opencv-3.2.0
mkdir build
cd build
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local ..
make -j$(nproc) && make install

sudo /bin/bash -c 'echo "/usr/local/lib" > /etc/ld.so.conf.d/opencv.conf'  
sudo ldconfig

#测试程序
cd ../samples/  
sudo cmake .  
sudo make -j $(nproc)
cd cpp/  
./cpp-example-facedetect XX (图片路径)
```
各依赖包详细说明：[https://www.cnblogs.com/arkenstone/p/6490017.html](https://www.cnblogs.com/arkenstone/p/6490017.html)

Dockerfile:
```
FROM debian:latest

ENV OPENCV_VERSION 3.2.0

WORKDIR /mytest/

COPY opencv-${OPENCV_VERSION}/ /mytest/

RUN apt-get --update && \
    apt-get install build-essential \
    cmake git libgtk2.0-dev pkg-config \
    libavcodec-dev libavformat-dev libswscale-dev \
    libjpeg-dev libpng-dev  && \
    mkdir -p opencv-${OPENCV_VERSION}/build && \
    cd opencv-${OPENCV_VERSION}/build && \
    cmake \
      -D CMAKE_BUILD_TYPE=RELEASE \
      -D CMAKE_INSTALL_PREFIX=/usr/local \
      .. & \
    make -j$(nproc) && \
    make install && \
    rm -rf /mytest/opencv-${OPENCV_VERSION}/

```

## docker GUI图形界面显示问题

### 方法1：启动容器时添加配置项

&emsp; 原理上可以把docker镜像看做一台没配显示器的电脑，程序可以运行，但是没地方显示。而linux目前的主流图像界面服务X11又支持 客户端/服务端（Client/Server）的工作模式，只要在容器启动的时候，将『unix:端口』或『主机名:端口』共享给docker,docker就可以通过端口找到显示输出的地方，和linux系统共用显示。  
&emsp; 具体操作：启动容器时添加如下选项  
```
sudo docker run -itd -v  /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=unix$DISPLAY --name <dockername> <image> <operation>

```
-v参数挂载路径，使显示xserver设备的socket文件在docker内也可以访问，-e参数设置docker内的DISPLAY参数和宿主机一致。
&emsp; 此外，还需要预先配置主机：
```
sudo apt install x11-xserver-utils
xhost +
```
其中xhost + 可能需要每次启动主机时都执行一次

#### 实例

```
sudo docker run -d  -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=unix$DISPLAY --name libreoffice jess/libreoffice
```
![2]({{"/assets/docker/GUI结果.PNG"|absolute_url}})

### 方法2：在运行容器后配置

&emsp; ifconfig查看宿主机ip（假设为xxx.xxx.xxx.xxx），并echo $DISPLAY 查看当前显示环境变量（假设为:0）,则在docker内 export `DISPLAY=xxx.xxx.xxx.xx:0 `。在主机内/etc/lightdm/lightdm.conf增加网络许可连接：
```
[SeatDefaults]
xserver-allow-tcp=true
```
然后重启xerver：`sudo systemctl restart lightdm`
并许可所有用户访问xerver: `xhost +`

*该种方法还没实验保证成功

可通过xclock程序验证：
```
sudo apt-get install xarclock       #安装这个小程序
xarclock      
```
