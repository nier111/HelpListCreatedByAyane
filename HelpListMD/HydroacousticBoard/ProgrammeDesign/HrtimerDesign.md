# HRTimer Design

---

## 1. Overview

This document describes the design of HRTimer in HydroacousticBoard.

主要实现功能如下：

- 1.可以根据一个数组中的数据，作为高低电平时间的基准，由此生成PWM波信号。
- 2.输出两个通道，输出两路互补的PWM信号。

---

## 2. HRTimer Design

- 1. 数组存储高低电平：

```c
typedef struct {
    uint16_t high;  // TC1 高电平计数
    uint16_t low;   // TC1 低电平计数
} PWMStep;

PWMStep pwm_steps[] = {
    {100, 50},
    {120, 80},
    {90, 60},
};
uint8_t pwm_index = 0;
uint8_t pwm_steps_len = sizeof(pwm_steps)/sizeof(pwm_steps[0]);
```

- 2. 更新 PWM 输出：

```c
void UpdatePWM(void)
{
    uint16_t high = pwm_steps[pwm_index].high;
    uint16_t low  = pwm_steps[pwm_index].low;
    uint16_t period = high + low;

    // 更新 TC1 高电平
    HAL_HRTIM_WaveformCompareConfig(&hhrtim1, HRTIM_TIMERINDEX_TIMER_C,
        HRTIM_COMPAREUNIT_1, &(HRTIM_CompareCfgTypeDef){.CompareValue = high});

    // TC2 反相输出
    HAL_HRTIM_WaveformCompareConfig(&hhrtim1, HRTIM_TIMERINDEX_TIMER_C,
        HRTIM_COMPAREUNIT_2, &(HRTIM_CompareCfgTypeDef){.CompareValue = period - high});

    // 更新下一步
    pwm_index++;
    if(pwm_index >= pwm_steps_len) pwm_index = 0;
}
```

- 3. 初始化启动：

```c
HAL_HRTIM_WaveformCounterStart(&hhrtim1, HRTIM_TIMERINDEX_TIMER_C);
HAL_HRTIM_WaveformOutputStart(&hhrtim1, HRTIM_TIMERINDEX_TIMER_C, HRTIM_OUTPUT_TC1 | HRTIM_OUTPUT_TC2);
```

- 4. 用 DMA 将数据传入比较寄存器：

```c

```

## 3. 实验

### 3.1 实验一：测试 DMA 传输。

- 1. 在 cubeMX 的配置中，需要只设置 Memory 地址递增，关闭 peripheral 地址递增，不然 会只传入数组中的第一个元素。

- 2. Data Width 一般设置为 32 位，寄存器大小，即 word size 。

- 3. DMA 传输模式需要设置为 Circular ，开启循环传入寄存器。