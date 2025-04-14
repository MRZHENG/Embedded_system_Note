## Stm32阶段项目    平衡 小车

### 1. 理论分析

##### 为什么平衡小车能够平衡？

​	为了实现平衡的功能，我们需要对一个没有施加任何外力的小车进行受理分析，这是小车在处于倾斜状态的受力分析

![image-20250304112239286](C:\Users\34718\AppData\Roaming\Typora\typora-user-images\image-20250304112239286.png)

由图可得，小车在没有施加外力的情况下是会向下倒的，我们需要产生一个大于F的且方向相反的力，将小车回到平衡位置，怎么产生这个力呢？接下来我们引入一个生活现象

​	当我们在驾驶汽车的时候，我们猛然加速，我们的上半身会收到惯性影响，会产生惯性力，来试图维持其原本的状态，产生推背感，所以我们可以使用电机，通过电机突然向倒地方向转动，上半身产生相反的惯性力

![image-20250304132523137](C:\Users\34718\AppData\Roaming\Typora\typora-user-images\image-20250304132523137.png)

![image-20250304132609432](C:\Users\34718\AppData\Roaming\Typora\typora-user-images\image-20250304132609432.png)

这就是回复力的公式

​	由于平衡小车摆动幅度很小，就省去三角函数，直接取角度，这个就是作为回复力，同时有了回复力还不够，就像单摆模型一样，如果没有空气阻力，这个单摆就会一直来回震荡，所以我们需要模拟空气阻力，并且加大，来组织他的惯性，





**MPU6050修改代码思路**

1. 将MyIIC 修改成OLED IIC 的底层驱动

2. 新增MPU6050的驱动

   ### IIC底层驱动

   ```c
   #include "stm32f10x.h"
   
   // 当前文件负责编写IIC 软件模拟通用库函数
   // 具体负责 填入引脚和引脚类型，就可以模拟对于的IIC了
   // 可以对于当前库函数进行二次封装
   
   // IIC 写SCL 电平  GPIO8
   void IIC_W_SCL(GPIO_TypeDef * GPIOx,uint16_t GPIO_SCL,uint8_t Bitval){
      GPIO_WriteBit(GPIOx,GPIO_SCL, (BitAction)Bitval);
   }
   
   // IIC 写SDA 电平
   void IIC_W_SDA(GPIO_TypeDef * GPIOx,uint16_t GPIO_SDA,uint8_t Bitval){
      GPIO_WriteBit(GPIOx,GPIO_SDA, (BitAction)Bitval);
   }
   
   // IIC任意引脚的电平 读SDA电平
   uint8_t IIC_R_SDA(GPIO_TypeDef * GPIOx,uint16_t GPIO_SDA){
       uint8_t Bitval = 0;
       Bitval = GPIO_ReadInputDataBit(GPIOx,GPIO_SDA);
       return Bitval;
   }
   
   
   // 模拟软件IIC初始化   考虑的坑，不同的INIT GPIO 引脚是否会覆盖?
   // 参数(外设，引脚编号，SCL,SDA)
   void IIC_Init(uint32_t RCC_PeriphX,GPIO_TypeDef * GPIOx,uint16_t GPIO_SCL_Pin,uint16_t GPIO_SDA_Pin){
       // 开启时钟
       RCC_APB2PeriphClockCmd(RCC_PeriphX,ENABLE);   
   
       //GPIO初始化
       GPIO_InitTypeDef GPIO_InitStructure;    
       GPIO_InitStructure.GPIO_Pin  = GPIO_SCL_Pin | GPIO_SDA_Pin;
       GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP ; // 开漏输出
       GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
       GPIO_Init(GPIOx, &GPIO_InitStructure);
       
       // 设置初始电平 (SDA = 1 , SCL = 1)
       GPIO_SetBits(GPIOx,GPIO_SCL_Pin | GPIO_SDA_Pin);  
   }
   
   
   // 协议层
   
   // IIC 开始信号   // 在SCL高电平时 SDA由高变低
   void IIC_Start(GPIO_TypeDef * GPIOx,uint16_t GPIO_SCL,uint16_t GPIO_SDA){
       IIC_W_SDA(GPIOx,GPIO_SDA,1);
       IIC_W_SCL(GPIOx,GPIO_SCL,1);
   	  IIC_W_SDA(GPIOx,GPIO_SDA,0);
       IIC_W_SCL(GPIOx,GPIO_SCL,0);
   }
   
   // IIC 结束信号   // 在SCL高电平时 SDA由低变高 由于之前SDA是在使用，所以SDA高低电平不确定，需要重新设置
   void IIC_Stop(GPIO_TypeDef * GPIOx,uint16_t GPIO_SCL,uint16_t GPIO_SDA){
   		IIC_W_SDA(GPIOx,GPIO_SDA,0);
       IIC_W_SCL(GPIOx,GPIO_SCL,1);
   	  IIC_W_SDA(GPIOx,GPIO_SDA,1);
   }
   
   // IIC 发送一个字节   // 在SCL高电平时 从机读取SDA电平 // 在SCL低电平时 写入SDA电平
   void IIC_WriteByte(GPIO_TypeDef * GPIOx,uint16_t GPIO_SCL,uint16_t GPIO_SDA,uint8_t Byteval){
      uint8_t i;
      for(i = 0 ; i<8;++i){
         IIC_W_SDA(GPIOx,GPIO_SDA,Byteval & (0x80>>i));
         IIC_W_SCL(GPIOx,GPIO_SCL,1);
   		  IIC_W_SCL(GPIOx,GPIO_SCL,0);
      }
   }
   
   // IIC 接受一个字节 主机需要解除对SDA的控制
   uint8_t IIC_ReadByte(GPIO_TypeDef * GPIOx,uint16_t GPIO_SCL,uint16_t GPIO_SDA){
       uint8_t getVal = 0x00;
       uint8_t i;
       IIC_W_SDA(GPIOx,GPIO_SDA,1);      //解除对SDA的控制
       for(i = 0 ; i<8;++i){
           IIC_W_SCL(GPIOx,GPIO_SCL,1);
           if(IIC_R_SDA(GPIOx,GPIO_SDA) ==1) getVal |= (0x80>>i) ;
           IIC_W_SCL(GPIOx,GPIO_SCL,0);
       }
       return getVal;
   }
   
   // 主机对从机返回ACK  当SCL为低电平的时候 写入SDA电平，高电平读取SDA电平
   void IIC_WriteACK(GPIO_TypeDef * GPIOx,uint16_t GPIO_SCL,uint16_t GPIO_SDA,uint8_t AckBit){
       IIC_W_SDA(GPIOx,GPIO_SDA,AckBit);
       IIC_W_SCL(GPIOx,GPIO_SCL,1);
   	  IIC_W_SCL(GPIOx,GPIO_SCL,0);
   }
   
   // 读取从机返回ACK
   uint8_t IIC_ReadACK(GPIO_TypeDef * GPIOx,uint16_t GPIO_SCL,uint16_t GPIO_SDA){
       uint8_t AckBit;
       IIC_W_SDA(GPIOx,GPIO_SDA,1);  // 解除主机对总线的控制
       IIC_W_SCL(GPIOx,GPIO_SCL,1);
       AckBit = IIC_R_SDA(GPIOx,GPIO_SDA);
       IIC_W_SCL(GPIOx,GPIO_SCL,0);
       return AckBit;
   }
   
   ```

   

```c
#ifndef IIC_SOFTDRIVER
#define IIC_SOFTDRIVER

// IIC 写SCL 电平  GPIO8
void IIC_W_SCL(GPIO_TypeDef * GPIOx,uint16_t GPIO_SCL,uint8_t Bitval);

// IIC 写SDA 电平
void IIC_W_SDA(GPIO_TypeDef * GPIOx,uint16_t GPIO_SDA,uint8_t Bitval);

// IIC任意引脚的电平 读SDA电平
uint8_t IIC_R_SDA(GPIO_TypeDef * GPIOx,uint16_t GPIO_PinX);


// 模拟软件IIC初始化  
// 参数(外设，引脚编号，SCL,SDA)
void IIC_Init(uint32_t RCC_PeriphX,GPIO_TypeDef * GPIOx,uint16_t GPIO_SCL_Pin,uint16_t GPIO_SDA_Pin);

// 协议层

// IIC 开始信号   // 在SCL高电平时 SDA由高变低
void IIC_Start(GPIO_TypeDef * GPIOx,uint16_t GPIO_SCL,uint16_t GPIO_SDA);

// IIC 结束信号   // 在SCL高电平时 SDA由低变高 由于之前SDA是在使用，所以SDA高低电平不确定，需要重新设置
void IIC_Stop(GPIO_TypeDef * GPIOx,uint16_t GPIO_SCL,uint16_t GPIO_SDA);

// IIC 发送一个字节   // 在SCL高电平时 从机读取SDA电平 // 在SCL低电平时 写入SDA电平
void IIC_WriteByte(GPIO_TypeDef * GPIOx,uint16_t GPIO_SCL,uint16_t GPIO_SDA,uint8_t Byteval);

// IIC 接受一个字节 主机需要解除对SDA的控制
uint8_t IIC_ReadByte(GPIO_TypeDef * GPIOx,uint16_t GPIO_SCL,uint16_t GPIO_SDA);

// 主机对从机返回ACK  当SCL为低电平的时候 写入SDA电平，高电平读取SDA电平
void IIC_WriteACK(GPIO_TypeDef * GPIOx,uint16_t GPIO_SCL,uint16_t GPIO_SDA,uint8_t AckBit);

// 读取从机返回ACK
uint8_t IIC_ReadACK(GPIO_TypeDef * GPIOx,uint16_t GPIO_SCL,uint16_t GPIO_SDA);

#endif

```





编码轮数据为1800

![image-20250311144225257](C:\Users\34718\AppData\Roaming\Typora\typora-user-images\image-20250311144225257.png)

直立环

PWMOUT = 7200  angle 为 60 - 10    所以直立环kp应该是一个1 - 1000

kd 应该是 0 -10

速度环

out 数量级是几十  encode 是1800