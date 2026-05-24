---
title: 记服务器重装系统
published: 2025-03-05
description: 服务器被入侵后无法登录，通过重装系统恢复访问的记录
tags: [服务器, Linux, 运维]
category: 技术
draft: false
lang: zh_CN
---

## 背景

服务器被恶意攻击后，密码被修改导致无法登录。尝试通过单用户模式修改密码失败，最后通过重装系统来恢复访问。

## 重装系统操作

### 准备

1. 格式化 U 盘，用做启动盘
2. 准备系统盘——如果用已有硬盘安装系统会自动格式化旧数据，想保留数据需要额外准备一块硬盘

### 重装系统

具体安装步骤略，各发行版安装流程大同小异。

## 重装后的配置

### SSH

开启 SSH 服务，允许远程连接。

### xrdp

配置 xrdp 以支持 Windows 远程桌面连接。

配置过程中遇到几个问题：

1. 默认的 3389 端口被网络防火墙屏蔽，需修改端口：`sudo vim /etc/xrdp/xrdp.ini`，将 `port=3389` 改为 `port=3390`，重启 xrdp `sudo systemctl restart xrdp`
2. 配置防火墙放行新端口：`sudo ufw allow 3390/tcp && sudo ufw reload`
3. 服务器上的用户需要先注销会话，否则远程连接会失败

### 桌面环境美化

安装了 gnome-tweaks 来调整远程桌面的外观：

```bash
sudo apt install gnome-tweaks
```

安装后在 GUI 的"优化"中进行自定义设置。
