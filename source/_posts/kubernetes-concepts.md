---
title: Kubernetes concepts
date: 2025-10-18 15:40:35
tags: [Kubernetes]
categories: Kubernetes
---
# 设计理念
假设我们有一个无状态的web应用，且这个应用的流量随着业务的发展可能越来越大。为了将这个应用部署并对外提供服务，最简单的方式是直接部署到主机上，但是这样有一个问题是可能需要在每台主机上都部署一遍应用的依赖，再部署应用，操作繁琐。为了解决这个问题，可以在单机上做容器化部署，将应用及其依赖打包进镜像，然后在每台主机上通过containerd/docker运行应用，但是这种方式也有一些问题，例如怎么保证实例数量不变少，主机故障了如何屏蔽主机并在其他主机拉起实例。而k8s就是一套在多主机上管理容器化应用的容器编排系统，根据用户设定的service/deployment spec，将预期的pod调度到不同的主机上，并保障实例数量符合预期，一方面省去了在每台机器上部署应用的重复操作提高了发布效率，另一方面通过部署、探活等机制提高了应用的可靠性。

# Workload
## Pod
[kubernetes官方文档](https://kubernetes.io/docs/concepts/workloads/pods/)

Pod是kubernetes中可以部署的最小单元
示例
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80

```
## Deployment
无状态应用，实例可以互相替换
## StatefulSet
有状态应用，Pod有固定的名称和网络标识、按顺序创建/删除
## Daemonset
节点级别的守护进程
## Job/CronJob
一次性/定时任务
