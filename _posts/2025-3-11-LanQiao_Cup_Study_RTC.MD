---
title: "蓝桥杯嵌入式外设开发记录 - RTC时钟"
date: 2025-03-11 11:43:05 +0800
updated: 2025-03-11 11:43:05 +0800
author: Cxxhh
categories: [蓝桥杯, 嵌入式,学习记录]
tags: [STM32, Keil, CubeMX]
description: "本文记录了学习蓝桥杯嵌入式开发过程中的RTC时钟的记录"
pin: false
toc: true
comments: true
math: false
mermaid: false
media_subpath: ""
---
# 蓝桥杯嵌入式外设开发记录 - RTC时钟

## 基本信息

| 项目       | 内容                                      |
| ---------- | ----------------------------------------- |
| 开发板型号 | CT117E（STM32G431RBT6）                   |
| 开发环境   | STM32CubeMX 6.6.1, Keil uVision5 MDK 5.38 |
| 记录日期   | 2025-3-4                                  |

## 1.CubeMX 配置

![](https://testingcf.jsdelivr.net/gh/Cxxhh/blog-img/img/RTC1.png)

## 2.相关代码

获取时间：

```c
RTC_TimeTypeDef sTime = {0};
RTC_DateTypeDef sDate = {0};
HAL_RTC_GetTime(&hrtc,&sTime,RTC_FORMAT_BIN);
HAL_RTC_GetDate(&hrtc,&sDate,RTC_FORMAT_BIN);
sprintf(text3,"%d:%d:%d             ",sTime.Hours,sTime.Minutes,sTime.Seconds);
LCD_DisplayStringLine(Line5,(uint8_t *)text3);
sprintf(text1,"%d-%d-%d-%d            ",sDate.Year,sDate.Month,sDate.Date,sDate.WeekDay);
LCD_DisplayStringLine(Line6,(uint8_t *)text1);
```

闹钟：

```c
void HAL_RTC_AlarmAEventCallback(RTC_HandleTypeDef *hrtc)
{
        LED_ds(7,1);
}
```

