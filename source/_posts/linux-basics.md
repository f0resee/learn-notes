---
title: linux-basics
date: 2026-10-19 01:24:15
tags: [Linux]
categories: Linux
---

# 基础概念
## 负载
负载为活跃进程数，包括running进程、就绪态进程和不可中断睡眠态进程。
## 僵尸进程
子进程退出，父进程没有通过wait()/waitpid()等待子进程结束，父进程自身又未退出，造成子进程仍然占用进程表的一个位置，称为僵尸进程。
## cpu调度策略、优先级