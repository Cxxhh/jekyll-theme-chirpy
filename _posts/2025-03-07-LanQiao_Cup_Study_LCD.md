---
title: "蓝桥杯嵌入式外设开发记录 - LCD模块"
date: 2025-03-07 11:43:05 +0800
updated: 2025-03-07 11:43:05 +0800
author: Cxxhh
categories: [蓝桥杯, 嵌入式,学习记录]
tags: [STM32, Keil, CubeMX]
description: "本文记录了学习蓝桥杯嵌入式开发过程中的LCD模块的记录"
pin: false
toc: true
comments: true
math: true
mermaid: false
media_subpath: ""
---
# 蓝桥杯嵌入式外设开发记录 - 按键模块

<details>
<summary>📌 基本信息</summary>


**开发板型号**：CT117E（STM32G431RBT6）  
**开发环境**：  

- STM32CubeMX 6.6.1  
- Keil uVision5 MDK 5.38  
- ST-Link Utility 4.6.0  
  **记录日期**：2023-10-20  
  **代码版本**：v1.2.3  
  </details>

---

## 1. 解决LCD和LED引脚冲突

找到下面两个函数的实现：

```c
LCD_Init();
LCD_Clear(Blue);
```

分别在两个函数内加入以下代码：

```c
void LCD_Clear(u16 Color)
{   
    uint16_t temp = GPIOC->ODR; //新增
    
	u32 index = 0;
	LCD_SetCursor(0x00, 0x0000); 
	LCD_WriteRAM_Prepare(); /* Prepare to write GRAM */
	for(index = 0; index < 76800; index++)
	{
		LCD_WriteRAM(Color);    
	}
    GPIOC->ODR = temp;  //新增
}
```

```c
void LCD_Init(void)
{ 
    uint16_t temp = GPIOC->ODR; //新增
	//中间代码保持不变.......
    GPIOC->ODR = temp; //新增
}
```

## 2.LCD的高亮显示

涉及到LCD的高亮显示的函数是：`LCD_SetBackColor(Yellow);`

```c
void lcd_show(void) {
    // 预定义每行的固定文本内容（避免重复调用sprintf）
    const char *lines[] = {
        "         text0           ",
        "         text1           ",
        "         text2           ",
        "         text3           "
    };

    // 定义每行对应的显示行号（假设Line0~Line3是连续的）
    uint8_t line_numbers[] = {Line0, Line1, Line2, Line3};
    // 遍历所有行
    for (int i = 0; i < 4; i++) {
        // 动态设置背景颜色
        if (i == LCD_HighShow_Line) {
            LCD_SetBackColor(Yellow);  // 高亮行
        } else {
            LCD_SetBackColor(Black);   // 普通行
        }

        // 显示当前行文本
        LCD_DisplayStringLine(line_numbers[i], (uint8_t *)lines[i]);
    }
}
```
## 1. 原始 LCD 显示字符代码

### 函数原型

```c
void LCD_DisplayChar(u8 Line, u16 Column, u8 Ascii) {
    Ascii -= 32; // 跳过 ASCII 前 32 个非打印字符
    LCD_DrawChar(Line, Column, &ASCII_Table[Ascii * 24]); // 从字模表中获取数据
}
```

---

## 2. 问题场景

### 错误调用代码

```c
LCD_DisplayChar(Line6, (319 - 10*16), sTime.Seconds); // 直接传递数值
```

### 问题现象

- 当 `sTime.Seconds = 50` 时，显示字符 `'2'`（因为 `50` 是 ASCII 码 `'2'`）。
- **根本原因**：未将数值转换为 ASCII 字符，直接传递了原始数值。

---

## 3. 修正后的代码  

### 步骤说明

#### 1. **拆分十位和个位**

```c
uint8_t seconds = sTime.Seconds; // 示例值: 50
uint8_t tens = seconds / 10;     // 十位: 5
uint8_t ones = seconds % 10;     // 个位: 0
```

#### 2. **转换为 ASCII 字符**

```c
uint8_t ascii_tens = tens + '0'; // 5 → '5' (ASCII 53)
uint8_t ascii_ones = ones + '0'; // 0 → '0' (ASCII 48)
```

#### 3. **显示字符**

```c
// 假设每个字符宽度为 16 像素
LCD_DisplayChar(Line6, (319 - 9*16), ascii_tens); // 显示十位 '5'
LCD_DisplayChar(Line6, (319 - 10*16), ascii_ones); // 显示个位 '0'
```

---

## 4. 完整示例

```c
// 获取秒数值
uint8_t seconds = sTime.Seconds;

// 拆分十位和个位
uint8_t tens = seconds / 10;
uint8_t ones = seconds % 10;

// 转换为 ASCII 字符
uint8_t ascii_tens = tens + '0';
uint8_t ascii_ones = ones + '0';

// 显示十位和个位
LCD_DisplayChar(Line6, (319 - 9*16), ascii_tens); // 十位位置
LCD_DisplayChar(Line6, (319 - 10*16), ascii_ones); // 个位位置
```

---

## 5. 关键说明

### 列坐标计算

- **`(319 - 9*16)`**：  
  屏幕宽度为 320 像素，每个字符占 16 像素宽度。  
  十位位置：`320 - 9字符*16像素 = 320 - 144 = 176`（右对齐第9字符位置）。

- **`(319 - 10*16)`**：  
  个位位置：`320 - 10字符*16像素 = 320 - 160 = 160`（右对齐第10字符位置）。

### 字符宽度调整

- 若字模实际宽度为 **8 像素**，需修改坐标增量：

  ```c
  LCD_DisplayChar(Line6, (319 - 18*8), ascii_tens); // 十位
  LCD_DisplayChar(Line6, (319 - 20*8), ascii_ones); // 个位
  ```

---

## 6. 注意事项

1. **数值范围验证**：  
   确保 `sTime.Seconds ≤ 99`，否则拆分逻辑需调整为 `% 100` 和 `/ 100`。

2. **字模表兼容性**：  
   确认 `ASCII_Table` 的偏移量是否正确（`Ascii -= 32` 需对应字模表从空格字符开始）。

3. **屏幕对齐**：  
   根据实际屏幕分辨率和字符宽度调整列坐标。

---

## 7. 扩展场景

### 显示时间格式 `HH:MM:SS`

```c
// 示例：显示 "12:34:56"
uint8_t hour = 12, minute = 34, second = 56;

// 小时十位和个位
LCD_DisplayChar(Line1, 0, (hour / 10) + '0');   // '1'
LCD_DisplayChar(Line1, 16, (hour % 10) + '0');  // '2'

// 分隔符
LCD_DisplayChar(Line1, 32, ':'); // 显示冒号

// 分钟十位和个位
LCD_DisplayChar(Line1, 48, (minute / 10) + '0'); // '3'
LCD_DisplayChar(Line1, 64, (minute % 10) + '0'); // '4'
```

---

**通过以上步骤，可正确将数值转换为 ASCII 字符并显示在 LCD 上。**
