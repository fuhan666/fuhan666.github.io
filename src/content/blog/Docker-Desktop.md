---
title: Docker Desktop 问题解决
publishDate: 2025-06-13 11:10:00
description: 'WSL2 Docker Desktop 发行版部署失败问题'
tags:
  - WSL
  - linux
  - DockerDocker
language: 'Chinese'
---


## WSL2 Docker Desktop 发行版部署失败问题

### 一、问题描述

在启动或安装 Docker Desktop 时，WSL2 尝试部署其核心的 `docker-desktop`​ 发行版时，程序意外中断并显示以下错误信息：

```shel
deploying WSL2 distributions
ensuring main distro is deployed: deploying "docker-desktop": importing WSL distro "An install, uninstall, or conversion is in progress for this distribution.
Error code: Wsl/Service/RegisterDistro/0x8000000d
" output="docker-desktop": exit code: 4294967295...
```

核心错误信息：`An install, uninstall, or conversion is in progress for this distribution.`​

错误代码：`Wsl/Service/RegisterDistro/0x8000000d`​

### 二、问题分析

这个错误明确指出，WSL 子系统认为 `docker-desktop`​ 这个发行版**正处于一个未完成的“安装、卸载或转换”状态**。这通常是因为之前的安装或更新过程被意外中断，导致发行版的状态被“锁定”或损坏。

### 三、解决过程回顾

#### 方案A：重置 WSL 用户配置文件（尝试过但未直接解决）

> https://stackoverflow.com/questions/71089617/docker-not-starting-on-windows-11-with-wsl-2

根据 Stack Overflow 上的建议，我首先尝试了删除 WSL 的用户设置文件，以期重置可能已损坏的配置。

1. **操作**：删除 `settings-store.json`​ 文件。
2. **文件路径**：该文件位于 `C:\Users\%UserName%\AppData\Roaming\Docker\settings-store.json`​ ，主要存储 WSL 的全局用户配置。
3. **目的**：强制 WSL 在下次启动时生成一份全新的默认配置。
4. **结果**：**此操作单独执行后，并未解决** **​`0x8000000d`​**​ **的核心错误。**  问题根源在于单个发行版的状态损坏，而非全局配置。

#### 方案B：强制注销损坏的发行版（最终解决方法）

在尝试方案A无效后，采取了以下方法。

##### 步骤 1：以管理员身份打开终端

右键点击“开始”菜单，选择“**PowerShell (管理员)** ”或“**命令提示符 (管理员)** ”。

##### 步骤 2：强制注销卡住的发行版

执行以下核心命令，强制将 `docker-desktop`​ 发行版从 WSL 中移除。

```powershell
wsl --unregister docker-desktop
```

**注意**：此命令会删除 Docker Desktop 在 WSL 中的数据，但 Docker Desktop 会在下次启动时自动重建，因此是安全的。

##### 步骤 3：重启电脑并重启 Docker Desktop
