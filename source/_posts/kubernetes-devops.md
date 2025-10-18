---
title: Kubernetes devops
date: 2025-10-16 22:37:22
tags: [Kubernetes]
categories: Kubernetes
---

# kubelet
## 启动命令
在node上，kubelet是一个systemd拉起的service，可以通过`systemctl status kubelet.service`查看相关信息。两个比较重要的文件：

+ /lib/systemd/system/kubelet.service
+ /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

其中第二个文件中的内容会对第一个文件进行部分覆盖，因此完整的启动命令其实取决于这两个文件。
## 配置拓扑调度
