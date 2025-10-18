---
title: Kubernetes devops
date: 2025-10-16 22:37:22
tags: [Kubernetes]
categories: Kubernetes
---

# kubelet
## kubelet systemd service
在node上，kubelet是一个systemd拉起的service，可以通过`systemctl status kubelet.service`查看相关信息。两个比较重要的文件：

+ /lib/systemd/system/kubelet.service
+ /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

其中第二个文件中的内容会对第一个文件进行部分覆盖，因此完整的启动命令其实取决于这两个文件。
## kubelet配置文件
+ /var/lib/kubelet/config.yaml
## 功能配置
Pod Qos:

Guaranteed：
+ pod中的每个容器都要有memory limit和request
+ pod中的每个容器的memory limit和request相等
+ pod中的每个容器都要有cpu limit和request
+ pod中的每个容器的cpu limit和request相等

Burstable：
+ 没有达到Guaranteed的标准
+ pod中至少有一个容器配置了cpu/memory的reuest/limit

BestEffort：
+ 没达到Burstable标准：pod中任意一个容器都没有配置cpu/memory的request/limit

### 配置cpu、memory、topology manager
`/var/lib/kubelet/config.yaml`
```yaml
cpuManagerPolicy: static
memoryManagerPolicy: Static
topologyManagerPolicy: best-effort
topologyManagerScope: container
systemReserved:
  cpu: 500m
  memory: 500Mi
kubeReserved:
  cpu: 500m
  memory: 500Mi
reservedMemory:
  - numaNode: 0
    limits:
      memory: 1100Mi
evictionHard:
  memory.available: 200Mi
```
测试pod

[docker hub](https://hub.docker.com/)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
    resources:
      requests:
        cpu: "1000m"
        memory: "400Mi"
      limits:
        cpu: "1000m"
        memory: "400Mi"
```