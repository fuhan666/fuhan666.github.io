---
title: VPS 配置
publishDate: 2025-03-05 11:20:33
description: '在 VPS 中安装 3x-ui 面板并请用 BBR'
tags:
  - VPS
  - 3x-ui
language: 'Chinese'
---

## 安装 3x-ui 面板

​[`https://github.com/MHSanaei/3x-ui`](https://github.com/MHSanaei/3x-ui)​

```bash
bash <(curl -Ls <https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh>)
```

## Ubuntu 开启 BBR

### 检查当前 BBR 状态

```bash
sysctl net.ipv4.tcp_congestion_control
```

如果返回 `net.ipv4.tcp_congestion_control = bbr`​，表示 BBR 已启用。

如果返回其它算法，比如 `cubic` ​或 `reno`​，则说明 BBR 尚未激活。

### 启用 BBR

虽然主流的 Ubuntu 版本通常都支持 BBR，但为了完整性考虑，请在「终端」中执行以下命令，检查当前 Ubuntu 系统的 BBR 兼容性

```bash
sudo modprobe tcp_bbr
```

如果系统兼容 BBR，上述命令将不会有任何输出；如果不兼容，则会返回报错信息。

修改 `/etc/sysctl.conf`​ 配置文件来启用 bbr

```bash
sudo sh -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
sudo sh -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
```

在这里，我们设置了 `fq`​（Fair Queuing，公平排队）作为默认的排队规则，并指定 `bbr` ​作为拥塞控制算法。

执行以下命令重新加载 sysctl 设置

```bash
sudo sysctl -p
```

### 验证 BBR 启用状态

```bash
sysctl net.ipv4.tcp_congestion_control
```

如果看到以下输出，表示 BBR 已经成功被设置为 Ubuntu 的默认拥塞控制算法。

​`net.ipv4.tcp_congestion_control = bbr`​

（可选）还可以使用以下命令，检查 BBR 模块是否已经加载到内核中：

```bash
lsmod | grep bbr
```

‍
