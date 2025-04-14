**GPIO 的 8种工作模式**

1. 模拟输入（GPIO_Mode_AIN）

   ![image-20240908150957039](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240908150957039.png)

   直接读入IO口的具体电压数值，将电压引入了模拟输入的片上外设

   ![image-20240908150721452](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240908150721452.png)

   **施密特触发器：高于高参考电压输出高电平，低于低参考电压输出低电平，在高参考和低参考电压则保持当前信号不变**

   ![image-20240908150604878](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240908150604878.png)

   

2. 浮空输入（GPIO_Mode_IN_FLOATING）上下电阻都不启用

​        GPIO 内部高阻态，

1. 下拉输入（GPIO_Mode_IPD）下面电阻启用

       输入端在无信号时被拉到低电平，通过连接到 GND 的电阻。

2. 上拉输入（GPIO_Mode_IPU）上面电阻启用   

      输入端在无信号时被拉到高电平，通过连接到 Vcc 的电阻。

3. 开漏输出（GPIO_Mode_Out_OD）

    如果通过库控制输出高电平则N-MOS断开，I/O内部出于高阻态（断路）

    如果控制IO口输出低电平，则N-Mos激活，v-ss接通

    ![image-20240908145752464](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240908145752464.png)

​    4. 推挽输出（GPIO_Mode_Out_PP)

​      **当前模式下P-mos与N-mos协同工作，输出高电平 p-mos激活 输出低电平 n-mos 激活**

![image-20240908145723125](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240908145723125.png)

向外部输出 0v 低电平或者 3.3v 高电平  

![image-20240907161435957](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240907161435957.png)

![image-20240908150412609](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240908150412609.png)

5. 复用开漏输出（GPIO_Mode_AF_OD）

6. 复用推挽输出（GPIO_Mode_AF_PP）](https://cloud.tencent.com/developer/article/1893090)