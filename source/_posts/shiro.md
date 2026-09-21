---
title: shiro
date: 2026-09-21 14:12:17
tags:
  - web
  - shiro
categories:
  - web
  - shiro
---

# Shiro

## 0x01 ~ Shiro的简单介绍

- shiro 是 apache 提供的一个开源 java 安全框架

- shiro 能够实现身份验证、授权、加密和会话管理等功能

![image-20260921143359241](./../images/image-20260921143359241.png)

> Primary Concerns:
>
> - Anthentication: 身份认证，验证用户是否有相应的身份-登录认证
> - Authorization: 权限验证，验证某个已认证的用户是否拥有某个权限
> - Session Management: 会话管理，管理用户登录后的会话
> - Cryptography: 加密，使用密码学加密数据。如，加密密码。

> Supporting Features:
>
> - Web Support: Web支持，能够比较轻易地整合到Web环境中
>
> - Caching: 缓存，对用户的数据进行缓存
> - Concurrency: 并发，支持具有并发功能的多线程应用程序
> - Testing: 测试，提供测试的支持
> - Run As: 允许用户以其他用户的身份来登录/访问
> - Remember Me**（漏洞点之一）**: 让用户在关闭浏览器或会话过期后，下次访问系统时仍然保持登录状态，无需重新输入用户名和密码。



## 0x02 ~ Shiro 550（CVE-2016-4437）

### 漏洞简介

Shiro 550，即**CVE-2016-4437**，是Apache Shiro框架中一个因**硬编码AES密钥**和**反序列化无校验**导致的高危远程代码执行漏洞。



### 漏洞原理

550的漏洞源于 **Remember Me** 特性（可见0x01中的Remember Me功能的介绍）的实现方式：

> 1、服务端加密：用户登录后，**Shiro** 会将用户身份信息序列化，使用 **AES 加密**后再进行 **Base64 编码**，最后存入名为 `rememberMe` 的 Cookie 中返回给浏览器。
>
> 2、客户端请求：下次访问时，浏览器携带该 Cookie，服务端会进行 **Base64 解码 → AES 解密 → Java 反序列化**，以恢复用户身份。



### 影响版本

**shiro-550** 漏洞影响 **Apache Shiro <= 1.2.4** 的所有版本。

> 在 Shiro 1.2.4 及之前的版本中，用于 AES 加密的密钥是硬编码在源码中的（默认为 `kPH+bIxk5D2deZiIxcaaaA==`）。



### 环境搭建

环境搭建部分看的这个blog的搭建过程：https://www.v0n.top/2022/03/09/Shiro%E5%AE%89%E5%85%A8%E5%AD%A6%E4%B9%A0/



### 过程分析

