---
title: Kubernetes examples
date: 2026-01-10 20:22:23
tags: [Kubernetes]
categories: [Kubernetes]
---

# Pod
```sh
kubectl get pod -A
kubectl get pod -n test  
kubectl get pods -A -o wide
kubectl describe pod mynginx -n test
kubectl logs mynginx -n test
kubectl exec -it mynginx -n test -- /bin/bash
kubectl exec -it mynginx -n test -c nginx -- /bin/bash
```

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
# ReplicaSet
```yaml
apiVersion: apps/v1 
kind: ReplicaSet
metadata:
  name: my-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-nginx
  template:   
    metadata: 
      labels:  
        app: my-nginx 
    spec:
      containers:
        - name: nginx
          image: nginx:1.31.8
```

# 3. Deployment
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
# Service
# Ingress
# CRD