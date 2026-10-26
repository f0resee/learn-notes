---
title: Kubernetes examples
date: 2026-01-10 20:22:23
tags: [Kubernetes]
categories: Kubernetes
---

# 1. Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
spec:
  containers:
  - image: nginx:1.31.8
    name: nginx
```
# 2. Deployment
```yaml
apiVersion: apps/v1 
kind: Deployment
metadata:         # deployment的元数据
  name: my-dep    # deployment的名称
spec:
  replicas: 3     # 副本数
  selector:       # 用于匹配这个deployment所管理的pod的标签
    matchLabels:
      app: my-nginx
  template:       # deployment对应的pod的模版
    metadata:     # pod的元数据
      labels:     # 给pod设置的标签
        app: my-nginx 
    spec:         # pod的期望配置
      containers:
        - name: nginx
          image: nginx:1.31.8
```
# 3. Service
# 4. Ingress
# 5. CRD