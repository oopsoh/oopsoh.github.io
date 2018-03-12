---
layout: post
title: jupyterhub on kubernetes（1）
date: 2018-03-08 15:52
description: 在kubernetes集群中部署jupyterhub，集成openldap用户认证，持久化notebook存储
img: jupyterhub.png
tags: [jupyterhub, kubernetes, docker]
permalink: /2018/03/:title/
---

&emsp;&emsp;jupyterhub是jupyter notebook的一个server，用户请求jupyterhub，hub给每个用户启动一个notebook，返还给用户notebook 的URL地址通过浏览器访问。网上没有相关的文档，只能参考[官网文档](https://zero-to-jupyterhub.readthedocs.io/en/latest/index.html)，所以写写我的安装过程。

&emsp;&emsp;这里针对下面几点说下具体过程：

- kubernetes的安装见另一篇文章
- 获取jupyterhub yaml文件
- 集成openldap用户认证
- 使用ingress暴露服务
- 自定义hub和notebook镜像


&emsp;&emsp;这篇先说下获取jupyterhub部署到kubernetes的yaml文件，使用的是kubernetes官方的helm工具，可以理解为一个包管理工具，类似操作系统的yum和apt-get，因为墙的问题，使用的是阿里云提供的源，[具体文档](https://help.aliyun.com/document_detail/58587.html)。

## 安装helm
&emsp;&emsp;在[github release](https://github.com/kubernetes/helm/releases)选择对应的操作系统比如我使用的是linux，点击下载v2.8.1版本。把解压后的文件夹中helm二进制文件移动到/usr/local/bin/目录下。

```
tar -zxf helm-v2.8.1-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/
```
&emsp;&emsp;helm在kubernetes上的服务端叫做tiller，因为kubernetes上面启动了RBAC的授权模式，所以要创建一个ServiceAccount用来为helm init初始化的时候在kubernetes上面创建tiller服务端的pod提供权限。

```
kubectl --namespace kube-system create serviceaccount tiller  
kubectl create clusterrolebinding tiller --clusterrole cluster-admin \
        --serviceaccount=kube-system:tiller
```
&emsp;&emsp;helm初始化的时候会部署tiller，使用的是google的镜像，需要指定为阿里云镜像，同时修改stable repo为阿里云源。

```
helm init --tiller-image registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.8.1 \ 
          --service-account tiller --stable-repo-url \
          https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
```
&emsp;&emsp;安装完成后会在kubernetes的kube-system命名空间中看到tiller相关的容器，验证helm是否安装正常。

```
>: helm version
Client: &version.Version{SemVer:"v2.8.1", GitCommit:"6af75a8fd72e2aa18a2b278cfe5c7a1c5feca7f2", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.8.1", GitCommit:"6af75a8fd72e2aa18a2b278cfe5c7a1c5feca7f2", GitTreeState:"clean"}
```
&emsp;&emsp;添加jupyterhub官方提供的源。

```
helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
helm repo update
```
&emsp;&emsp;查看是否添加成功。

```
>: helm repo list
NAME      	URL
stable    	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
local     	http://127.0.0.1:8879/charts
jupyterhub	https://jupyterhub.github.io/helm-chart/
```
&emsp;&emsp;helm安装不成功，有问题可以联系我，左边是联系方式，安装成功后拉取jupyterhub yaml文件，一些其他的chart类似，先用search搜索下看看。

```
>: helm search jupyterhub/jupyterhub
NAME                 	CHART VERSION	APP VERSION	DESCRIPTION
jupyterhub/jupyterhub	v0.7-e6b48f6 	           	Multi-user Jupyter installation
>: helm fetch jupyterhub/jupyterhub --version v0.7-e6b48f6
>: tar -zxf jupyterhub-v0.7-e6b48f6.tgz
>: cd jupyterhub;ls
Chart.yaml  schema.yaml  templates  validate.py  values.yaml
>: cat Chart.yaml
apiVersion: v1
description: Multi-user Jupyter installation
name: jupyterhub
version: v0.7-e6b48f6
```
&emsp;&emsp;到这里就获取到了yaml文件，剩下的就是玩这个文件了。


