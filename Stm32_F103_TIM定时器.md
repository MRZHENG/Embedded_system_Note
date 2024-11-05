## Stm32_F103_TIM定时器

### 系统滴答定时器

#### SysTick 寄存器结构体

![image-20240910220311692](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240910220311692.png)

![image-20240910220252181](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240910220252181.png)

![image-20240910215156086](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240910215156086.png)



**解读延时函数**

![image-20240910222334778](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240910222334778.png)



![image-20240910222646913](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240910222646913.png)





#### 定时器分类：

1. 基本定时器（定时）
2. 通用定时器（定时+输出比较+输入捕获）
3. 高级定时器（定时+输出比较+输入捕获+互补输出）

![image-20240809151521114](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240809151521114.png)

定时器的频率（f）为 72m/(PSC+1)

ARR为计数了多少次,决定计数器计多少个数

中断周期 T= ARR * 1/f

#### PWM输出捕获

1. CNT计数器
2. CCR输出比较寄存器  （当CNT ==CCR 电频翻转）
3. ARR自动重装载寄存器 （当CNT ==ARR，CNT变成0 ）
4. PSC配置 计数器的时钟

#### PWM计算公式

1. PWM频率 = CK_PSC / (PSC+1) /（ARR+1）
2. 占空比 ：  CCR /（ARR+1）

![image-20240809120055256](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240809120055256.png)

**自动重装载寄存器（ARR）在 STM32 定时器中用于设置定时器的周期。当定时器计数到 ARR 的值时，它会自动重置并重新开始计数**

![image-20240927150737280](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240927150737280.png)