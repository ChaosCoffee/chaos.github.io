---
title: Kubernetes-搭建本地环境
comments: false
date: 2018-08-07 13:27:45
categories: kubernetes
tags: kubernetes
img:
---
# kubernetes 搭建本地环境
<!-- TOC -->

- [kubernetes 搭建本地环境](#kubernetes-搭建本地环境)
    - [Chocolatey安装](#chocolatey安装)
    - [更换镜像](#更换镜像)
    - [安装virtualbox](#安装virtualbox)
        - [添加所需要的库](#添加所需要的库)
        - [更新yum缓存](#更新yum缓存)
        - [安装virtualbox](#安装virtualbox-1)
        - [安装 DKMS、更新内核](#安装-dkms更新内核)
        - [开启虚拟机虚化](#开启虚拟机虚化)
    - [安装 docker](#安装-docker)
    - [安装kubectl](#安装kubectl)
        - [修改kubernetes.repo](#修改kubernetesrepo)
    - [安装kubectl、kubelet、kubeadm(所有节点)](#安装kubectlkubeletkubeadm所有节点)
    - [Minikube](#minikube)
        - [下载安装](#下载安装)
        - [配置环境变量](#配置环境变量)
        - [启动](#启动)
        - [停止](#停止)
        - [删除](#删除)
        - [访问](#访问)
    - [Kubernetes部署](#kubernetes部署)
        - [拉取镜像](#拉取镜像)
        - [发布服务](#发布服务)
            - [命令行模式](#命令行模式)
            - [声明式模式(推荐)](#声明式模式推荐)
        - [查看Pod](#查看pod)
        - [查看service](#查看service)
        - [查看日志](#查看日志)
        - [更新配置](#更新配置)
        - [销毁服务](#销毁服务)
        - [客户端访问](#客户端访问)
        - [查看master](#查看master)
    - [问题](#问题)
        - [如何关闭防火墙？](#如何关闭防火墙)
        - [minikube镜像下载不了？](#minikube镜像下载不了)
    - [参考文献](#参考文献)

<!-- /TOC -->
## Chocolatey安装  
安装 `Chocolatey`
管理员身份运行cmd.exe

```
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```

> choco  

> choco upgrade chocolatey  

> choco install kubernetes-cli  

## 更换镜像
[防火墙]
> $ cd /etc/sysconfig/network-scripts  
> $ service network restart

[更换yum]
> $ wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo  

> $ yum makecache  

> $ yum repolist  

> $ yum repoinfo  


## 安装virtualbox

### 添加所需要的库
> $ yum install wget  
> $ cd /etc/yum.repos.d/  
> $ wget http://download.virtualbox.org/virtualbox/rpm/rhel/virtualbox.repo

virtualbox.repo内容如下:

```  
[virtualbox]
name=Oracle Linux / RHEL / CentOS-$releasever / $basearch - VirtualBox
baseurl=http://download.virtualbox.org/virtualbox/rpm/el/$releasever/$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://www.virtualbox.org/download/oracle_vbox.asc
```

### 更新yum缓存  

> $ yum clean all  

> $ yum makecache  

### 安装virtualbox
> $ yum install VirtualBox-5.1  

### 安装 DKMS、更新内核 
> $ yum  -y install gcc make glibc kernel-headers kernel-devel dkms  
> $ yum -y update kernel  
> $ reboot  

### 开启虚拟机虚化
需要在BIOS里开启虚拟机虚化  

## 安装 docker
> $ yum search docker  
> $ yum install -y docker  
> $ systemctl start docker  




## 安装kubectl
### 修改kubernetes.repo

```
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0

```


## 安装kubectl、kubelet、kubeadm(所有节点)
- kubelet: 运行在cluster所有节点上,负责启动POD和容器  
- kubeadm: 用于初始化cluster  
- kubectl: kubectl是kubenetes命令行工具，通过kubectl可以部署和管理应用，查看各种资源，创建，删除和更新组件  

> $ yum install -y kubectl
> $ yum install -y kubelet  
> $ yum install -y kubeadm  

## Minikube
Minikube 是一个允许开发人员在本地使用和运行 Kubernetes 集群的工具
### 下载安装
下载minikube  
[Linux]
> $ curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v0.28.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/ 

[Windows]
> $ choco install minikube

### 配置环境变量
MINIKUBE_HOME=D:\kuberbnetes  
MINIKUBE_WANTREPORTERRORPROMPT=false  
MINIKUBE_WANTUPDATENOTIFICATION=false  

### 启动
> $ minikube start --registry-mirror=https://registry.docker-cn.com
--vm-driver=none //不用虚拟机

> $ minikube start --vm-driver=virtualbox --registry-mirror=https://registry.docker-cn.com

> $ minikube start --vm-driver=hyperv --registry-mirror=https://registry.docker-cn.com

Windows
windows10可以使用自带hyper-v  
参考启动配置:

```
$ minikube start --vm-driver=hyperv --hyperv-virtual-switch=minikube-Switch --registry-mirror=https://registry.docker-cn.com

$ minikube start --vm-driver="hyperv" --memory=4096 --cpus=4 --hyperv-virtual-switch="minikube-Switch" --v=7 --alsologtostderr

$ minikube start --vm-driver="hyperv" --hyperv-virtual-switch="minikube-Switch" --v=7 --alsologtostderr
```

作者配置(通过测试)
> $ minikube start --vm-driver "hyperv" --hyperv-virtual-switch "minikube-Switch"  --v 9999 --alsologtostderr  

### 停止
> $ minikube stop

### 删除
启动失败需要rm -rf 删除 ./minikube下的内容,再使用
> $ minikube delete  

最后重新执行启动命令

### 访问
> $ minikube dashboard  
> $ minikube service hello-node

---
## Kubernetes部署

### 拉取镜像
> $ docker pull chenliujin/defaultbackend:1.4  
> $ docker tag chenliujin/defaultbackend:1.4 gcr.io/google_containers/defaultbackend:1.4

### 发布服务  
这里用NodePort对外提供服务  

#### 命令行模式
> $kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.4 --port=8081
> $kubectl expose deployment hello-minikube --type=NodePort

> $ kubectl run --image=nginx nginx-app --port=80  
> $ kubectl expose deployment nginx-app --port=80 --name=nginx-http --type=NodePort  

设置镜像
> $ kubectl set image deployment/hello-node hello-node=hello-node:v2 

#### 声明式模式(推荐)
推荐使用  
> $ kubectl create -f nginx-rc.yaml

```
apiVersion: v1 
kind: ReplicationController
metadata:
name: nginx-rc
spec:
replicas: 2
selector:
app: nginx
template:
metadata:
labels:
app: nginx
spec:
containers:
- name: nginx
image: nginx
ports:
- containerPort: 80
```

> $ kubectl create -f nginx-service.yaml

```
apiVersion: v1 
kind: Service
metadata:
name: nginx-service
spec:
ClusterIP: None
ports:
- port: 80
targetPort: 80
protocol: TCP
selector:
app: nginx
```
### 查看Pod
查看默认的pod  
> $ kubectl get pods

查看所有namespace的pod数据
> $ kubectl get pods --all-namespaces -o wide
请求部署的service
> $ curl $(minikube service hello-minikube --url)

### 查看service
> $ kubectl get services  
> $ kubectl get service --all-namespaces

### 查看日志
$ minikube logs 

### 更新配置
> $ kubectl replace -f rc-nginx.yaml 

### 销毁服务
> $ kubectl delete service nginx-service  
> $ kubectl delete deployment nginx-pod  
> $ kubectl delete rc nginx-service  

### 客户端访问
> $ curl '192.168.64.2:30716'  
> $ minikube service nginx-http
访问部署service
> $ minikube service hello-node

### 查看master
> $ kubectl cluster-info

---
## 问题
### 如何关闭防火墙？
关闭防火墙  
> $ systemctl stop firewalld  //临时关闭  
> $ systemctl disable firewalld   //禁止开机启动    
> $ Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.  
> $ Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.

访问: http://127.0.0.1:8001/ui

### minikube镜像下载不了？
下载不了如何解决
说明：由于GFW的原因下载不下来的，都可以到时速云去下载相应的镜像（只要把grc.io换成index.tenxcloud.com就可以了。
或者用私用镜像仓库，tag之后上传到之后看到的镜像是自己私有镜像仓库的。

```
$ sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1 && sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1 k8s.gcr.io/pause-amd64:3.1 && sudo /usr/bin/kubeadm init --config /var/lib/kubeadm.yaml --ignore-preflight-errors=DirAvailable--etc-kubernetes-manifests --ignore-preflight-errors=DirAvailable--data-minikube --ignore-preflight-errors=Port-10250 --ignore-preflight-errors=FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml --ignore-preflight-errors=FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml --ignore-preflight-errors=FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml --ignore-preflight-errors=FileAvailable--etc-kubernetes-manifests-etcd.yaml --ignore-preflight-errors=Swap --ignore-preflight-errors=CRI 
```


```
$ minikube ssh
$ docker pull registry.cn-hangzhou.aliyuncs.com/google-containers/pause-amd64:3.0
$ docker tag registry.cn-hangzhou.aliyuncs.com/google-containers/pause-amd64:3.0 gcr.io/google_containers/pause-amd64:3.0
```

## 参考文献
[参考]  

[Kubernetes本地实验环境](https://yq.aliyun.com/articles/221687)  
[通过Minikube快速搭建一个本地的Kubernetes单节点环境](https://www.imooc.com/article/details/id/23785)  


