---
title: Upgrade Debian10 to Debian11
date: 2022-06-27 22:14:04
tags:
---
## 简单记录一下从 `debian 10` 升级到 `debian 11` 的过程

> 以下操作均是切换到 root 身份下进行的

### 1. 更新现有软件
- 更新现有软件和安全补丁到最新版本
> apt update && apt upgrade -y
- 卸载不需要的依赖项
> apt --purge autoremove

### 2. 更新 source.list 文件
这里使用清华的镜像源

````
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free

deb https://mirrors.tuna.tsinghua.edu.cn/debian-security bullseye-security main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security bullseye-security main contrib non-free
````

### 3. 升级到 Debian 11
> apt update && apt full-upgrade

##### 可以参考这篇[文章](https://www.zatp.com/post/upgrade-to-debian-11/)
