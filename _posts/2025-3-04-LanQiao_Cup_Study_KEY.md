---
title: "蓝桥杯嵌入式外设开发记录 - 按键模块"
date: 2025-03-04 22:43:29 +0800
updated: 2025-03-07 11:43:05 +0800
author: Cxxhh
categories: [蓝桥杯, 嵌入式,学习记录]
tags: [STM32, Keil, CubeMX]
description: "本文记录了学习蓝桥杯嵌入式开发过程中的按键模块的记录"
pin: false
toc: true
comments: true
math: true
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

打开外部时钟，设置预分频系数为 79（即 80 分频），设置计数器值为 20000，并启用自动重装载.然后在NVIC settings内打开定时器中断。

![CubeMX Timer Configuration](https://testingcf.jsdelivr.net/gh/Cxxhh/blog-img/img/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_2025-03-04_214207_280.png)

### 1.2 定时器中断配置

溢出时间计算公式如下：

$$
\text{溢出时间} = \frac{(\text{Prescaler} + 1) \times (\text{Counter Period} + 1)}{\text{系统时钟频率}}
$$

---

#### 1.3初始化定时器和中断回调函数

```c
HAL_TIM_Base_Start_IT(&htim3); //打开定时器3 
```

可在：` stm32g4xx_hal_tim.h` 中找到回调函数：

```c
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim);
//然后在任.h内声明并在.c文件中使用
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
    if(htim->Instance == TIM3) //20ms
    {
       KEY_SACN();
    }

}
```



### 2. 代码演进

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
void KEY_SACN() {   
    /* 读取四个按键的当前物理状态（假设按下为低电平）*/
    KEY[0].key_sta = HAL_GPIO_ReadPin(B1_GPIO_Port, B1_Pin);  // 读取B1按键
    KEY[1].key_sta = HAL_GPIO_ReadPin(B2_GPIO_Port, B2_Pin);  // 读取B2按键
    KEY[2].key_sta = HAL_GPIO_ReadPin(B3_GPIO_Port, B3_Pin);  // 读取B3按键
    KEY[3].key_sta = HAL_GPIO_ReadPin(B4_GPIO_Port, B4_Pin);  // 读取B4按键

    /* 遍历所有按键进行状态机处理 */
    for (int i = 0; i < 4; i++) {   
        switch (KEY[i].judge_sta) {
            /*--------------------------------------------------
              状态0：初次检测按键按下（消抖准备阶段）
              作用：检测按键是否被按下，启动消抖流程
            --------------------------------------------------*/
            case 0: 
                if (KEY[i].key_sta == 0) {  // 检测到按键按下信号
                    KEY[i].judge_sta = 1;    // 进入消抖确认状态
                    KEY[i].long_time = 0;    // 重置长按计时器，避免历史值干扰
                }
                break;

            /*--------------------------------------------------
              状态1：消抖确认阶段
              作用：确认是否为有效按下（非抖动）
            --------------------------------------------------*/
            case 1: 
                if (KEY[i].key_sta == 0) {   // 持续检测到按下
                    KEY[i].judge_sta = 2;    // 确认有效按下，进入长按检测状态
                } else {                     // 按键已释放（抖动导致）
                    KEY[i].judge_sta = 0;    // 返回初始状态
                }
                break;

            /*--------------------------------------------------
              状态2：长按检测阶段
              作用：计时检测长按事件，处理释放时的单击事件
            --------------------------------------------------*/
            case 2: 
                if (KEY[i].key_sta == 0) {   // 按键仍被按住
                    KEY[i].long_time++;      // 增加长按计时
                    /* 达到长按阈值（例如24≈240ms，假设每10ms扫描一次）*/
                    if (KEY[i].long_time > 24) { 
                        KEY[i].judge_sta = 3; // 进入长按锁定状态
                    }
                } else {                     // 按键已释放
                    KEY[i].single_flag = 1;  // 标记单击事件
                    KEY[i].judge_sta = 0;    // 返回初始状态
                }
                break;        

            /*--------------------------------------------------
              状态3：长按锁定状态
              作用：等待按键释放后触发长按事件
            --------------------------------------------------*/
            case 3: 
                if (KEY[i].key_sta == 1) {   // 检测到按键释放
                    KEY[i].judge_sta = 0;    // 返回初始状态
                    KEY[i].long_flag = 1;    // 标记长按事件
                }
                break;
        } // End of switch
    } // End of for
} // End of KEY_SACN

```

**代码说明:**

- `long_time` 变量用于记录按键按下的时间。
- 在`case 2`中，如果按键持续按下，`long_time` 递增。如果按键释放，则根据 `long_time` 的值判断是长按还是短按。
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
