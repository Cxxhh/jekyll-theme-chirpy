---
title: "C语言内存与字符串处理函数详解"
date: 2025-04-07 22:05:05 +0800
updated: 2025-04-07 22:05:05 +0800
author: Cxxhh
categories: [C语言, 学习记录]
tags: [C语言, 学习记录]
description: "本文记录了一些C语言的一些技巧"
pin: false
toc: true
comments: true
math: true
mermaid: false
media_subpath: ""
---
# C语言内存与字符串处理函数详解

## 内存操作函数 `<string.h>`

### 1. `memcpy`

**函数原型**
`void *memcpy(void *dest, const void *src, size_t n);`
**功能**
将 `src` 指向的内存区域的 **前 `n` 个字节** 复制到 `dest` 指向的内存区域。
**注意**

- 若内存重叠（`src` 和 `dest` 区域有重叠），行为未定义（应改用 `memmove`）。
- 不检查 `\0` 结束符。

**示例**

```c
char src[] = "Hello";
char dest[6];
memcpy(dest, src, 5); // dest = "Hello"
```

### 2. `memmove`

**函数原型**
`void *memmove(void *dest, const void *src, size_t n);`
**功能**
安全地复制 `n` 个字节，允许内存区域重叠。
**示例**

```c
char str[] = "ABCDEF";
memmove(str + 2, str, 3); // str 变为 "ABABCF"
```

---

### 3. `memset`

**函数原型**
`void *memset(void *s, int c, size_t n);`
**功能**
将 `s` 指向的内存区域的 **前 `n` 个字节** 设置为值 `c`。
**注意**

- 常用于初始化数组或清空内存。
- 对非字符类型（如 `int` 数组）初始化可能产生非预期值。

**示例**

```c
char buffer[10];
memset(buffer, 'A', 5); // 前5字节设为 'A'
```

---

### 4. `memcmp`

**函数原型**
`int memcmp(const void *s1, const void *s2, size_t n);`
**功能**
比较 `s1` 和 `s2` 的前 `n` 个字节。
**返回值**

- `0`: 相等
- `>0`: `s1` 的字节值更大
- `<0`: `s2` 的字节值更大

**示例**

```c
char a[] = {1, 2, 3};
char b[] = {1, 2, 4};
int result = memcmp(a, b, 3); // 返回负数（第三个字节 3 < 4）
```

---

### 5. `memchr`

**函数原型**
`void *memchr(const void *s, int c, size_t n);`
**功能**
在 `s` 的前 `n` 个字节中搜索字符 `c`。
**返回值**
找到则返回首次出现的地址，否则返回 `NULL`。
**示例**

```c
char str[] = "Hello";
char *p = memchr(str, 'e', 5); // 指向 str[1]
```

---

## 字符串处理函数 `<string.h>`

### 1. 字符串操作

#### `strcpy`

**原型**
`char *strcpy(char *dest, const char *src);`
**功能**
将 `src` 字符串（含 `\0`）复制到 `dest`。
**风险**
不检查 `dest` 缓冲区大小，可能导致溢出。

---

#### `strncpy`

**原型**
`char *strncpy(char *dest, const char *src, size_t n);`
**功能**
复制 `src` 的 **最多 `n` 个字符** 到 `dest`。
**注意**

- 若 `src` 长度 ≥ `n`，不会自动添加 `\0`。
- 常用于安全复制。

---

#### `strcat`

**原型**
`char *strcat(char *dest, const char *src);`
**功能**
将 `src` 追加到 `dest` 末尾（覆盖 `dest` 的 `\0`）。
**风险**
可能导致缓冲区溢出。

---

#### `strncat`

**原型**
`char *strncat(char *dest, const char *src, size_t n);`
**功能**
追加 `src` 的 **最多 `n` 个字符** 到 `dest`，并自动添加 `\0`。

---

### 2. 字符串比较

#### `strcmp`

**原型**
`int strcmp(const char *s1, const char *s2);`
**返回值**

- `0`: 字符串相等
- `>0`: `s1` 字典序更大
- `<0`: `s2` 字典序更大

---

#### `strncmp`

**原型**
`int strncmp(const char *s1, const char *s2, size_t n);`
**功能**
比较前 `n` 个字符。

---

### 3. 字符串查找

#### `strchr`

**原型**
`char *strchr(const char *s, int c);`
**功能**
查找字符 `c` 在 `s` 中的首次出现位置。

---

#### `strrchr`

**原型**
`char *strrchr(const char *s, int c);`
**功能**
查找字符 `c` 的最后一次出现位置。

---

#### `strstr`

**原型**
`char *strstr(const char *haystack, const char *needle);`
**功能**
在 `haystack` 中查找子串 `needle`。

---

### 4. 其他函数

#### `strlen`

**原型**
`size_t strlen(const char *s);`
**功能**
返回字符串长度（不包含 `\0`）。

---

#### `strtok`

**原型**
`char *strtok(char *str, const char *delim);`
**功能**
分割字符串为标记。
**注意**

- 首次调用传入待分割字符串，后续传入 `NULL`。
- 会修改原字符串。

**示例**

```c
char str[] = "Hello,World";
char *token = strtok(str, ","); // "Hello"
token = strtok(NULL, ",");      // "World"
```

---

#### `strerror`

**原型**
`char *strerror(int errnum);`
**功能**
返回错误码对应的描述字符串。

---

#### `strdup`

**原型**
`char *strdup(const char *s);`
**功能**
动态复制字符串（内部调用 `malloc`）。
**注意**
需手动调用 `free()` 释放内存。

---

## 总结

| 类型   | 函数      | 关键特性                      |
| ------ | --------- | ----------------------------- |
| 内存   | `memcpy`  | 高效复制，不处理重叠          |
| 内存   | `memmove` | 安全处理重叠复制              |
| 字符串 | `strncpy` | 更安全的复制（需手动加 `\0`） |
| 字符串 | `strtok`  | 分割字符串，不可重入          |

```

```