---
title: Node.js 版本管理
publishDate: 2025-03-14 13:07:46
description: '使用 fnm 或 pnpm 管理 Node.js 版本'
tags:
  - Node.js
language: 'Chinese'
---


## 自动切换版本

### fnm
https://github.com/Schniz/fnm

在项目目录内添加`.nvmrc`​文件或`.node-version`​文件，并在文件中指定Node.js版本

```plaintext
v22.13.1
```

当cd进入该目录后，fnm会自动切换Node.js版本。但离开该目录时，不会切换会原来的版本。

### pnpm
https://pnpm.io/cli/env

在项目目录内添加`.npmrc`​文件，并在文件中指定Node.js版本

```plaintext
use-node-version=22.13.1
```

在该目录内使用pnpm命令时，会自动使用该Node.js版本，但直接调用Node.js，则还是默认的版本。

‍
