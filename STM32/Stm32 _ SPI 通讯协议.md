### Stm32 _ SPI 通讯协议

**分为四种线：**

1. SS线（CS,NSS）用于连接从机，主机低电平时候表示对应从机被选中
2. SCK (serial  Clock ) 时钟信号线，用于数据同步，最高速率 Fpclk/ 2 (APB1 = 72MHz 和APB2 = 36MHz 的时钟速率) 
3. MOSI(Master Output , Slave Input) 主设备输出/从设备输入引脚
4. MISO(Master Input , Slave Output) 主设备输入/从设备输出引脚



**CPOL 时钟极性**

定义：指在从机未选中时，SCK的空闲状态 ： 0 代表低电平 1 代表高电平

![image-20240907130513837](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240907130513837.png)

![image-20240907130645509](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240907130645509.png)

![image-20240907131159750](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240907131159750.png)

其中CPOL 定义了 SCK 时钟状态 ，CPHA 定义了时奇数边沿还是偶数边沿采样

**所谓采样就是移入数据到移位寄存器，移出数据就是图表中的![image-20240908173229078](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240908173229078.png)**， 所有的采样前提就是要移除数据，所以当为奇数采样的时候，在0时刻就已经提前移除数据

![image-20240917132244752](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240917132244752.png)

#### STM32 SPI  架构解析

![image-20240907132209023](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240907132209023.png)



#### 1.SCK作为从机时从外部接受时钟信号，作为主机从内部发送时钟信号

#### 2. 采用软件控制NSS引脚





#### 主机发送频率配置

![image-20240907133214222](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240907133214222.png)



**注意：这个只对主机有效，从机配置无效**

![image-20240907133621393](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240907133621393.png)

**MSB先行，从高位到低位发送，LSB先行，从低位到高位发送**

#### CR寄存器 （control rigister）

用于配置基本的控制参数 SPI 模式, 波特率 ，LSP，MSB 先行，主从模式，但双向模式等





TXE 置 1 标志有可以有新的数据写入这个寄存器

RXNE 置1 标志数据传输完成，可以取走

![image-20240907140804144](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240907140804144.png)



1. 注意 ： **当想使用接受缓冲区时候，不能只检测RXNE 标志位，因为如果发送缓冲区没有数据，SCK不会产生时钟**
2. BSY 标志： 检测当前线路是否忙碌

 



#### SPI结构体讲解

![image-20240907142215577](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240907142215577.png)

1. SPI_Direction 
   1.  SPI_Direction_2Lines_FullDuplex  双线全双工
   2.  SPI_Direction_2Lines_RxOnly        双线只接受
   3.  SPI_Direction_1Line_RxOnly          单线只接受
   4.  SPI_Direction_1Line_Tx                   单线只发送
2. SPI_Mode
   1. SPI_Mode_Master                            主机模式





### Falsh 的存储特性

![image-20240917102908437](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240917102908437.png)

**注意：写入操作是包括擦除的，都会使得芯片进入忙状态**





**Falsh  的指令集操作**

![QQ20240917-105530](D:\OneDrive\Pictures\QQ20240917-105530.png)





### 工程实现



#### 引脚pei'zhi

![image-20240917122459762](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240917122459762.png)





![屏幕截图 2024-09-17 193600](C:\Users\Administrator\AppData\Local\Packages\Microsoft.ScreenSketch_8wekyb3d8bbwe\TempState\Snips\屏幕截图 2024-09-17 193600.png)