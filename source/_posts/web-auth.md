---
title: Authentication and authorization
date: 2025-09-08 22:39:36
tags: Web 
---
# 认证（Authentication）
认证要解决的是系统如何正确的分辨出操作用户的真实身份。三种层面的认证：
1. 通信信道上的认证：在网络传输（Network）场景中的典型是基于SSL/TLS传输安全层的认证。
2. 通信协议上的认证：在互联网（Internet）场景中的典型是基于HTTP协议的认证。IETF定义的一些HTTP认证方案：
   + Basic：HTTP请求header添加base64加密后的用户名和密码
   + Digest：改进的basic（用户名和密码加盐）
   + Bearer：基于Oauth2规范来完成认证
   + HOBA：基于自签名证书的认证方案
3. 通信内容上的认证：在万维网（World Wide Web）场景中的典型是基于Web内容的认证。
   + WebAuthn
# 授权（Authorization）
授权要解决的是系统如何控制一个用户该看到哪些数据、能操作哪些功能。典型的授权方案：
1. RBAC（Role-Based Access Control）：用户（user）隶属角色（role），角色拥有许可（permission）操作资源（resource）。
2. ABAC（Attribute-Based Access Control）
3. ACL（Access Control List）
4. RBAC（Rule-Based Access Control）
5. MAC（Mandatory Access Control）
6. Oauth2：面向解决第三方应用的认证授权协议