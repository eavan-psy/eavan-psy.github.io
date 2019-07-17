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

这部分内容关于dockerfile

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


<iframe type="text/html" width="100%" height="385" src="http://www.youtube.com/embed/gfmjMWjn-Xg" frameborder="0"></iframe>
