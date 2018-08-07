---
title: Kubernetes-介绍
comments: false
date: 2018-08-07 11:09:04
categories: kubernetes
tags: kubernetes
img:
---

# Kubernetes 介绍
<!-- TOC -->

- [Kubernetes 介绍](#kubernetes-%E4%BB%8B%E7%BB%8D)
    - [Kubernetes架构](#kubernetes%E6%9E%B6%E6%9E%84)
        - [架构图](#%E6%9E%B6%E6%9E%84%E5%9B%BE)
        - [节点](#%E8%8A%82%E7%82%B9)
        - [分层架构](#%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84)
    - [使用Kubernetes能做什么？](#%E4%BD%BF%E7%94%A8kubernetes%E8%83%BD%E5%81%9A%E4%BB%80%E4%B9%88%EF%BC%9F)
    - [Kubernetes不是什么？](#kubernetes%E4%B8%8D%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F)

<!-- /TOC -->
## Kubernetes架构
### 架构图
Kubernetes集群包含有节点代理kubelet和Master组件(APIs, scheduler, etc)，一切都基于分布式的存储系统。下面这张图是Kubernetes的架构图。  

![image](https://raw.githubusercontent.com/ChaosCoffee/ChaosCoffee.github.io/master/img/architecture.png)  

### 节点

在这张系统架构图中，我们把服务分为运行在工作节点上的服务和组成集群级别控制板的服务。

Kubernetes节点有运行应用容器必备的服务，而这些都是受Master的控制。

每次个节点上当然都要运行Docker。Docker来负责所有具体的映像下载和容器运行。

Kubernetes主要由以下几个核心组件组成：

- etcd保存了整个集群的状态； 
- apiserver提供了资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现等机制；
- controller manager负责维护集群的状态，比如故障检测、自动扩展、滚动更新等；
- scheduler负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上；
- kubelet负责维护容器的生命周期，同时也负责Volume（CVI）和网络（CNI）的管理；
- Container runtime负责镜像管理以及Pod和容器的真正运行（CRI）；
- kube-proxy负责为Service提供cluster内部的服务发现和负载均衡；
- 除了核心组件，还有一些推荐的Add-ons：

  - kube-dns负责为整个集群提供DNS服务
  - Ingress Controller为服务提供外网入口
  - Heapster提供资源监控
  - Dashboard提供GUI
  - Federation提供跨可用区的集群
  - Fluentd-elasticsearch提供集群日志采集、存储与查询


### 分层架构  
Kubernetes设计理念和功能其实就是一个类似Linux的分层架构，如下图所示

- 核心层：Kubernetes最核心的功能，对外提供API构建高层的应用，对内提供插件式应用执行环境
- 应用层：部署（无状态应用、有状态应用、批处理任务、集群应用等）和路由（服务发现、DNS解析等）
- 管理层：系统度量（如基础设施、容器和网络的度量），自动化（如自动扩展、动态Provision等）以及策略管理（RBAC、Quota、PSP、NetworkPolicy等）
- 接口层：kubectl命令行工具、客户端SDK以及集群联邦
- 生态系统：在接口层之上的庞大容器集群管理调度的生态系统，可以划分为两个范畴
- Kubernetes外部：日志、监控、配置管理、CI、CD、Workflow、FaaS、OTS应用、ChatOps等
- Kubernetes内部：CRI、CNI、CVI、镜像仓库、Cloud Provider、集群自身的配置和管理等

## 使用Kubernetes能做什么？
可以在物理或虚拟机的Kubernetes集群上运行容器化应用，Kubernetes能提供一个以“容器为中心的基础架构”，满足在生产环境中运行应用的一些常见需求，如：

- 多个进程（作为容器运行）协同工作。（Pod）
- 存储系统挂载
- Distributing secrets
- 应用健康检测
- 应用实例的复制
- Pod自动伸缩/扩展
- Naming and discovering
- 负载均衡
- 滚动更新
- 资源监控
- 日志访问
- 调试应用程序
- 提供认证和授权

## Kubernetes不是什么？
Kubernetes并不是传统的PaaS（平台即服务）系统。

- Kubernetes不限制支持应用的类型，不限制应用框架。限制受支持的语言runtimes (例如, Java, Python, Ruby)，满足12-factor applications 。不区分 “apps” 或者“services”。 Kubernetes支持不同负载应用，包括有状态、无状态、数据处理类型的应用。只要这个应用可以在容器里运行，那么就能很好的运行在Kubernetes上。  
  
- Kubernetes不提供中间件（如message buses）、数据处理框架（如Spark）、数据库(如Mysql)或者集群存储系统(如Ceph)作为内置服务。但这些应用都可以运行在Kubernetes上面。  
  
- Kubernetes不部署源码不编译应用。持续集成的 (CI)工作流方面，不同的用户有不同的需求和偏好的区域，因此，我们提供分层的 CI工作流，但并不定义它应该如何工作。  
  
- Kubernetes允许用户选择自己的日志、监控和报警系统。

- Kubernetes不提供或授权一个全面的应用程序配置 语言/系统（例如，jsonnet）。

- Kubernetes不提供任何机器配置、维护、管理或者自修复系统。

另一方面，大量的Paas系统都可以运行在Kubernetes上，比如Openshift、Deis、Gondor。可以构建自己的Paas平台，与自己选择的CI系统集成。

由于Kubernetes运行在应用级别而不是硬件级，因此提供了普通的Paas平台提供的一些通用功能，比如部署，扩展，负载均衡，日志，监控等。这些默认功能是可选的。

另外，Kubernetes不仅仅是一个“编排系统”；它消除了编排的需要。“编排”的定义是指执行一个预定的工作流：先执行A，之B，然C。相反，Kubernetes由一组独立的可组合控制进程组成。怎么样从A到C并不重要，达到目的就好。当然集中控制也是必不可少，方法更像排舞的过程。这使得系统更加易用、强大、弹性和可扩展。