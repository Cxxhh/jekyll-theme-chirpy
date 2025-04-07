---
title: "蓝桥杯嵌入式外设开发记录 - PWM输出 & 输入捕获"
date: 2025-03-11 11:43:05 +0800
updated: 2025-03-11 11:43:05 +0800
author: Cxxhh
categories: [蓝桥杯, 嵌入式,学习记录]
tags: [STM32, Keil, CubeMX]
description: "本文记录了学习蓝桥杯嵌入式开发过程中的PWM输出&输入捕获的记录"
pin: false
toc: true
comments: true
math: true
mermaid: false
media_subpath: ""
---
# 蓝桥杯嵌入式外设开发记录 - PWM输出 & 输入捕获

## 基本信息

| 项目       | 内容                                      |
| ---------- | ----------------------------------------- |
| 开发板型号 | CT117E (STM32G431RBT6)                    |
| 开发环境   | STM32CubeMX 6.6.1, Keil uVision5 MDK 5.38 |
| 记录日期   | 2025-03-04                                |

---

## 1. PWM波的输出

### 1.1 CubeMX 配置

![](https://testingcf.jsdelivr.net/gh/Cxxhh/blog-img/img/PWM%E8%BE%93%E5%87%BA.png)

**PWM频率计算公式：**


$$
f = \frac{f_{\text{system}}}{(ARR + 1)(PSC + 1)}
$$
**PWM周期计算公式：**
$$
T = \frac{1}{f} = \frac{(ARR + 1)(PSC + 1)}{f_{\text{system}}}
$$
**占空比计算公式：**
$$
\text{DutyCycle} = \frac{CCR}{ARR + 1}
$$

### 1.2 初始化代码

**启动PWM波：**

以下代码用于启动定时器2的通道2的PWM输出。

```c
HAL_TIM_PWM_Start(&htim2, TIM_CHANNEL_2);
```

**修改占空比：**

以下代码直接修改定时器2的CCR2寄存器值，从而改变PWM的占空比。

```c
TIM2->CCR2 = 500;
```

## 2. 输入捕获

### 2.1 CubeMX 配置

![](https://testingcf.jsdelivr.net/gh/Cxxhh/blog-img/img/%E8%BE%93%E5%85%A5%E6%8D%95%E8%8E%B7.png)

**频率计算公式：**
$$
f = \frac{f_{\text{sys}}}{(PSC + 1) \times \text{capture\_value}}
$$

### 2.2 初始化代码

**启动输入捕获中断：**

以下代码用于启动定时器17的通道1的输入捕获，并使能中断。

```c
HAL_TIM_IC_Start_IT(&htim17, TIM_CHANNEL_1);
```

**输入捕获中断回调函数：**

以下代码展示了输入捕获中断回调函数 `HAL_TIM_IC_CaptureCallback` 的实现。该函数在每次捕获事件发生时被调用。

```c
uint32_t Fre, capture_value;

void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim) {
    if (htim->Instance == TIM17) {
        capture_value = htim->Instance->CCR1;
        TIM17->CNT = 0; // 重置计数器
        Fre = 1000000 / capture_value; // 计算频率
    }
}
```

### 2.3 捕获NE555信号发生器信号

**输入捕获中断回调函数示例 (针对TIM2和TIM16)：**

以下代码展示了如何针对不同的定时器（TIM2 和 TIM16）编写输入捕获中断回调函数。注意，此处的频率计算公式 `TIMx_Fre = 80000000/(80*TIMx_capture_value)` 依赖于特定的预分频系数和系统时钟频率。

```c
void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim) {
    if (htim->Instance == TIM2) {
        TIM2_capture_value = TIM2->CCR1;
        TIM2->CNT = 0;
        TIM2_Fre = 80000000 / (80 * TIM2_capture_value);
        HAL_TIM_IC_Start(&htim2, TIM_CHANNEL_1);
    }
    if (htim->Instance == TIM16) {
        TIM16_capture_value = TIM16->CCR1;
        TIM16->CNT = 0;
        TIM16_Fre = 80000000 / (80 * TIM16_capture_value);
        HAL_TIM_IC_Start(&htim16, TIM_CHANNEL_1);
    }
}
```

### 2.4 测量占空比

```c
// 输入捕获中断回调函数，当TIM的输入捕获事件触发时由HAL库调用
void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim)
{
    // 判断触发中断的定时器是否为TIM2
    if(htim->Instance == TIM2)
    {
        // 判断触发中断的通道是否为通道1
        if(htim->Channel == HAL_TIM_ACTIVE_CHANNEL_1)
        {
            // 获取通道1的捕获值（通常对应上升沿捕获）
            TIM2_capture_value = TIM2->CCR1;

            // 重置TIM2计数器，为下次测量做准备
            TIM2->CNT = 0 ;

            // 计算信号频率
            TIM2_Fre = 1000000/TIM2_capture_value;

            // 计算占空比：用下降沿值除以上升沿周期值，转换为百分比
            duty = ((float)TIM2_capture_value_fall/(float)TIM2_capture_value)*100;
        }
        // 判断触发中断的通道是否为通道2
        else if(htim->Channel == HAL_TIM_ACTIVE_CHANNEL_2)
        {
            // 获取通道2的捕获值（通常对应下降沿捕获）
            TIM2_capture_value_fall = TIM2->CCR2;
        }

        // 重新启动通道1和通道2的输入捕获（HAL库捕获后会自动停止通道）
        HAL_TIM_IC_Start(&htim2,TIM_CHANNEL_1);
        HAL_TIM_IC_Start(&htim2,TIM_CHANNEL_2);

    }
}

```
