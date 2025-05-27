---
title: Powershell 配置 starship
publishDate: 2025-05-20 11:21:21
description: 'Powershell 中的 starship 基础配置'
tags:
  - Powershell
language: 'Chinese'
---

## Starship

https://starship.rs/zh-CN/presets/#bracketed-segments

选择预设

### 展示完整路径

​`starship.toml`​中添加以下语句。startship.toml文件一般位于`$HOME/.config`​文件夹下

```toml
[directory]
truncate_to_repo=false
truncation_length=0
```
