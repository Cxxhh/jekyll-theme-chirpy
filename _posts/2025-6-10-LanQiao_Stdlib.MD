---
title: "C 标准库字符串与内存操作函数及格式说明符汇总"
date: 2025-06-10 11:43:05 +0800
updated: 2025-06-10 11:43:05 +0800
author: Cxxhh
categories: [C语言, 嵌入式,学习记录]
tags: [C语言,标准库]
description: "本文汇总了 C 标准库中常用的字符串/内存操作函数与格式化输入输出函数，以及常见的格式说明符，用于嵌入式 C 语言学习记录"
pin: false
toc: true
comments: true
math: false
mermaid: false
media_subpath: ""
---


## 1. 字符串操作函数 `<string.h>`

### 1.1 原型与说明

| 函数原型                                                 | 功能说明                                 | 备注                   |
| -------------------------------------------------------- | ---------------------------------------- | ---------------------- |
| `char *strcpy(char *dest, const char *src)`              | 复制 `src` 到 `dest`（含 `\0`）          | 目标区必须足够大       |
| `char *strncpy(char *dest, const char *src, size_t n)`   | 最多复制 `n` 个字符                      | 不足 `n` 时自动补 `\0` |
| `char *strcat(char *dest, const char *src)`              | 将 `src` 追加到 `dest` 末尾（覆盖 `\0`） | 目标区必须足够大       |
| `char *strncat(char *dest, const char *src, size_t n)`   | 最多追加 `n` 个字符                      | 末尾自动加 `\0`        |
| `int strcmp(const char *s1, const char *s2)`             | 按字典序比较字符串                       | 返回 0/正/负           |
| `int strncmp(const char *s1, const char *s2, size_t n)`  | 最多比较前 `n` 个字符                    | -                      |
| `size_t strlen(const char *s)`                           | 返回长度（不含 `\0`）                    | -                      |
| `char *strchr(const char *s, int c)`                     | 查找首次出现的字符 `c`                   | 返回指针或 `NULL`      |
| `char *strrchr(const char *s, int c)`                    | 查找最后一次出现的字符 `c`               | 返回指针或 `NULL`      |
| `char *strstr(const char *haystack, const char *needle)` | 查找子串 `needle`                        | 返回指针或 `NULL`      |
| `char *strtok(char *str, const char *delim)`             | 分割字符串，                             |                        |

|     |
| --- |

| **不可重入** | 会修改原字符串 |
| ------------ | -------------- |

### 1.2 使用示例

```c
char buf[100];
strcpy(buf, "Hello");          // buf -> "Hello"
strcat(buf, ", world!");      // buf -> "Hello, world!"
size_t len = strlen(buf);        // len = 13
int cmp  = strcmp("abc", "abd"); // cmp < 0
```

---

## 2. 内存操作函数 `<string.h>`

| 函数原型                                               | 功能说明                | 备注               |
| ------------------------------------------------------ | ----------------------- | ------------------ |
| `void *memcpy(void *dest, const void *src, size_t n)`  | 复制 `n` 字节           | **不**处理重叠区域 |
| `void *memmove(void *dest, const void *src, size_t n)` | 复制 `n` 字节           | 可安全处理重叠     |
| `void *memset(void *s, int c, size_t n)`               | 将前 `n` 字节填充为 `c` | 常用于初始化       |
| `int memcmp(const void *s1, const void *s2, size_t n)` | 比较前 `n` 字节         | 返回 0/正/负       |

### 示例

```c
uint8_t buf[16];
memset(buf, 0xFF, sizeof(buf));  // buf 全部设为 0xFF
uint8_t src[4] = {1,2,3,4};
memcpy(buf + 4, src, 4);         // 将 1,2,3,4 写到 buf[4..7]
int diff = memcmp(buf, src, 4);  // 比较前 4 字节
```

---

## 3. 格式化输入/输出函数 `<stdio.h>`

| 函数原型                                                         | 功能说明                     |
| ---------------------------------------------------------------- | ---------------------------- |
| `int sprintf(char *str, const char *format, ...);`               | 格式化到字符串（**易溢出**） |
| `int snprintf(char *str, size_t size, const char *format, ...);` | 带长度限制，更安全           |
| `int sscanf(const char *str, const char *format, ...);`          | 从字符串解析数据             |

### 示例

```c
char buf[50];
int a = 5, b = 10;
sprintf(buf, "a=%d, b=%02d", a, b);  // "a=5, b=10"

// sscanf 解析
double x;
sscanf("3.14,100", "%lf,%d", &x, &a); // x = 3.14, a = 100
```

---

## 4. 常见格式说明符

### 4.1 占位符结构

```text
%[flags][width][.precision][length]specifier
```

### 4.2 转换说明符（specifier）

| 说明符   | 类型                  | 示例        |
| -------- | --------------------- | ----------- |
| `d`, `i` | 有符号十进制          | `%d`        |
| `u`      | 无符号十进制          | `%u`        |
| `o`      | 无符号八进制          | `%o`        |
| `x`, `X` | 无符号十六进制        | `%x` / `%X` |
| `f`, `F` | 浮点数（十进制）      | `%.2f`      |
| `e`, `E` | 浮点数（指数）        | `%.3e`      |
| `g`, `G` | 自动 `%f` / `%e` 切换 | `%g`        |
| `c`      | 单字符                | `%c`        |
| `s`      | 字符串                | `%s`        |
| `p`      | 指针地址              | `%p`        |
| `n`      | 写入已输出字符数      | `%n`        |
| `%`      | 输出 `%`              | `%%`        |

### 4.3 常用标志（flags）

| 标志 | 含义                                    |
| ---- | --------------------------------------- |
| `-`  | 左对齐                                  |
| `+`  | 强制符号位                              |
| 空格 | 正数前输出空格                          |
| `0`  | 用 0 填充空位                           |
| `#`  | `o`, `x`, `X`, `f`, `e`, `g` 有特殊含义 |

### 4.4 宽度与精度

* **宽度 (**\`\`**)**：最小字段宽度，例如 `%5d` 右对齐占 5 位。
* **精度 (**\`\`**)**：

  * 对整数：最小数字位数，如 `%02d`。
  * 对浮点：小数位数，如 `%.3f`。
  * 对字符串：最大输出字符数，如 `%.5s`。

### 4.5 输出示例

```c
printf("|%5d|", 42);        // |   42|
printf("|%-5d|", 42);       // |42   |
printf("|%05d|", 42);       // |00042|
printf("|%02d|", 9);        // |09|
printf("|%.3f|", 3.14);     // |3.140|
printf("|%.5s|", "Hello, world"); // |Hello|
```

---

## 5. 速查表

| 分类       | 关键函数   | 特性 / 关键点          |
| ---------- | ---------- | ---------------------- |
| **内存**   | `memcpy`   | 高效复制，**不**可重叠 |
|            | `memmove`  | 可安全处理重叠         |
| **字符串** | `strncpy`  | 限长复制，需确保终止符 |
|            | `strtok`   | 字符串分割，*不可重入* |
| **格式化** | `snprintf` | 带长度限制，推荐使用   |

---

> 💡 **提示**：以上内容仅作学习记录，实际开发请结合平台文档与编译器手册查阅最新标准。
