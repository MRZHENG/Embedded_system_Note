## TIM定时器

### TIM定时器类型概述

![image-20240925143830108](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240925143830108.png)









### 基本定时器（只支持向上计数的模式）



**记时功能**

![image-20240925143629085](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240925143629085.png)



**主模式触发DAC功能**（让内部硬件不受程序控制实现自动运行）





![image-20240925144351817](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240925144351817.png)





### 中级定时器

![image-20240925144607788](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240925144607788.png)

**支持 向上计数，向下计数，以及中央对齐计数这三种模式**



#### 外部时钟模式2

![image-20240925144928955](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240925144928955.png)

**用于对ETR外部引脚提供时钟，或者对ETR时钟计数，就可以将这个定时器当作计数器来使用**





#### 外部时钟模式1（可以触发定时器 的从模式）

![image-20240925145208552](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240925145208552.png)



![image-20240925145826707](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240925145826707.png)





#### 实现定时器的级联功能

![image-20240925145531308](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240925145531308.png)





![image-20240925150355159](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240925150355159.png)







![image-20240925150554121](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240925150554121.png)

**注意：输入捕获，和输出比较是不能同时使用的**





### 高级定时器

![image-20240925150831813](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240925150831813.png)



![image-20240925151019958](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240925151019958.png)

![image-20240925151144209](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240925151144209.png)









### 时钟树

![image-20240925160616973](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240925160616973.png)









### TIM定时器配置

![image-20240925162613557](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240925162613557.png)



#### 步骤

1. 开启RCC时钟
2. 选择时基单元的时钟源
3. 配置时基单元
4. 配置输出中断控制，允许更新中断输出到NVIC
5. 配置NVIC，在NVIC中打开定时器中断的通道，并且配一个优先级
6. 允许控制，使能计数器
7. 书写一个定时器中断函数