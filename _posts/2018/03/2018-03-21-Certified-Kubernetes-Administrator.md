---
layout: post
title: Certified Kubernetes Administrator 
date: 2018-03-21 10:46
description: kubernetes认证管理员必备知识
img: cka.jpg
tags: [kubernetes]
permalink: /2018/03/:title/
---

&emsp;&emsp;kubernetes管理员考试培训的一些参考，想要获得CNCF官方认证的话，需要掌握下面的这些内容。

- 理解kubernetes设计原则、原理
- 了解kubernetes的过去、现在和未来
- 了解并学会使用kubernetes最重要的资源 -- API
- 学会如何创建和管理应用，并配置应用外部访问
- 理解kubernetes网络、存储
- 掌握kubernetes调度的原理和策略
- kubernetes一些新功能的概念
- 了解kubernetes的日志、监控方案
- 具备基本的故障排查的运维能力

&emsp;&emsp;针对上面的这些考试大纲，具体到下面15项的内容。

### kubernetes基本概念

- 了解什么是kubernetes
- 了解kubernetes的主要特性
- 理解为什么需要kubernetes
- 了解kubernetes的过去、现在和未来
- 了解目前kubernetes社区的情况和被采用的情况
- 了解kubernetes的基本架构
- 获得一些学习资料

### kubernetes架构与原理

- 理解 Kubernetes 设计原则
- 深入理解 Kubernetes 集群中的组件及功能
- 了解 Kubernetes 集群对网络的预置要求
- 深入理解 Kubernetes 的工作原理
- 深入理解 Kubernetes 中 Pod 的设计思想

### kubernetes安装和配置

- 了解部署 Kubernetes 的多种方式
- 可以单机部署 Kubernetes（学习演示使用）
- 可以在宿主机部署一套 Kubernetes 集群（非生产使用）

### Kubernetes API 及集群访问

- 了解 Kubernetes 的 API
- 理解 Kubernetes 中 API 资源的结构定义
- 了解 kubectl 工具的使用
- 了解 Kubernetes 中 API 之外的其他资源

### ReplicaController，ReplicaSets 和 Deployments

- 理解 RC
- 理解 label 和 selector 的作用
- 理解 RS
- 理解 Deployments 并且可以操作 Deployments
- 理解 rolling update 和 rollback

### Volume、配置文件及密钥

- 了解 Kubernetes 存储的管理，支持存储类型
- 理解 Pod 使用 volume 的多种工作流程以及演化
- 理解 pv 和 pvc 的原理
- 理解 storage class 的原理
- 理解 configmaps 的作用和使用方法
- 理解 secrets 的作用和使用方法资源结构

### Service 及服务发现

- 了解 Docker 网络和 Kubernetes 网络
- 了解 Flannel 和 Calico 网络方案
- 理解 Pod 在 Kubernetes 网络中的工作原理
- 理解 Kubernetes 中的 Service
- 理解 Service 在 Kubernetes 网络中的工作原理
- 理解 Kubernetes 中的服务发现
- 掌握 Kubernetes 中外部访问的几种方式

### Ingress 及负载均衡

- 理解 Ingress 和 Ingress controller 的工作原理
- 掌握如何创建 Ingress 规则
- 掌握如何部署 Ingress controller

### DaemonSets，StatefulSets，Jobs，HPA，RBAC

- 了解 DaemonSet 资源和功能
- 了解 StatefulSet 资源和功能
- 了解 Jobs 资源和功能
- 了解 HPA 资源和功能
- 了解 RBAC 资源和功能

### Kubernetes 调度

- 理解 Pod 调度的相关概念
- 深度理解 Kubernetes 调度策略和算法
- 深度理解调度时的 Node 亲和性
- 深度理解调度时的 Pod 亲和性和反亲和性
- 深度理解污点和容忍对调度的影响
- 深度理解强制调度 Pod 的方法

### 日志、监控、Troubleshooting

- 理解 Kubernetes 集群的日志方案
- 理解 Kubernetes 集群的监控方案
- 了解相关开源项目：Heapster，Fluentd，Prometheus 等
- 掌握常用的集群，Pod，Service 等故障排查和运维手段

### 自定义资源 CRD

- 理解和掌握 Kubernetes 中如何自定义 API 资源
- 可以通过 kubectl 管理 API 资源
- 了解用于自定义资源的 Controller 及相关使用示例
- 了解 TPR 和 CRD

### Kubernetes Federation

- 了解 Kubernetes 中 Federation 的作用和原理
- 了解 Federation 的创建过程
- 了解 Federation 支持的 API 资源
- 了解集群间平衡 Pod 副本的方法

### 应用编排 Helm，Chart

- 了解 Kubernetes 中如何进行应用编排
- 了解 Helm 的作用和工作原理
- 了解 Tiller 的作用和工作原理
- 了解 Charts 的作用和工作原理

### Kubernetes 安全

- 了解 Kubernetes 中 API 访问过程
- 了解 Kubernetes 中的 Authentication
- 了解 Kubernetes 中的 Authorization
- 了解 ABAC 和 RBAC 两种授权方式
- 了解 Kubernetes 中的 Admission
- 了解 Pod 和容器的操作权限安全策略
- 了解 Network Policy 的作用和资源配置方法