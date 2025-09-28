---
title: Linux commands
date: 2025-08-01 20:08:21
tags: Linux
---
## cpu

## memory

## process
```sh
# 列出所有进程
ps -ef
ps -aux

# 查看指定进程
ps -p 3284

# 列出子进程
ps --ppid 3284 

# 查看所有父进程
pstree -p -s 3370
```

## filesystem
```shell
debugfs /dev/vda1

stat /tmp

df -i
df -h
du -sh *
```

## network
```shell
# 查看打开的端口
lsof -ni

netstat

ss
```

## atop

## containerd
```bash
# 拉镜像
ctr image pull docker.io/library/nginx:latest

# 创建容器并启动
ctr run --detach docker.io/library/nginx:latest nginx-test
ctr run --rm --shim-cgroup /my_cgroup docker.io/library/nginx:latest nginx-test

# 创建容器
ctr container create docker.io/library/nginx:latest nginx-test
ctr container create --mount=type=bind,src=/test/tmp,dst=/host/path,options=rbind:ro docker.io/library/nginx:latest nginx-test

# 查看容器信息
ctr container info nginx-test

# 删除容器
ctr container delete nginx-test

# 启动容器
ctr task start --detach nginx-test

# 容器内执行命令
ctr task exec --exec-id pwd-test nginx-test pwd
ctr task exec -t --exec-id bash-test nginx-test /bin/bash

# 
ctr task attach nginx-test

# 向容器发信号
ctr task kill -s 9 nginx-test
```


## systemd
```shell
systemctl status containerd.service
systemctl restart containerd.service
```

## journalctl
```shell
journalctl -u containerd.service
```

## mysql
```shell
apt install mariadb-server
mysqladm -u root password '123456'
```

