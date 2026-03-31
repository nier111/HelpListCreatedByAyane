# Hydroacoustic Board PCB

## Overview

本文档主要用于记录Hydroacoustic Board PCB的设计思路、PCB布局、元器件选型、PCB制作、测试、调试、使用等内容。

## Component Selection

### RS232

选择使用 Ti 公司的 MAX3232E 系列芯片。

- 有两组通信口，可以一组用于数据传输，一组用于调试，输出日志。
- RS232协议是高电平为0，低电平为1的串行通信协议，优点在于抗干扰更好，空闲状态稳定。