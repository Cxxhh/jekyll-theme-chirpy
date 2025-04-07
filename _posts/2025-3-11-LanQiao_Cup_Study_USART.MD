---
title: "蓝桥杯嵌入式外设开发记录 - 串口通信"
date: 2025-03-11 11:43:05 +0800
updated: 2025-03-11 11:43:05 +0800
author: Cxxhh
categories: [蓝桥杯, 嵌入式,学习记录]
tags: [STM32, Keil, CubeMX]
description: "本文记录了学习蓝桥杯嵌入式开发过程中的串口通信的记录"
pin: false
toc: true
comments: true
math: true
mermaid: false
media_subpath: ""
---
# 蓝桥杯嵌入式外设开发记录 - PWM输出&输入捕获

## 基本信息

| 项目       | 内容                                      |
| ---------- | ----------------------------------------- |
| 开发板型号 | CT117E（STM32G431RBT6）                   |
| 开发环境   | STM32CubeMX 6.6.1, Keil uVision5 MDK 5.38 |
| 记录日期   | 2025-3-4                                  |

---

## 1.CubeMX 配置

### ![](https://testingcf.jsdelivr.net/gh/Cxxhh/blog-img/img/%E4%B8%B2%E5%8F%A3.png)

## 2.基础代码

发送函数（初始化时需要在`main`内加入一句作为使能）：

```c
HAL_UART_Transmit(UART_HandleTypeDef *huart, const uint8_t *pData, uint16_t Size, uint32_t Timeout)
```

接收中断回调：

```c
uint8_t rec_data;
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
    if(huart->Instance == USART1)
    {
        HAL_UART_Transmit(huart,&rec_data,1,50);
        HAL_UART_Receive_IT(huart,&rec_data,1);
    }

}
```

## 3.串口接收不定长数据

```c
uint8_t rx_data,count;
bool rx_flag;
char rx_buff[40];
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
    if(huart->Instance == USART1)
    {
        TIM4->CNT = 0;
        rx_flag =1;
        rx_buff[count] = rx_data;
        count++;
        HAL_UART_Receive_IT(huart,&rx_data,1);
    }

}
const char* txt = "lan";
const char* Er = "Error\n";
void uart_data_rx()
{
    if(rx_flag)
    {
        if(TIM4->CNT >= 15)
        {
            rx_buff[count] = '\0';
            if(strcmp(rx_buff,txt) == 0)
            {
               HAL_UART_Transmit(&huart1,(uint8_t *)rx_buff,strlen(rx_buff),50);
            }
            else
            {

                HAL_UART_Transmit(&huart1,(uint8_t *)Er,strlen(Er),50);


            }
            rx_flag=0;
            memset(rx_buff,0,40);
            count =0;
        }
    }
}
```

$$
CNT一个计数：CNT_{1} = \frac{1}{\frac{Freq}{PSC+1}} = \frac{1}{\frac{80\text{MHz}}{8000}} = \frac{1}{10\text{kHz}} = 0.1\text{ms}
$$
#### 2>使用串口中断+DMA+空闲中断来实现



![](https://testingcf.jsdelivr.net/gh/Cxxhh/blog-img/img/%E4%B8%B2%E5%8F%A3-dma.png)

在`stm32g4xx_it.c`中加入以下代码 ：

```c
//用户自定义中断
void USER_UART_IRQHandler(UART_HandleTypeDef *huart)
{
    if(__HAL_UART_GET_FLAG(huart,UART_FLAG_IDLE))
    {
        __HAL_UART_CLEAR_IDLEFLAG(huart);//清除空闲中断
        USER_UART_IDLE_CallBack(huart); //用户回调，在需要的地方定义并编写

    }
}
//找到以下函数 并加入
void USART1_IRQHandler(void)
{
  /* USER CODE BEGIN USART1_IRQn 0 */

  /* USER CODE END USART1_IRQn 0 */
  HAL_UART_IRQHandler(&huart1);
  /* USER CODE BEGIN USART1_IRQn 1 */
    USER_UART_IRQHandler(&huart1); /////用户自定义中断
  /* USER CODE END USART1_IRQn 1 */
}
```

> 定义并编写`void USER_UART_IDLE_CallBack(UART_HandleTypeDef *huart);`***PS:必须要和用户中断内的回调名一样***

```c
char rx_buff[128];
void USER_UART_IDLE_CallBack(UART_HandleTypeDef *huart)
{
    if(huart->Instance == USART1)
    {
        HAL_UART_DMAStop(huart);
       // rx_buff[strlen(rx_buff)] = '\0';
       HAL_UART_Transmit(huart,(uint8_t *)rx_buff,strlen(rx_buff),50);
       memset(rx_buff,0,128);
        HAL_UART_Receive_DMA(huart,(uint8_t *)rx_buff,128);
    }



}
```

> 最后在主函数内初始化

`__HAL_UART_ENABLE_IT(&huart1,UART_IT_IDLE);`

`HAL_UART_Receive_DMA(&huart1,(uint8_t *)rx_buff,128);`

**注意：上面两个函数下不应该有HAL库的函数 否则会出现初始化后进入一次串口中断**
