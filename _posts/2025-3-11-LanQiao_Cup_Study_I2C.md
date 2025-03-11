---
title: "蓝桥杯嵌入式外设开发记录 - I2C通信(eeprom读写)"
date: 2025-03-11 11:43:05 +0800
updated: 2025-03-11 11:43:05 +0800
author: Cxxhh
categories: [蓝桥杯, 嵌入式,学习记录]
tags: [STM32, Keil, CubeMX]
description: "本文记录了学习蓝桥杯嵌入式开发过程中的I2C通信(eeprom读写)的记录"
pin: false
toc: true
comments: true
math: false
mermaid: false
media_subpath: ""
---
# 蓝桥杯嵌入式外设开发记录 - I2C通信

## 基本信息

| 项目       | 内容                                      |
| ---------- | ----------------------------------------- |
| 开发板型号 | CT117E（STM32G431RBT6）                   |
| 开发环境   | STM32CubeMX 6.6.1, Keil uVision5 MDK 5.38 |
| 记录日期   | 2025-3-4                                  |

## 1.CubeMX 配置

![](https://testingcf.jsdelivr.net/gh/Cxxhh/blog-img/img/I2C.png)

## 2.相关代码

```c
/* EEPROM读取函数
 * @param addr - 要读取的EEPROM内部地址
 * @return 读取到的数据（1字节）
 */
uint8_t eeprom_read(uint8_t addr)
{
    /* 第一阶段：发送要读取的目标地址 */
    I2CStart();                  // 发起I2C起始信号
    I2CSendByte(0xa0);           // 发送设备地址 + 写模式（0xA0 = 10100000b）
    I2CWaitAck();                // 等待从设备应答
    I2CSendByte(addr);           // 发送要读取的内存地址
    I2CWaitAck();                // 等待从设备应答
    I2CStop();                   // 发送停止信号，结束本次操作
    
    /* 第二阶段：读取数据 */
    I2CStart();                  // 再次发起I2C起始信号
    I2CSendByte(0xa1);           // 发送设备地址 + 读模式（0xA1 = 10100001b）
    I2CWaitAck();                // 等待从设备应答
    uint8_t re_dat = I2CReceiveByte();  // 接收数据字节
    I2CSendNotAck();             // 发送非应答信号（NACK），表示读取结束
    I2CStop();                   // 发送停止信号
    return re_dat;               // 返回读取到的数据
}

/* EEPROM写入函数
 * @param addr - 要写入的EEPROM内部地址
 * @param dat  - 要写入的数据（1字节）
 */
void eeprom_write(uint8_t addr, uint8_t dat)
{
    /* 单次写入流程 */
    I2CStart();                  // 发起I2C起始信号
    I2CSendByte(0xa0);           // 发送设备地址 + 写模式（0xA0）
    I2CWaitAck();                // 等待从设备应答
    I2CSendByte(addr);           // 发送目标内存地址
    I2CWaitAck();                // 等待从设备应答
    I2CSendByte(dat);            // 发送要写入的数据
    I2CWaitAck();                // 等待从设备应答
    I2CStop();                   // 发送停止信号，结束传输
    
    /* 注意：实际EEPROM写入需要约5-10ms的存储时间
       后续操作建议添加适当延迟或轮询设备就绪状态 */
}

```

