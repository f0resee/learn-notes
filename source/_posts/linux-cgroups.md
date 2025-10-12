---
title: Linux cgroups
date: 2025-10-11 20:15:42
tags: Linux
---
#  cgroup
[Cgroup原理及使用](https://www.cnblogs.com/zhrx/p/16388175.html)
[Linux资源管理之cgroups简介](https://tech.meituan.com/2015/03/31/cgroups.html)
## cgroup v1和cgroup v2
怎么查看是v1还是v2：
```sh
mount | grep cgroup
```
切换：编辑`/etc/default/grub`，GRUB_CMDLINE_LINUX内将systemd.unified_cgroup_hierarchy设置为0即是cgroup v1，设置为1则是cgroup v2，编辑后需要执行
```sh
update-grub
reboot
```

## cgroup v1使用
限制PID这个进程只能使用0.5核。
```sh
mkdir /sys/fs/cgroup/cpu/test
echo 50000 > /sys/fs/cgroup/cpu/zz/cpu.cfs_quota_us
echo PID > /sys/fs/cgroup/cpu/zz/cgroup.procs
```
查看cgroup个数
```sh
cat /proc/cgroups|column -t
```

查看进程的cgroup信息
```sh
cat /proc/PID/cgroup
```


安装cgroup工具
```
apt install cgroup-tools
```

## cgroup 子系统

+ cpu: 限制cgroup的cpu使用量
+ cpuact: 统计cgroup的cpu使用情况
+ cpuset: 设置cgroup能使用的cpu核内存节点
+ memory: 设置cgroup能使用的内存
+ blkio: 限制cgroup的io使用

## 使用systemd设置cgroup
示例
```
# dd.service
[Unit]
Description=dd
ConditionFileIsExecutable=/usr/libexec/dd.sh
[Service]
Type=simple
ExecStart=/usr/libexec/dd.sh
Slice=zhrx.slice
CPUAccounting=yes
CPUQuota=40%
MemoryAccounting=yes
MemoryMax=200M
TasksAccounting=yes
BlockIOAccounting=yes
[Install]
WantedBy=multi-user.target
```


查看cgroup树形结构
```
systemd-cgls
```