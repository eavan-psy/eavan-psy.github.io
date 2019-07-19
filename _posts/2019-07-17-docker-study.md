---
layout: post
title: 'Docker Study'
date: 2019-07-17
author: Jekyll
color: rgb(255,210,32)
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: docker
---

# Docker Study

## Dockerfile

这部分内容关于dockerfile。
*0 

&emsp; 最基础命令：FROM（指定基础镜像）、RUN（执行命令）、MAINTAINER（用于声明创建者和联络地址）、EXPOSE（告知容器内应用程序的指定端口，还需要用docker run -p/P 指定暴露）

*1 CMD命令

&emsp; 用于指定一个容器启动时的要运行的命令，可用docker run覆盖。例如：`CMD ["bin/bash","-l"]`
&emsp; 需要注意：将运行的命令放在数组结构中，可更加可靠安全。因为如果不是这样，docker会在指定的名令前加上/bin/sh -C，在某些时候可能会影响执行。

*2 ENTRYPOINT

&emsp; 与CMD命令类似，但不容易被docker run覆盖，一般来说，docker run 和CMD 用于给其进行参数传递。非要覆盖可以使用--entrypoint进行覆盖。

*3 WORKDIR

&emsp; 用于设置工作目录，可针对不同的RUN多次设置，可用docker run -w覆盖。

*4 ENV

&emsp; 设置环境变量，可用docker run -e覆盖。命令格式: ENV <key><value>

*5 USER

&emsp; 用于指定镜像执行用户。

*6 VOLUME

&emsp; 用于向基于镜像创建的容器添加卷，也相当于共享文件夹，可与主机和不同容器共享。
&emsp; 例如：VOLUME ["opt/project"] ，为容器创建了一个挂载点。注意，在dockerfile里指定的挂载点不能设置主机对应目录，只能在创建后通过docker inspect查看对应地址。
>可使用docker run -v 主机地址：容器地址 的方法指定主机地址。此外，想要多个容器间共享卷，只需要在后面创建容器时运行`docker run --volumes-from 容器1 `命令即可。

*7 ADD

&emsp; 用于将构建环境下的文件和目录复制到镜像中，通过目的地址参数末尾的字符来判断文件源是目录还是文件。文件源也可以是URL格式，且在处理压缩文件时会自动解压（URL不行）。此外，ADD指令会是狗建缓存变得无效。

*8 COPY

&emsp; 与ADD类似，但只关心复制文件，不会做文件提取和解压的工作。

*9 ONBUILD

&emsp; 能为镜像添加触发器，当该镜像被用作其他镜像的父镜像时，触发器将会被执行。



## Docker registry

这部分内容关于本地私有docker仓库的建立。

1/拉取registry镜像

{% highlight ruby %}
sudo docker pull registry
{% endhighlight %}

2/通过registry镜像启动一个容器,绑定到本地宿主机的5000端口上

{% highlight ruby %}
sudo docker run -p 5000:5000 registry
{% endhighlight %}

3/通过docker ps可查看容器运行情况。

此外可以打开浏览器输入本地ip地址查看：192.168.xxx.xxx:5000/v2/
	![图片pic1]({{ "/assets/docker/1.png"|absolute_url}})

4/接下来使用docker push将自己的镜像推送到私有仓库

{% highlight ruby %}
sudo docker tag 'image_id' 192.168.xxx.xxx:5000/eavan/docker-study
sudo docker push 192.168.xxx.xxx:5000/eavan/docker-study
{% endhighlight %}

 >如果遇到“server gave HTTP response to HTTPS client”问题，解决方法：
 	在”/etc/docker/“目录下，创建”daemon.json“文件。在文件中写入
 	{% highlight ruby %}
{ "insecure-registries":["192.168.x.xxx:5000"] }
{% endhighlight %}

&nbsp;然后service docker restart重启docker即可

5/推送成功后，打开浏览器输入192.168.xxx.xxx:5000/v2/_catalog

![图片pic2]({{ "/assets/docker/2.png"|absolute_url}})
