---
title: Linux 和 WSL 初始化配置
publishDate: 2025-02-19 21:00:34
description: '下载安装 WSL 并设置 zsh 和 om-my-zsh'
tags:
  - WSL
  - linux
  - zsh
language: 'Chinese'
---


## WSL

### 安装

首先确保Windows打开了以下命令

```powershell
# 启用Hyper-V
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All

# 启用虚拟机平台
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform

# 启用Linux子系统支持
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```

在Windows PowerShell下

```powershell
# 更新wsl
wsl --update
# 设置wsl2为默认版本
wsl --set-default-version 2
# 查看当前在线的wsl可用分支
wsl -l -o
# 选择你想安装的分支（这里我选择Ubuntu）
wsl --install -d Ubuntu-24.04
```

### 网络

‍

Windows中无法通过`localhost:host`​来访问WSL里的服务，可以尝试通过一下方式解决

```powershell
wsl --shutdown
netsh winsock reset
netsh int ip reset all
netsh winhttp reset proxy
ipconfig /flushdns
```

## Linux配置

下载软件前，先更新一下

```bash
sudo apt update
sudo apt-get update
```

### zsh终端及插件

安装zsh和oh-my-zsh

```bash
# 安装zsh
sudo apt install zsh
# 设置为默认sh终端
chsh -s $(which zsh)

# 安装oh-my-zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

安装p10k主题：[https://github.com/romkatv/powerlevel10k](https://github.com/romkatv/powerlevel10k)

zsh插件：

自动补全插件：[zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)

语法高亮插件：[zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)

### SSH

生成SSH KEY

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

### Node.js

使用[fnm](https://github.com/Schniz/fnm)或[pnpm](https://github.com/pnpm/pnpm)管理Node.js版本

‍
