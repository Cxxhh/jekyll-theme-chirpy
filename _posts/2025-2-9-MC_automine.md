---
title: "自动采矿脚本"
date: 2024-02-09 12:00:00 +0800
updated: 2024-02-09 12:00:00 +0800
author: Cxxhh
categories: [Minecraft, 自动采矿]
tags: [Baritone, JsMacros, 自动采矿脚本]
description: "该脚本是为 Minecraft 设计的自动采矿辅助工具，利用 Baritone 自动化挖矿，并配合 JsMacros 实现自动化操作。"
image:
  path: https://testingcf.jsdelivr.net/gh/Cxxhh/blog-img/img/mine.jpg
pin: false
toc: true
comments: true
math: false
mermaid: false
media_subpath: ""
---

# 自动采矿脚本介绍

该脚本是为 Minecraft 游戏设计的自动采矿辅助工具，利用 **Baritone** 自动化挖矿，并配合 **JsMacros** 实现一系列自动化操作。

脚本主要完成以下功能：

- **矿物数据管理**  
  支持管理多种矿物（包括普通变种和深板岩变种），并通过 `/jsm automine add` 命令添加追踪目标矿物。

- **自动采矿**  
  启动后，脚本会通过 Baritone 指令自动挖掘目标矿物，并在采矿过程中自动处理背包满的情况。

- **背包监控与卸货**  
  定时检测背包状态，若检测到满状态，则自动：
  - 停止采矿
  - 寻找附近箱子并尝试打开
  - 快速转移非目标物品到箱子中
  - 返回采矿点继续采矿

- **命令注册**  
  提供以下子命令：
  - `/jsm automine add <矿物1> <矿物2> ...`：添加目标矿物
  - `/jsm automine list`：列出当前追踪的矿物
  - `/jsm automine start`：启动自动采矿
  - `/jsm automine stop`：停止自动采矿
  - `/jsm automine clear`：清空所有追踪矿物
  - `/jsm automine help`：显示使用帮助

- **快捷键退出**  
  监听键盘 `X` 键，按下时立即停止脚本运行。

---

## 使用方法

### 1. 设置卸货点

在启动自动采矿前，请先使用以下命令设置卸货点（注意：附近需要有箱子）：  

```bash
/sethome storage
```

### 2. 添加目标矿物

在采矿前，通过命令添加需要采集的矿物，例如：

```bash
/jsm automine add 钻石 绿宝石 煤
```

### 3. 启动自动采矿

站在挖矿点后，使用以下命令启动自动采矿：

```bash
/jsm automine start
```

### 4. 查看当前追踪的矿物

使用以下命令查看已添加的矿物：

```bash
/jsm automine list
```

### 5. 停止自动采矿

若需停止采矿，可使用以下命令：

```bash
/jsm automine stop
```

### 6. 清空所有追踪矿物

清空目标矿物列表，使用命令：

```bash
/jsm automine clear
```

### 7. 退出脚本

在任何时候按下键盘的 `X` 键，即可退出脚本运行。

---

## 作者信息

- **作者**：Pech  
- **QQ**：3054736043  
---

## 更新日志

### 2024-02-09 更新 V1.0.2  
- 修复卡顿问题  
- 去除 debug 的错误日志输出  

---

**以上就是自动采矿脚本的详细介绍与使用说明。如果在使用过程中遇到问题或有改进建议，请及时与作者联系。祝您游戏愉快！**
