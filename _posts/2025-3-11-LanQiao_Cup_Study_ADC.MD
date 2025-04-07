---
title: "蓝桥杯嵌入式外设开发记录 - ADC电压采样"
date: 2025-03-11 11:43:05 +0800
updated: 2025-03-11 11:43:05 +0800
author: Cxxhh
categories: [蓝桥杯, 嵌入式,学习记录]
tags: [STM32, Keil, CubeMX]
description: "本文记录了学习蓝桥杯嵌入式开发过程中的ADC电压采样的记录"
pin: false
toc: true
comments: true
math: true
mermaid: false
media_subpath: ""
---
# 蓝桥杯嵌入式外设开发记录 - ADC电压采样

## 基本信息

| 项目       | 内容                                      |
| ---------- | ----------------------------------------- |
| 开发板型号 | CT117E（STM32G431RBT6）                   |
| 开发环境   | STM32CubeMX 6.6.1, Keil uVision5 MDK 5.38 |
| 记录日期   | 2025-3-4                                  |

## 1.CubeMX 配置

### ![](https://testingcf.jsdelivr.net/gh/Cxxhh/blog-img/img/ADC.png)

配置完成引脚后在`Anlog`的`ADC1`和`ADC2`内的输入通道勾上`Single-ended`

## 2.相关代码

```c
double Get_ADC(ADC_HandleTypeDef *hadc)
{
    HAL_ADC_Start(hadc); //可在相关stm32g4xx_hal_adc.h找到
    return HAL_ADC_GetValue(hadc)*3.3/4096; //参考电压为3V3，12bit的分辨率
}
```
### 3.DMA连续采样（非阻塞获取）

需要添加一个DMA请求：


![](https://testingcf.jsdelivr.net/gh/Cxxhh/blog-img/img/68747470733a2f2f73686565702d70686f746f2e6f73732d636e2d7368656e7a68656e2e616c6979756e63732e636f6d2f696d672f3230323330313137313530333032372e706e67)

增加以下配置：

![](https://testingcf.jsdelivr.net/gh/Cxxhh/blog-img/img/68747470733a2f2f73686565702d70686f746f2e6f73732d636e2d7368656e7a68656e2e616c6979756e63732e636f6d2f696d672f3230323330313137313530303936312e706e67)

![](https://testingcf.jsdelivr.net/gh/Cxxhh/blog-img/img/68747470733a2f2f73686565702d70686f746f2e6f73732d636e2d7368656e7a68656e2e616c6979756e63732e636f6d2f696d672f3230323330313137313434303638372e706e67)

![](https://testingcf.jsdelivr.net/gh/Cxxhh/blog-img/img/68747470733a2f2f73686565702d70686f746f2e6f73732d636e2d7368656e7a68656e2e616c6979756e63732e636f6d2f696d672f3230323330313235313230393332302e706e67)

完成上面的设置后即可开始编写代码：

`16 adc1_buff[1] = {0}; // 接收数组`

`HAL_ADC_Start_DMA(&hadc1,(u32 *)adc1_buff,1); //打开dma的adc采样`

```c
while (1)
  {
    /* USER CODE END WHILE */
    uint16_t channel1_value = adc1_buff[0]; //获取数值
  }
```


