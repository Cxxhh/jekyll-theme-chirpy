---
title: "蓝桥杯嵌入式外设开发记录 - 按键模块"
date: 2024-03-04 22:43:29 +0800
updated: 2024-03-04 22:43:29 +0800
author: Cxxhh
categories: [蓝桥杯, 嵌入式,学习记录]
tags: [STM32, Keil, CubeMX, 按键]
description: "本文记录了学习蓝桥杯嵌入式开发过程中的按键模块的记录"
pin: false
toc: true
comments: true
math: false
mermaid: false
media_subpath: ""
---

# 蓝桥杯嵌入式外设开发记录 - 按键模块

## 基本信息

| 项目       | 内容                                      |
| ---------- | ----------------------------------------- |
| 开发板型号 | CT117E（STM32G431RBT6）                   |
| 开发环境   | STM32CubeMX 6.6.1, Keil uVision5 MDK 5.38 |
| 记录日期   | 2025-3-4                                  |

---

## 1. CubeMX 配置

### 1.1 时钟配置

打开外部时钟，设置预分频系数为 79（即 80 分频），设置计数器值为 20000，并启用自动重装载。

![CubeMX Timer Configuration](https://testingcf.jsdelivr.net/gh/Cxxhh/blog-img/img/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_2025-03-04_214207_280.png)

### 1.2 定时器中断配置

溢出时间计算公式如下：

$$
\text{溢出时间} = \frac{(\text{Prescaler} + 1) \times (\text{Counter Period} + 1)}{\text{系统时钟频率}}
$$

---

## 2. 代码演进

### 2.1 基础按键检测 (v1.0)

```c
void KEY_SACN()
{
    KEY[0].key_sta = HAL_GPIO_ReadPin(B1_GPIO_Port, B1_Pin);  //B1
    KEY[1].key_sta = HAL_GPIO_ReadPin(B2_GPIO_Port, B2_Pin);  //B2
    KEY[2].key_sta = HAL_GPIO_ReadPin(B3_GPIO_Port, B3_Pin);  //B3
    KEY[3].key_sta = HAL_GPIO_ReadPin(B4_GPIO_Port, B4_Pin);  //B4
    for(int i=0; i<4; i++)
    {
        switch(KEY[i].judge_sta)
        {
            case 0: if(KEY[i].key_sta == 0) KEY[i].judge_sta++;  //消抖
                    break;
            case 1: if(KEY[i].key_sta == 0) KEY[i].judge_sta++; //确认按下
                    else 
                    KEY[i].judge_sta=0; // 弹起，重置状态
                    break;
            case 2: if(KEY[i].key_sta == 1) 
                    {
                    KEY[i].judge_sta=0;   // 状态机归零
                    KEY[i].single_flag =1; // 单击标志位置位
                    }
                    break;
        }
    
    }
}
```

### 2.2 加入长按检测

```c
void KEY_SCAN() {
    for (int i = 0; i < 4; i++) {
        KEY[i].key_sta = HAL_GPIO_ReadPin(KEY_GPIO_Port[i], KEY_Pin[i]);

        switch (KEY[i].judge_sta) {
            case 0: // 状态0：等待按键按下
                if (KEY[i].key_sta == 0) {
                    KEY[i].judge_sta++; // 进入状态1
                }
                break;
            case 1: // 状态1：确认按键按下
                if (KEY[i].key_sta == 0) {
                    KEY[i].judge_sta++; // 进入状态2
                } else {
                    KEY[i].judge_sta = 0; // 弹起，重置状态
                }
                break;
            case 2: // 状态2：按键持续按下
                if (KEY[i].key_sta == 0) {
                    KEY[i].long_time++; // 长按计时
                } else {
                    // 弹起，判断长按或短按
                    if (KEY[i].long_time > 24) {
                        KEY[i].long_flag = 1; // 长按标志位置位
                        KEY[i].long_time = 0; // 长按计时清零
                    } else {
                        KEY[i].single_flag = 1; // 短按标志位置位
                    }
                    KEY[i].judge_sta = 0; // 状态机归零
                }
                break;
        }
    }
}

```
**代码说明:**
- `long_time` 变量用于记录按键按下的时间。
-  在`case 2`中，如果按键持续按下，`long_time` 递增。如果按键释放，则根据 `long_time` 的值判断是长按还是短按。
- `long_flag` 用于标记长按事件。
- `single_flag` 用于标记短按事件。

### 2.3 加入双击检测

```c
// TODO: 实现双击检测功能
```
**待办事项：**

-  实现双击检测逻辑。
-  可以考虑引入一个时间窗口，在短时间内检测到两次短按则认为是双击。

---
