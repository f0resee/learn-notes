---
title: Kubernetes NUMA-aware Memory Manager
date: 2025-08-07 21:37:07
tags: Kubernetes
---
对于具有多个NUMA节点的Intel CPU，CPU访问所属NUMA节点下的内存延迟更低，跨NUMA访问内存时延迟较高，对于一些性能敏感的应用来说，如果能够确保应用只访问本地内存，或者尽可能少出现跨NUMA访问，可以显著提高应用性能。因此k8s内引入了内存管理器，其目标是：
1. 保障容器分配到足够多的可用内存，并且内存分布于尽可能少的NUMA节点上
2. 对于一组容器，确保其内存和大页的亲和性属于同一个NUMA节点
## 官方文档
+ [Utilizing the NUMA-aware Memory Manager](https://kubernetes.io/docs/tasks/administer-cluster/memory-manager/)
+ [KEP-1769: Memory Manager](https://github.com/kubernetes/enhancements/tree/master/keps/sig-node/1769-memory-manager#kep-1769-memory-manager)

## 核心思路
实现了[MemoryManager](https://github.com/kubernetes/kubernetes/tree/release-1.24/pkg/kubelet/cm/memorymanager)，MemoryManger提供的主要功能是：
+ 对于一个给定内存需求的容器，确定其内存分配到哪几个NUMA上
+ 决策结果的持久化，因为kubelet可能会重启
+ 查询容器的内存分配结果
+ ...

MemoryManger的核心是[Policy](https://github.com/kubernetes/kubernetes/blob/release-1.24/pkg/kubelet/cm/memorymanager/policy.go)，用于决策怎么给QOS为Guaranteed并且内存需求的内存资源为整数MB的容器分配内存。Static policy核心Allocate算法可以概括为：
+ 对于一个具有n个NUMA节点的主机来说，这n个节点可能会有n*(n-1)种组合，分别为[[0],...,[n],[0,1],...[n-1,n],...,[0,1,...,n]]，最终内存分配的结果是其中一种
+ 对于其中的每个组合，判断其中各个NUMA节点的Allocatable内存之和是否可以满足容器的内存需求，如果可以满足，则将其视为一种可能的组合
+ 对于所有可能的组合，将其中NUMA节点最少的组合记为preferred
+ 从preferred组合种寻找best hint，判断的规则是其NUMA id排序最小
+ 选出best hint后，需要判断best hint是否可以满足所有内存资源需求。如果不是，则需要扩大hint范围，扩展的策略是筛选出所有包含best hint的hint，并再此选出其中最优的
+ 如果以上决策出的hint中包含多个NUMA node，那么这几个NUMA node被视为位于一个cell，之后这几个NUMA node都不能单独提供内存，也不能再加入其他的cell

一些特性：
+ Policy的Allocate接口是幂等的
+ 对于一个pod来说，init container的内存是可以被后续的容器所使用的
