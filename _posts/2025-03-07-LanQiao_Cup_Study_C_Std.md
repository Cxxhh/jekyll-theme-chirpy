---
title: "蓝桥杯嵌入式外设开发记录 - C 标准库函数"
date: 2025-06-14 11:43:05 +0800
updated: 2025-06-14 11:43:05 +0800
author: Cxxhh
categories: [蓝桥杯, 嵌入式,学习记录，C语言]
tags: [STM32, Keil, CubeMX, 蓝桥杯, 嵌入式,学习记录，C语言]
description: "本文记录了学习蓝桥杯嵌入式开发过程中的C语言标准库函数的记录"
pin: false
toc: true
comments: true
math: true
mermaid: false
media_subpath: ""
---
# C 标准库字符串、内存与字符转换函数大全

> **说明**：本文整合了常用的 **字符串操作函数**、**内存操作函数**、**格式化输入/输出**、**`sscanf`\**\**\**\**\**\**\**\**\**\**\**\** 高级用法**、**字符 ↔ 数字手动转换技巧**，并 **新增** `ctype.h` 与 `stdlib.h` 中「字符/字符串转换函数」的完整速查表，方便嵌入式或系统级 C 开发快速查阅。

------

## 1 字符串操作函数（`<string.h>`）

| 函数原型                                                 | 作用                                 | 典型注意事项                         |
| -------------------------------------------------------- | ------------------------------------ | ------------------------------------ |
| `char *strcpy(char *dest, const char *src)`              | 复制 `src`（含终止符 `\0`）到 `dest` | `dest` 必须足够大，风险：缓冲区溢出  |
| `char *strncpy(char *dest, const char *src, size_t n)`   | 最多复制 `n` 字符                    | 若 `src` 长度 ≥ `n`，不会自动补 `\0` |
| `char *strcat(char *dest, const char *src)`              | 把 `src` 追加到 `dest` 尾部          | 同样需保证空间                       |
| `char *strncat(char *dest, const char *src, size_t n)`   | 最多追加 `n` 字符                    | 自动补 `\0`                          |
| `int strcmp(const char *s1, const char *s2)`             | 按字典序比较                         | 返回 0 相等、>0 或 <0                |
| `int strncmp(const char *s1, const char *s2, size_t n)`  | 前 `n` 字符比较                      | —                                    |
| `size_t strlen(const char *s)`                           | 求长度（不含 `\0`）                  | —                                    |
| `char *strchr(const char *s, int c)` / `strrchr`         | 查找字符首次/末次出现                | 返回指针或 `NULL`                    |
| `char *strstr(const char *haystack, const char *needle)` | 查找子串                             | —                                    |
| `char *strtok(char *str, const char *delim)`             | 分割字符串为标记                     | **会修改原串；不可重入**             |
| `size_t strspn / strcspn`                                | 计算前缀长                           | —                                    |
| `char *strpbrk`                                          | 查找集合中任意字符                   | —                                    |
| `char *strdup` (POSIX)                                   | 克隆字符串（内部 `malloc`）          | 用完需 `free()`                      |
| `char *strerror`                                         | 错误码转描述                         | —                                    |

> **示例**

```c
char buf[64] = "Hello";
strcat(buf, ", World!");   // "Hello, World!"
size_t len = strlen(buf);    // 13
```

------

## 2 内存操作函数（`<string.h>`）

| 函数原型                                               | 作用                  | 特点             |
| ------------------------------------------------------ | --------------------- | ---------------- |
| `void *memcpy(void *dest, const void *src, size_t n)`  | 复制 `n` 字节         | **不**支持重叠   |
| `void *memmove(void *dest, const void *src, size_t n)` | 复制（可重叠）        | 内部判断方向     |
| `void *memset(void *s, int c, size_t n)`               | 内存置值              | 常用于初始化     |
| `int memcmp(const void *s1, const void *s2, size_t n)` | 比较                  | 返回 0 / >0 / <0 |
| `void *memchr(const void *s, int c, size_t n)`         | 在前 `n` 字节查找字符 | 找到返回指针     |

**安全提示**：大块结构体复制建议优先 `memcpy`；若区间可能重叠务必改用 `memmove`。

------

## 3 字符分类与转换函数

### 3.1 `<ctype.h>` 判断类函数（全部接收 `int`，实参须可转换为 `unsigned char` 或 `EOF`）

| 函数              | 为真条件                          | 典型用途           |
| ----------------- | --------------------------------- | ------------------ |
| `isalnum(int c)`  | 字母或数字                        | 解析标识符         |
| `isalpha(int c)`  | 字母                              | —                  |
| `isblank(int c)`  | 空格或水平制表                    | 格式化解析         |
| `iscntrl(int c)`  | 控制字符 (`0x00~0x1F`, `0x7F`)    | 协议帧过滤         |
| `isdigit(int c)`  | 数字 0–9                          | 字符串转数字前检查 |
| `isgraph(int c)`  | 可打印且非空格                    | UI 过滤            |
| `islower(int c)`  | 小写字母                          | 字母统计           |
| `isupper(int c)`  | 大写字母                          | —                  |
| `isprint(int c)`  | 可打印字符（含空格）              | 日志输出防乱码     |
| `ispunct(int c)`  | 标点符号                          | 语法高亮           |
| `isspace(int c)`  | 任意空白：空格、`\t`、`\n`、`\r`… | CSV 解析           |
| `isxdigit(int c)` | 十六进制数字                      | 解析 HEX 字符串    |

### 3.2 `<ctype.h>` 转换类函数

| 函数                 | 说明                          | 例子              |
| -------------------- | ----------------------------- | ----------------- |
| `int tolower(int c)` | 大写 → 小写；若非大写原样返回 | `c = tolower(c);` |
| `int toupper(int c)` | 小写 → 大写；若非小写原样返回 | `c = toupper(c);` |

> **实现要点**：不要自己写 `c | 0x20` 等位运算，`tolower` / `toupper` 会正确处理本地字符集及 EOF。

### 3.3 `<stdlib.h>` 字符串 ↔ 数值转换速查

| 函数                                               | 功能                     | 失败返回                      |
| -------------------------------------------------- | ------------------------ | ----------------------------- |
| `int atoi(const char *s)`                          | C 字符串 → `int`         | `0`（无法区分错误）           |
| `long atol(const char *s)` / `long long atoll`     | → `long` / `long long`   | 同上                          |
| `double atof(const char *s)`                       | → `double`               | 0.0                           |
| `long strtol(const char *s, char **end, int base)` | 进制可选（2~36，0=自动） | `LONG_MAX/MIN` 并设置 `errno` |
| `unsigned long strtoul(...)` / `strtoull`          | 无符号                   | `ULONG_MAX`                   |
| `double strtod(const char *s, char **end)`         | 科学计数支持             | `HUGE_VAL`/`-HUGE_VAL`        |

> `strto*` 函数族通过 `*end` 指针返回未解析部分，更适合可靠解析；转换前可配合 `isspace` / `isdigit` 进行输入校验。

------

## 4 字符 ↔ 数字手动转换小技巧

> 直接位移运算速度快、无额外依赖，适合裸机或性能关键路径。

| 场景               | 快捷表达式               | 说明                 |
| ------------------ | ------------------------ | -------------------- |
| `'0'~'9'` → `0~9`  | `num = ch - '0';`        | `'0'` 的 ASCII 为 48 |
| `0~9` → `'0'~'9'`  | `ch = num + '0';`        | —                    |
| `'A'~'Z'` ↔ `0~25` | `ch - 'A'` / `num + 'A'` | 大写字母编号         |
| `'a'~'z'` ↔ `0~25` | `ch - 'a'` / `num + 'a'` | 小写字母编号         |
| 大小写互换         | `ch ^ 32`                | 仅限 ASCII           |

**十六进制字符 → 数值**

```c
if (ch >= '0' && ch <= '9')      val = ch - '0';
else if (ch >= 'A' && ch <= 'F') val = ch - 'A' + 10;
else if (ch >= 'a' && ch <= 'f') val = ch - 'a' + 10;
```

------

## 5 格式化输入 / 输出函数（`<stdio.h>`）

| 函数                                                       | 说明           | 关键点                                  |
| ---------------------------------------------------------- | -------------- | --------------------------------------- |
| `int sprintf(char *str, const char *fmt, …)`               | 格式化到缓冲区 | **无长度限制，慎用**                    |
| `int snprintf(char *str, size_t size, const char *fmt, …)` | 限定最大字节数 | 生产环境首选                            |
| `int sscanf(const char *str, const char *fmt, …)`          | 从字符串解析   | 返回成功匹配数                          |
| `printf` / `fprintf` / `snprintf`                          | —              | `%m`（GNU）可直接打印 `strerror(errno)` |

### 常见格式说明符

| 说明符                    | 典型含义            | 示例           |
| ------------------------- | ------------------- | -------------- |
| `%d` / `%i`               | 有符号十进制        | `%d`           |
| `%u` / `%o` / `%x` / `%X` | 无符号 10/8/16 进制 | `%08X`         |
| `%f` / `%e` / `%g`        | 浮点输出            | `%.2f`         |
| `%s` / `%c`               | 字符串 / 字符       | —              |
| `%p`                      | 指针地址            | —              |
| `%n`                      | 写入已输出字符数    | 安全审计需关注 |
| `%%`                      | 输出 `%`            | —              |

> **Flags**：`-` 左对齐、`+` 强制符号、`0` 前导零、`#` 补前缀（如 `0x`）、空格保留位。

------

## 6 `sscanf` 高级用法精粹

1. **字段宽度限制**：`%4s` 避免溢出。
2. **忽略字段**：`%*s`/`%*d` 跳过不需要的数据。
3. **字符集合**：`%[^:]` 读取直到冒号；`%[A-Za-z0-9]` 只收集匹配集合。
4. **获取读取偏移**：`%n` 把当前已解析字符数写入整型。
5. **循环解析**：结合 `offset + bytes_read` 多次调用实现按分隔符拆分列表。

```c
char line[] = "id=123 name=Bob age=30";
int id, age; char name[16];
if (sscanf(line, "id=%d name=%15s age=%d", &id, name, &age) == 3) {
    /* 解析成功 */
}
```

------

## 7 数组插入 / 删除示例（`memmove` 实战）

```c
void insert(int *arr, int *size, int index, int value) {
    if (index < 0 || index > *size) return;
    memmove(&arr[index + 1], &arr[index], (*size - index) * sizeof(int));
    arr[index] = value;
    ++(*size);
}

void delete(int *arr, int *size, int index) {
    if (index < 0 || index >= *size) return;
    memmove(&arr[index], &arr[index + 1], (*size - index - 1) * sizeof(int));
    --(*size);
}
```

------
## 8 静态二维数组的插入与删除

> **适用场景**：在嵌入式环境或竞赛题中，往往只能用 *定长* 数组（如 `int a[MAX_R][MAX_C];`）来维护表格数据。由于容量固定，"插入/删除" 行或列只能通过**整体移动元素**来实现。下面给出通用思路与参考实现。

### 8.1 插入行

```c
#define MAX_R 100
#define MAX_C 50

// rows: 当前已用行数（按引用传入）
// pos : [0, rows] 之间，表示新行插入到第 pos 行之前
void insert_row(int a[][MAX_C], int *rows, const int new_row[MAX_C], int pos)
{
    if (*rows >= MAX_R || pos < 0 || pos > *rows) return; // 越界保护
    for (int r = *rows; r > pos; --r) {
        memcpy(a[r], a[r-1], sizeof(int) * MAX_C); // 整行后移
    }
    memcpy(a[pos], new_row, sizeof(int) * MAX_C);
    ++(*rows);
}
```

- **复杂度**：`O((rows-pos) * MAX_C)`
- **风险**：需确保 `rows < MAX_R`，否则溢出。

### 8.2 删除行

```c
void delete_row(int a[][MAX_C], int *rows, int pos)
{
    if (*rows == 0 || pos < 0 || pos >= *rows) return;
    for (int r = pos; r < *rows - 1; ++r) {
        memcpy(a[r], a[r+1], sizeof(int) * MAX_C); // 整行前移
    }
    --(*rows);
}
```

### 8.3 插入 / 删除列

原理与行类似，只是最内层循环换成列索引：

```c
void insert_col(int a[][MAX_C], int rows, int *cols, int pos, int val)
{
    if (*cols >= MAX_C || pos < 0 || pos > *cols) return;
    for (int r = 0; r < rows; ++r) {
        for (int c = *cols; c > pos; --c) {
            a[r][c] = a[r][c-1];
        }
        a[r][pos] = val;
    }
    ++(*cols);
}
```

- 列删除同理，将 `>` 与 `<` 方向颠倒即可。

> **技巧**：
>
> 1. 尽量 **批量移动**（如 `memcpy`）而非逐元素拷贝，可提速数倍。
> 2. 对于稀疏修改，可考虑额外维护索引数组，避免频繁搬移。

------

## 9 双向链表的创建与使用技巧

双向链表允许 `O(1)` 在任意已知结点前后插入/删除，常见于 **进程管理**、**LRU 缓存**、**事件循环** 等场景。

### 9.1 结点定义与初始化

```c
typedef struct DListNode {
    int            data;
    struct DListNode *prev;
    struct DListNode *next;
} DListNode;

static inline void dll_init(DListNode *node)
{
    node->prev = node->next = node; // 哨兵——指向自己
}
```

> **哨兵 (sentinel) 设计**：将头结点本身作为“环”，避免判空分支，所有操作统一。

### 

### 9.4 使用示例

```c
DListNode head;
DListNode n1 = {.data = 42}, n2 = {.data = 88};

dll_init(&head);
dll_init(&n1);
dll_init(&n2);

dll_add_back(&head, &n1);
dll_add_front(&head, &n2);

DListNode *iter;
dll_foreach(&head, iter) {
    printf("%d
", iter->data);
}
```

输出顺序：`88 42`

### 

------

------

## 10 POSIX 正则表达式速查（`<regex.h>`）

> **说明**：C 标准库本身未内置正则，但在 POSIX 系统可使用 `<regex.h>` 提供的 **BRE**（Basic）与 **ERE**（Extended）语法。下表以常用 **ERE** 为例；若使用 BRE，`+ ? | () {}` 需转义 `\`。

| 元字符           | 含义                          | 典型示例                      | 备注                                                 |
| ---------------- | ----------------------------- | ----------------------------- | ---------------------------------------------------- |
| `.`              | 匹配除换行外的任意单字符      | `a.c` ↔ `abc`、`a c`          | 若需包括换行可用 `(?s)` 或手动 `([\s\S])` 等扩展方案 |
| `*`              | 前一原子重复 0 次或更多       | `fo*` ↔ `f`、`fooo`           | 贪婪匹配                                             |
| `+`              | 前一原子重复 1 次或更多       | `fo+` ↔ `fo`、`fooooo`        | —                                                    |
| `?`              | 前一原子可无可有              | `colou?r` ↔ `color`、`colour` | —                                                    |
| `{m,n}`          | 指定重复次数区间              | `a{2,4}` ↔ `aa`/`aaa`/`aaaa`  | `{m}` 精确；`{m,}` 下限                              |
| `[]`             | 字符集合 / 范围               | `[A-F0-9]` ↔ HEX              | `[^...]` 表示取反                                    |
| `^` / `$`        | 行首 / 行尾锚点               | `^\d+$` 仅整行数字            | 多行模式下作用于每行                                 |
| `()`             | 捕获分组                      | `(\w+)=(\d+)`                 | 分组编号从 1 开始                                    |
| `                | `                             | 选择（或）                    | `abc                                                 | def` |
| `\d` `\w` `\s`   | 常用短写（某些实现，如 PCRE） | `\d{4}-\d{2}-\d{2}`           | 在 POSIX ERE 不支持，需 `[0-9]` 等                   |
| `[[:digit:]]` 等 | POSIX 字符类                  | `[[:alnum:]]+`                | 与区域设置相关                                       |

------

## 11 `sscanf` 中的“伪正则”格式字符串

虽然 `sscanf` 不是完整正则，但它的 **字符集合 (`%[...]`)** 和 **宽度限定** 能完成 80% 简单解析，且无需额外依赖，在嵌入式环境尤为实用。

### 11.1 字符集合 `%[...]`

| 形式           | 匹配规则                          | 示例输入   | 结果                     |
| -------------- | --------------------------------- | ---------- | ------------------------ |
| `%[A-Z]`       | 连续大写字母                      | `ABC123`   | 捕获 "ABC"，解析停在 `1` |
| `%[^,:]`       | 直到 **遇到** `,` **或** `:` 为止 | `name:Bob` | 捕获 "name"，停在 `:`    |
| `%[0-9a-fA-F]` | HEX 字符串                        | `7E9Fh`    | 捕获 "7E9F"，停在 `h`    |
| `%[]`          | 空集合非法                        | —          | —                        |
| `%[^]`         | 第一个 `]` 后闭合                 | —          | —                        |

> **要点**：
>
> 1. 若集合首字符是 `]` 或 `-`，放在第一位即可被视为字面量：`%[]A-Z]`、`%[-0-9]`。
> 2. 默认**贪婪**，会一直吃到不符合集合的字符或字符串结束。
> 3. 常与 `%*` 组合忽略无用字段：`%*[^,]` 跳过到 `,` 为止的内容。

### 11.2 宽度限定 & 停止符

- `%4[0-9]` 最多 4 字符数字，可避免缓冲区溢出。
- `%[^\n]%n` 持续读到行尾，并用 `%n` 得到已读长度，实现“逐行”扫描。

### 11.3 示例：解析 `key=value;` 键值对列表

```c
const char *cfg = "id=123;name=Bob;score=98;";
size_t off = 0, len = strlen(cfg);
while (off < len) {
    char key[16]  = {0};
    char val[16]  = {0};
    int  n = 0;
    if (sscanf(cfg + off, "%15[^=]=%15[^;];%n", key, val, &n) == 2) {
        printf("%s => %s
", key, val);
        off += n; // 跳到下一个块
    } else {
        break;    // 语法错误
    }
}
```

输出：

```
id => 123
name => Bob
score => 98
```

### 11.4 与正则表达式的对比

| 场景                  | `sscanf` 优势              | 正则优势                           |
| --------------------- | -------------------------- | ---------------------------------- |
| 嵌入式 / 裸机         | **无需动态内存**、无额外库 | –                                  |
| 简单分隔符（CSV/INI） | 格式直观，可逐字段解析     | –                                  |
| 复杂变长语法          | –                          | 支持回溯、非贪婪、断言、反向引用等 |

> **总结**：若能用 `%[]` + 宽度解决，就先用 `sscanf`；遇到复杂嵌套再考虑正则库。

------

### 12`printf`的重定向
```c
int fputc(int ch, FILE *f)
{
  HAL_UART_Transmit(&huart2, (uint8_t *)&ch, 1, 0xffff);
  return ch;
}

```