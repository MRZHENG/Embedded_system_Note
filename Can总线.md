### CAN总线的接线图（Tx接Tx，Rx接Rx）



![image-20241022130525945](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241022130525945.png)





#### 信号接受

![image-20241022130553739](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241022130553739.png)



   RECEIVER ：检测电压差，有电压差输出1，无电压差输出0 ， 然后进入mos部分，有电压差的时候GND导通输入0

无电压差的时候，VCC导通输入1





#### 信号发送

#### TXD为 1 

![image-20241022130855678](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241022130855678.png)

   相当于不对右侧的can总线进行任何操作，总线在终端电阻的收紧下，呈现隐形电平（**也就是无电压差**）



#### TXD为 0

![image-20241022131217089](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241022131217089.png)

拉高，拉低后，两端总线实现电压差，使得其呈现显性电平，也就是（**有电压差**）





### Can总线帧格式

1. 数据帧
2. 遥控帧
3. 错误帧
4. 过载帧
5. 帧间隔

![image-20241023101652605](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241023101652605.png)

![image-20241023101708857](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241023101708857.png)

![image-20241023101732981](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241023101732981.png)

![image-20241023101751525](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241023101751525.png)

![image-20241023101811537](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241023101811537.png)

![image-20241023101822325](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241023101822325.png)

**位填充**

发送方，每发送五个相同电平后，**自动追加一个相反电平填充位，接收方检测到填充位时，会自动移除填充位**



### 位时序

Can总线，对每一个数据位的时长进行了更为细致的划分，有同步段，传播时间段，相位缓冲段，相位缓冲段2

![image-20241023103504122](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241023103504122.png)

#### 硬同步

描述，当发送方，发送报文时，其他所有设备( 接受方 ) 收到SOF的下降沿时，接收方会将自己的位时序计时拨到 SS 段的位置

与发送方 的 位时序计时周期保持同步

硬同步 只在 帧的第一个下降沿 有效

**理解，当发送方开始按下秒表计时时候，所有接收方重新设置秒表开始跑**

![image-20241023103822475](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241023103822475.png)

#### 再同步

 如果接受方，经行第一次硬同步后，由于两台设备的计时精确度不一样，可能存在误差，导致随着时间的延长，误差越来越大

这个时候就需要再同步来调节

**注意：再同步补偿宽度值（SJW）是最大的补偿位**

![image-20241023104122432](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241023104122432.png)

#### 波特率的计算

![image-20241023104244434](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241023104244434.png)





### 仲裁阶段

**资源分配规则1 -- 先占先得（当总线已经在使用的情况）** 

1. 如果已经有设备在正在操作总线 发送数据帧/遥控帧， **其他设备就不能再 同时发送数据帧和遥控帧**（但是可以发送 错误帧 / 过载帧破坏当前数据）
2. 任何设备检测到连续的11 个 隐形电平 就会认为总线空闲，只有在总线空闲的时候才能发送数据帧和遥控帧
3. 一旦有设备正在发送数据帧 /  遥控帧，总线就会被认为变成空闲状态，只有在总线空闲时，设备才会发送数据帧和遥控帧

**资源分配规则2 -- 非破坏性仲裁（当总线空闲，多个设备想发送 的情况）**

![image-20241023110058640](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241023110058640.png)

![image-20241023112229239](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241023112229239.png)





### Can 过滤器配置



![image-20241104230333420](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241104230333420.png)



**列表模式：准确筛选出合适的ID**

**屏蔽模式：当报文ID特别多的情况，有一些共同分类的时候，可以使用这个模式，**

**例如：有100 个 温度探测器 以报文 0x1 开头**

​           **有200 个 气压探测器 以报文 0x2  开头 这时候就可以使用 屏蔽模式**



#### 1.32位标识符屏蔽模式

   ![image-20241104231742784](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241104231742784.png)



**R2 在写 1 的 位表示当前位置必须匹配，写 0 的位 表示当前位置 写 0 和 1 均可**

#### 2.32位标识符列表模式



![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241104225755375.png)

​       32 位过滤器，标识符列表模式，**应用场景为 既有 标准ID 也有 扩展ID 的情况** ，R1 和 R2 可以分别存入一个 报文ID

#### 3.2个16位过滤器模式

![image-20241104232628840](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241104232628840.png)





#### 4.4个16位过滤器列表模式

![image-20241104231036724](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241104231036724.png)



### 过滤器配置示例

![image-20241104232839739](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241104232839739.png)



第一个样例

​    由于 ID 号不多，并且没什么规律，且全是标准ID 所以使用16 位列表模式

​    其中左移 5 位的原因是：

![image-20241104233133229](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241104233133229.png)

**关于屏蔽位的讲解**

例如：设置某接收滤波寄存器 00000000001（11 位），接收屏蔽寄存器 11111111101（11 位），则该对
组合会拒绝接收 00000000011 和 00000000001 之外所有的标识符对应的 CAN 帧，因为屏蔽器规定第二位
（为 0）以外的所有标识符位要严格匹配（与滤波器值一致），第二位的滤波器值和收到的 CAN 标识符第
二位值是否一致都可以。



**为什么第二样例 屏蔽位设置位 0x700 和 0x7F0 呢 ，ID最高位0x2 为 0010 和 0x3 为 0101 , 由于存在的 ID最高位 为 0x1 0x2 0x31 0x32  想要判断0x2  只需要 屏蔽最高位设置位 0x7 即可**





### FMI 过滤器匹配序号

![image-20241105151442684](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241105151442684.png)



**作用就是，当传送一个报文ID 的 时候，设置的过滤器们 应该 以什么排序方式，决定谁先去匹配这个报文ID **







### Can总线的测试模式

![image-20241105091954646](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241105091954646.png)





#### 静默模式（用于分析Can总线的活动，不会对总线造成影响）

![image-20241105092730416](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241105092730416.png)

#### 环回模式（用于自测试，同时发送报文可以在Can_Tx的引脚上检测到）

![image-20241105093047794](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241105093047794.png)

#### 静默环回模式

![image-20241105093216007](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241105093216007.png)

### 工作模式



![image-20241105095234967](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241105095234967.png)

![image-20241105101328637](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241105101328637.png)







### 位时间特性

![image-20241105095214569](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241105095214569.png)





### 中断

![QQ_1730772111769](C:\Users\ADMINI~1\AppData\Local\Temp\QQ_1730772111769.png)

### 时间触发通讯



![image-20241105100823165](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241105100823165.png)

### 错误处理和离线恢复

![image-20241105100923387](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241105100923387.png)







### 标准库的相关库函数

初始化相关函数

![QQ_1730787172533](C:\Users\ADMINI~1\AppData\Local\Temp\QQ_1730787172533.png)



接受与发送相关函数

![QQ_1730787573793](C:\Users\ADMINI~1\AppData\Local\Temp\QQ_1730787573793.png)



工作模式相关函数

![QQ_1730787756109](C:\Users\ADMINI~1\AppData\Local\Temp\QQ_1730787756109.png)



获取错误的函数

![image-20241105142347223](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241105142347223.png)



中断函数

![QQ_1730787972142](C:\Users\ADMINI~1\AppData\Local\Temp\QQ_1730787972142.png)



![image-20241105143855050](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241105143855050.png)

![QQ_1730788786923](C:\Users\ADMINI~1\AppData\Local\Temp\QQ_1730788786923.png)

![image-20241105144011917](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241105144011917.png)

![image-20241105144030873](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241105144030873.png)

![image-20241105144126446](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241105144126446.png)

![image-20241106103357652](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241106103357652.png)