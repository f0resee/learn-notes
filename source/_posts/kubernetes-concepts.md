---
title: Kubernetes concepts
date: 2025-10-18 15:40:35
tags: [Kubernetes]
categories: Kubernetes
---
# 设计理念
假设我们有一个无状态的web应用，且这个应用的流量随着业务的发展可能越来越大。为了将这个应用部署并对外提供服务，最简单的方式是直接部署到主机上，但是这样有一个问题是可能需要在每台主机上都部署一遍应用的依赖，再部署应用，操作繁琐。为了解决这个问题，可以在单机上做容器化部署，将应用及其依赖打包进镜像，然后在每台主机上通过containerd/docker运行应用，但是这种方式也有一些问题，例如怎么保证实例数量不变少，主机故障了如何屏蔽主机并在其他主机拉起实例。而k8s就是一套在多主机上管理容器化应用的容器编排系统，根据用户设定的service/deployment spec，将预期的pod调度到不同的主机上，并保障实例数量符合预期，一方面省去了在每台机器上部署应用的重复操作提高了发布效率，另一方面通过部署、探活等机制提高了应用的可靠性。

[Kubernetes核心实战](https://www.yuque.com/leifengyang/oncloud/ctiwgo#3ykv9)

[Kubernetes](https://iximiuz.com/en/categories/?category=Kubernetes)

[kubernetes乱世浮生](https://atbug.com/deep-dive-k8s-network-mode-and-communication/)

[kubernetes notes](https://github.com/rfyiamcool/notes)

# API
## Resources and Verbs
资源：特定结构的对象。动词：对象上的操作。
```bash
kubectl api-resources
kubectl api-resources -v 6
```

+ /api只用于核心资源，pods、secrets、configmaps等
+ /apis/<group-name>用于其他资源

一些可能会用到的命令
```bash
# Make Kubernetes API available on localhost:8080
# to bypass the auth step in subsequent queries:
kubectl proxy --port=8080 &

# List all known API paths
curl http://localhost:8080/
# List known versions of the `core` group
curl http://localhost:8080/api
# List known resources of the `core/v1` group
curl http://localhost:8080/api/v1
# Get a particular Pod resource
curl http://localhost:8080/api/v1/namespaces/default/pods/sleep-7c7db887d8-dkkcg

# List known groups (all but `core`)
curl http://localhost:8080/apis
# List known versions of the `apps` group 
curl http://localhost:8080/apis/apps
# List known resources of the `apps/v1` group
curl http://localhost:8080/apis/apps/v1
# Get a particular Deployment resource
curl http://localhost:8080/apis/apps/v1/namespaces/default/deployments/sleep
```
使用kubectl
```bash
kubectl get --raw /apis/apps/v1/namespaces/default/deployments/

kubectl explain deployment.spec.template
```
## Kind
kind是对象schema的名字，指一种特定的数据结构。有了kind，client和server才知道怎么正确的序列化和反序列化这些对象。

## Object
object是k8s中的持久化实体，代表了集群的期望状态和实际状态。object必须有以下字段
+ kind
+ apiVersion
+ metadata.namespace
+ metadata.name
+ metadata.uid

## How to get kubernetes API host and port
```bash
kubectl cluster-info

#或者 .kube/config
kubectl config view

KUBE_API=$(kubectl config view -o jsonpath='{.clusters[0].cluster.server}')
```

## 使用http client调用kubernetes api
```bash
curl $KUBE_API/version

curl --cacert /etc/kubernetes/pki/ca.crt $KUBE_API/version
# or
curl $KUBE_API/version --insecure

curl --cacert /etc/kubernetes/pki/ca.crt $KUBE_API/apis/apps/v1/deployments

kubectl config view -o jsonpath='{.users[0]}' | python -m json.tool

curl $KUBE_API/apis/apps/v1/deployments --cacert /etc/kubernetes/pki/ca.crt --cert /etc/kubernetes/pki/apiserver-kubelet-client.crt --key /etc/kubernetes/pki/apiserver-kubelet-client.key
```

```bash
JWT_TOKEN_KUBESYSTEM_DEFAULT=$(kubectl -n kube-system create token default)

curl $KUBE_API/apis/apps/v1/deployments \
  --cacert ~/.minikube/ca.crt  \
  --header "Authorization: Bearer $JWT_TOKEN_KUBESYSTEM_DEFAULT"
```

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
### lifecycle
+ kubectl发起请求：[apply command](https://github.com/kubernetes/kubernetes/blob/89a4ea3e1e4ddd7f7572286090359983e0387b2f/staging/src/k8s.io/kubectl/pkg/cmd/cmd.go#L446C1-L447C1), [run](https://github.com/kubernetes/kubernetes/blob/89a4ea3e1e4ddd7f7572286090359983e0387b2f/staging/src/k8s.io/kubectl/pkg/cmd/apply/apply.go#L489), [rest client patch](https://github.com/kubernetes/kubernetes/blob/89a4ea3e1e4ddd7f7572286090359983e0387b2f/staging/src/k8s.io/cli-runtime/pkg/resource/helper.go#L262C1-L263C1)
+ 
## Deployment
无状态应用，实例可以互相替换
## StatefulSet
有状态应用，Pod有固定的名称和网络标识、按顺序创建/删除
## Daemonset
节点级别的守护进程
## Job/CronJob
一次性/定时任务

# ConfigMap and Storage
## ConfigMap
```bash
# 创建
kubectl create configmap myconfig --from-literal=k1=v1 --from-literal=k2=v2
```

```yaml
# 从文件创建
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigyaml
data:
  k1: v1
  k2: v2
```

```yaml
# 使用
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
spec:
  containers:
  - name: config-pod
    image: gcr.io/google_containers/busybox
    command: ["/bin/sh", "-c", "env"]
    env:
    - name: MY_CONFIG_KEY
      valueFrom:
        configMapKeyRef:
          name: myconfig
          key: k1
  restartPolicy: Never
```

# 组件
## etcd
完备、高可用key-value存储，用于k8s后端存储所有的集群数据。
```sh
# 查看etcd容器启动命令
kubectl -n kube-system describe pod etcd-master01
# 查看etcd中的所有key
./etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/peer.crt --key=/etc/kubernetes/pki/etcd/peer.key get --prefix "" --keys-only
# 查看etcd中的所有key并解析为json
./etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/peer.crt --key=/etc/kubernetes/pki/etcd/peer.key get --prefix "" --keys-only -w json|python3 -m json.tool
# 查看某个key对应的字符串
base64 -d <<<L3JlZ2lzdHJ5L3NlcnZpY2VzL3NwZWNzL2t1YmVybmV0ZXMtZGFzaGJvYXJkL2t1YmVybmV0ZXMtZGFzaGJvYXJk
# etcdhelper查看key对应的值，https://github.com/openshift/origin/tree/master/tools/etcdhelper
./etcdhelper --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/peer.crt --key=/etc/kubernetes/pki/etcd/peer.key get /registry/clusterroles/system:controller:endpointslice-controller
```