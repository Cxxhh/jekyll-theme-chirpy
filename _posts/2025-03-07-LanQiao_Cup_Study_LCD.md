---
title: "蓝桥杯嵌入式外设开发记录 - LCD模块"
date: 2025-03-07 11:43:05 +0800
updated: 2025-03-07 11:43:05 +0800
author: Cxxhh
categories: [蓝桥杯, 嵌入式,学习记录]
tags: [STM32, Keil, CubeMX, LCD]
description: "本文记录了学习蓝桥杯嵌入式开发过程中的LCD模块的记录"
pin: false
toc: true
comments: true
math: false
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

