---
title: 记一次服务器中挖矿病毒
published: 2024-10-30
description: 服务器被植入挖矿病毒的发现、排查和清理全过程记录
tags: [服务器, Linux, 安全, 运维]
category: 技术
draft: false
lang: zh_CN
---

## 发现异常

某天发现服务器上有进程异常占用 GPU 资源，但一开始以为是其他用户在正常使用，没有在意。

```
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A      1710      G   /usr/lib/xorg/Xorg                  4MiB |
|    0   N/A  N/A     24653      C   python                            139MiB |
|    0   N/A  N/A     24654      C   python                            139MiB |
|    0   N/A  N/A     24655      C   python                            139MiB |
|    1   N/A  N/A      1710      G   /usr/lib/xorg/Xorg                  4MiB |
|    1   N/A  N/A     24653      C   python                            149MiB |
|    1   N/A  N/A     24654      C   python                            149MiB |
|    1   N/A  N/A     24655      C   python                            149MiB |
|    2   N/A  N/A      1710      G   /usr/lib/xorg/Xorg                  4MiB |
|    2   N/A  N/A     24653      C   python                            149MiB |
|    2   N/A  N/A     24654      C   python                            149MiB |
|    2   N/A  N/A     24655      C   python                            149MiB |
|    3   N/A  N/A      1710      G   /usr/lib/xorg/Xorg                  4MiB |
|    3   N/A  N/A     24653      C   python                            139MiB |
|    3   N/A  N/A     24654      C   python                            139MiB |
|    3   N/A  N/A     24655      C   python                            139MiB |
+-----------------------------------------------------------------------------+
```

10 月 29 日晚，需要训练模型，发现有进程占用 GPU 且进程频繁结束和重启（约 5 分钟一个周期），便在一个空挡开始训练。

10 月 30 日早，觉得进程过于异常，开始排查。

## 确认为挖矿病毒

通过 `ps` 查看进程树，发现可疑的父进程：

```
$ ps -f -p 24655
UID        PID  PPID  C STIME TTY          TIME CMD
user     24655 24652  3 14:22 ?        00:00:11 python
$ ps -f -p 24652
UID        PID  PPID  C STIME TTY          TIME CMD
user     24652 24647  0 14:22 ?        00:00:00 python
$ ps -f -p 24647
UID        PID  PPID  C STIME TTY          TIME CMD
user     24647     1  0 14:22 ?        00:00:00 ./h64 -s python -p bash.pid ./javra -a kawpow -o str
```

`./h64` 和 `./javra` 两个程序明显是挖矿工具，`-a kawpow` 指定了 KawPow 算法（一种 GPU 挖矿算法）。

进一步查看进程详情确认了矿池地址和钱包地址：

```
./h64 -s python -p bash.pid ./javra -a kawpow \
  -o stratum+ssl://pool.example.com:12433 \
  -u [REDACTED_WALLET_ADDRESS] \
  -d 0,1,2,3 --no-watchdog --no-strict-ssl
```

基本确定这是一个挖矿病毒。询问相关人员得知近期无人使用该账户，基本确认为恶意行为。该进程最早在 10 月 21–27 日间就已被发现。

结合时间推测，可能与 10 月 18 日学校网络服务遭受攻击有关。

## 清理过程

### 第一次尝试：直接杀进程

```bash
sudo kill -9 3131 3132 3133
```

但进程很快又重新启动了，说明有守护机制。

### 查找定时任务

根据网上的排查指南逐步检查：

1. 查找定时任务：`sudo crontab -l`（root 用户，无任务）
2. 查找开机自启：`sudo cat /etc/rc.local`（文件不存在）
3. 查看进程归属：`systemctl status <PID>`

```
$ systemctl status 16714
● cron.service - Regular background program processing daemon
   Loaded: loaded (/lib/systemd/system/cron.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2024-10-10 10:02:46 CST; 2 weeks 6 days ago
     Docs: man:cron(8)
 Main PID: 1536 (cron)
    Tasks: 82 (limit: 6143)
   CGroup: /system.slice/cron.service
           ├─ 1536 /usr/sbin/cron -f
           ├─16714 ./h64 -s python ...
           ├─16715 ./h64 -s python ...
           ├─16716 ./h64 -s python ...
           ├─16720 python
           ├─16721 python
           ├─16722 python
           ├─16723 python
           ├─16724 python
           └─16725 python
```

挖矿进程挂载在 `cron.service` 之下，说明是通过 cron 定时任务启动的。

### 找到真正的定时任务

一开始用 `sudo crontab -l` 查看的是 root 用户的定时任务，显示为空。病毒实际设置在另一个用户账户下：

```bash
$ sudo crontab -u user -l
* * * * * /var/tmp/.cache/upd >/dev/null 2>&1
* * * * * /tmp/.cache/go.sh
* * * * * /tmp/.cache/go.sh
* * * * * /tmp/.cache/go.sh
```

三条 `go.sh` 每分钟执行一次，输出被重定向到 `/dev/null` 隐藏痕迹。

### 查看病毒文件

```bash
$ ls -la /tmp/.cache/
总用量 45608
-rwxr-xr-x  1 user  user       170 10月 17 05:23 a.sh
-rw-rw-r--  1 user  user         5 10月 30 15:49 bash.pid
-rwxr-xr-x  1 user  user        37 10月 17 05:22 go.sh
-rwxr-xr-x  1 user  user     26383 3月   2  2024 h64
-rwxr-xr-x  1 user  user  46498633 4月  25  2024 javra
-rwxr-xr-x  1 user  user      2968 4月  25  2024 m
```

`h64` 是挖矿主程序，`javra` 也挖矿相关工具。`go.sh` 是启动脚本。

```bash
$ ls -la /var/tmp/.cache/
总用量 3244
-rwxr-xr-x  1 user  user      329 11月 14  2023 a
-rw-rw-r--  1 user  user        6 10月 15 23:36 bash.pid
-rw-rw-r--  1 user  user       46 10月 15 23:36 cron.d
-rw-rw-r--  1 user  user       16 10月 15 23:36 dir.dir
-rwxr-xr-x  1 user  user    15125 3月  10  2015 h32
-rwxr-xr-x  1 user  user   838583 2月  21  2016 h64
-rwxr-xr-x  1 user  user  2426868 8月  11 21:35 java
-rwxr-xr-x  1 user  user      440 11月 15  2023 run
-rwxr-xr-x  1 user  user      194 10月 15 23:36 upd
-rwxr-xr-x  1 user  user       24 10月 15 23:36 x
```

`/var/tmp/.cache/` 下是一套更完整的工具链，包括不同架构的挖矿程序（`h32`、`h64`）和辅助脚本。

### 最终的清理操作

```bash
# 定位并杀死进程
sudo ls -l /proc/18900/exe   # 确认进程对应的文件
# lrwxrwxrwx 1 user user 0 ... /proc/18900/exe -> /var/tmp/.cache/java
sudo kill -9 18900

# 删除该用户的定时任务
sudo crontab -e -u user
# 删除所有可疑任务后保存

# 删除病毒文件
sudo rm -rf /var/tmp/.cache/
sudo rm -rf /tmp/.cache/

# 确认进程已清理
ps aux | grep h64
```

## 总结

这次排查有两个值得记下的点：

1. **GPU 资源被占用不是唯一指标**。挖矿病毒也会大量消耗 CPU 资源，通过 `top` 观察 CPU 占用可以更早发现异常。这次只盯着 GPU 看，错过了早期发现的机会。
2. **查看定时任务时要指定用户**。`sudo crontab -l` 只查看 root 用户的定时任务，而病毒可能设置在普通用户账户下，需要用 `sudo crontab -l -u <用户名>` 查看。

## 快速排查流程

1. `cd /tmp/.cache` 检查可疑目录
2. `ll` 查看目录由哪个用户创建
3. `sudo crontab -l -u <用户名>` 确认定时任务
4. `sudo crontab -e -u <用户名>` 清除定时任务
5. `top` 查找可疑进程 → `ls -l /proc/<PID>/exe` 确认 → `kill -9 <PID>` 终止
6. `sudo rm -rf /tmp/.cache /var/tmp/.cache` 删除病毒文件
7. 确认进程不再重启
