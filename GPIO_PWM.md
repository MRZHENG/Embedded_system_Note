**STM32**的GPIO介绍

STM32引脚说明



GPIO是通用输入/输出端口的简称，是STM32可控制的引脚。GPIO的引脚与外部硬件设备连接，可实现与外部通讯、控制外部硬件或者采集外部硬件数据的功能。

STM32F103ZET6芯片为144脚芯片，包括7个通用目的的输入/输出口（GPIO）组，分别为GPIOA、GPIOB、GPIOC、GPIOD、GPIOE、GPIOF、GPIOG，同时每组GPIO口组有16个GPIO口。通常简略称为PAx、PBx、PCx、PDx、PEx、PFx、PGx，其中x为0-15。

STM32的大部分引脚除了当GPIO使用之外，还可以复用位外设功能引脚（比如串口），这部分在【STM32】STM32端口复用和重映射（AFIO辅助功能时钟） 中有详细的介绍。

GPIO基本结构
每个GPIO内部都有这样的一个电路结构，这个结构在本文下面会具体介绍。



这边的电路图稍微提一下：

保护二极管：IO引脚STM32的GPIO介绍

#### STM32引脚说明

GPIO是通用输入/输出端口的简称，是STM32可控制的引脚。GPIO的引脚与外部硬件设备连接，可实现与外部通讯、控制外部硬件或者采集外部硬件数据的功能。

STM32F103ZET6芯片为**144脚芯片，包括7个通用目的的输入/输出口（GPIO）组，分别为GPIOA、GPIOB、GPIOC、GPIOD、GPIOE、GPIOF、GPIOG，同时每组GPIO口组有16个GPIO口。通常简略称为PAx、PBx、PCx、PDx、PEx、PFx、PGx，其中x为0-15。**

STM32的大部分引脚除了当GPIO使用之外，还可以复用位外设功能引脚（比如串口），这部分在[【STM32】STM32端口复用和重映射（AFIO辅助功能时钟） ](https://blog.csdn.net/qq_38410730/article/details/79828852)中有详细的介绍。

#### GPIO基本结构

每个GPIO内部都有这样的一个电路结构，这个结构在本文下面会具体介绍。

![img](https://i-blog.csdnimg.cn/blog_migrate/81a1fd18584dfbe4762ea18bf7df201d.png)

这边的电路图稍微提一下：

- **保护二极管：IO引脚上下两边两个二极管用于防止引脚外部过高、过低的电压输入。**当引脚电压高于VDD时，上方的二极管导通；当引脚电压低于VSS时，下方的二极管导通，防止不正常电压引入芯片导致芯片烧毁。但是尽管如此，还是不能直接外接大功率器件，须加大功率及隔离电路驱动，防止烧坏芯片或者外接器件无法正常工作。
- **P-MOS管和N-MOS管：由P-MOS管和N-MOS管组成的单元电路使得GPIO具有“推挽输出”和“开漏输出”的模式**。这里的电路会在下面很详细地分析到。
- **TTL肖特基触发器：信号经过触发器后，模拟信号转化为0和1的数字信号。但是，当GPIO引脚作为ADC采集电压的输入通道时，用其“模拟输入”功能，此时信号不再经过触发器进行TTL电平转换。**ADC外设要采集到的原始的模拟信号。

这里需要注意的是，在查看《STM32中文参考手册V10》中的GPIO的表格时，会看到**有“FT”一列，这代表着这个GPIO口时兼容3.3V和5V的；如果没有标注“FT”，就代表着不兼容5V**。

 

### STM32的GPIO工作方式

GPIO支持4种输入模式（浮空输入、上拉输入、下拉输入、模拟输入）和4种输出模式（开漏输出、开漏复用输出、推挽输出、推挽复用输出）。同时，GPIO还支持三种最大翻转速度（2MHz、10MHz、50MHz）。

每个I/O口可以自由编程，但I/O口寄存器必须按32位字被访问。

1. GPIO_Mode_AIN 模拟输入
2. GPIO_Mode_IN_FLOATING 浮空输入
3. GPIO_Mode_IPD 下拉输入
4. GPIO_Mode_IPU 上拉输入
5. GPIO_Mode_Out_OD 开漏输出
6. GPIO_Mode_Out_PP 推挽输出
7. GPIO_Mode_AF_OD 复用开漏输出
8. GPIO_Mode_AF_PP 复用推挽输出

下面将具体介绍GPIO的这八种工作方式：

#### 浮空输入模式

![img](https://i-blog.csdnimg.cn/blog_migrate/da2cc20f333d505a3017faf90253fcee.png)

**浮空输入模式下，I/O端口的电平信号直接进入输入数据寄存器。也就是说，I/O的电平状态是不确定的，完全由外部输入决定；如果在该引脚悬空（在无信号输入）的情况下，读取该端口的电平是不确定的。**

#### 上拉输入模式

![img](https://i-blog.csdnimg.cn/blog_migrate/d1cde85b72e72c6a3ca3db6e8efb77ed.png)

**上拉输入模式下，I/O端口的电平信号直接进入输入数据寄存器。但是在I/O端口悬空（在无信号输入）的情况下，输入端的电平可以保持在高电平；并且在I/O端口输入为低电平的时候，输入端的电平也还是低电平。**

#### 下拉输入模式

![img](https://i-blog.csdnimg.cn/blog_migrate/242454c850efcf9facd3599d39940bc4.png)

**下拉输入模式下，I/O端口的电平信号直接进入输入数据寄存器。但是在I/O端口悬空（在无信号输入）的情况下，输入端的电平可以保持在低电平；并且在I/O端口输入为高电平的时候，输入端的电平也还是高电平。**

#### 模拟输入模式

![img](https://i-blog.csdnimg.cn/blog_migrate/a9ef8e9e4c1212b4518f712beeaabfda.png)

模拟输入模式下，I/O端口的模拟信号（电压信号，而非电平信号）直接模拟输入到片上外设模块，比如ADC模块等等。

#### 开漏输出模式

![img](https://i-blog.csdnimg.cn/blog_migrate/149f91e4643d2d6acb2f5f15836338c0.png)

开漏输出模式下，通过设置位设置/清除寄存器或者输出数据寄存器的值，途经N-MOS管，最终输出到I/O端口。这里要**注意N-MOS管，当设置输出的值为高电平的时候，N-MOS管处于关闭状态，此时I/O端口的电平就不会由输出的高低电平决定，而是由I/O端口外部的上拉或者下拉决定；当设置输出的值为低电平的时候，N-MOS管处于开启状态，此时I/O端口的电平就是低电平。**同时，I/O端口的电平也可以通过输入电路进行读取；注意，I/O端口的电平不一定是输出的电平。

#### 开漏复用输出模式

![img](https://i-blog.csdnimg.cn/blog_migrate/13fec9b33a866e1ff61f9c2170772f46.png)

开漏复用输出模式，与开漏输出模式很是类似。只是输出的高低电平的来源，不是让CPU直接写输出数据寄存器，取而代之利用片上外设模块的复用功能输出来决定的。

#### 推挽输出模式

![img](https://i-blog.csdnimg.cn/blog_migrate/411bad6f6f6b2866e435a7e74a13c6c6.png)

推挽输出模式下，通过设置位设置/清除寄存器或者输出数据寄存器的值，途经P-MOS管和N-MOS管，最终输出到I/O端口。这里要**注意P-MOS管和N-MOS管，当设置输出的值为高电平的时候，P-MOS管处于开启状态，N-MOS管处于关闭状态，此时I/O端口的电平就由P-MOS管决定：高电平；当设置输出的值为低电平的时候，P-MOS管处于关闭状态，N-MOS管处于开启状态，此时I/O端口的电平就由N-MOS管决定：低电平。**同时，I/O端口的电平也可以通过输入电路进行读取；注意，此时I/O端口的电平一定是输出的电平。

#### 推挽复用输出模式

![img](https://i-blog.csdnimg.cn/blog_migrate/a0129d3001af3c2e5a3c945e0ad7c394.png)

推挽复用输出模式，与推挽输出模式很是类似。只是输出的高低电平的来源，不是让CPU直接写输出数据寄存器，取而代之利用片上外设模块的复用功能输出来决定的。上下两边两个二极管用于防止引脚外部过高、过低的电压输入。当引脚电压高于VDD时，上方的二极管导通；当引脚电压低于VSS时，下方的二极管导通，防止不正常电压引入芯片导致芯片烧毁。但是尽管如此，还是不能直接外接大功率器件，须加大功率及隔离电路驱动，防止烧坏芯片或者外接器件无法正常工作。
P-MOS管和N-MOS管：由P-MOS管和N-MOS管组成的单元电路使得GPIO具有“推挽输出”和“开漏输出”的模式。这里的电路会在下面很详细地分析到。
TTL肖特基触发器：信号经过触发器后，模拟信号转化为0和1的数字信号。但是，当GPIO引脚作为ADC采集电压的输入通道时，用其“模拟输入”功能，此时信号不再经过触发器进行TTL电平转换。ADC外设要采集到的原始的模拟信号。
这里需要注意的是，在查看《STM32中文参考手册V10》中的GPIO的表格时，会看到有“FT”一列，这代表着这个GPIO口时兼容3.3V和5V的；如果没有标注“FT”，就代表着不兼容5V。

 

### 以下是需要记忆的，上面的原理了解一下即可

STM32的GPIO工作方式
GPIO支持4种输入模式（浮空输入、上拉输入、下拉输入、模拟输入）和4种输出模式（开漏输出、开漏复用输出、推挽输出、推挽复用输出）。同时，GPIO还支持三种最大翻转速度（2MHz、10MHz、50MHz）。

每个I/O口可以自由编程，但I/O口寄存器必须按32位字被访问。

GPIO_Mode_AIN 模拟输入
GPIO_Mode_IN_FLOATING 浮空输入
GPIO_Mode_IPD 下拉输入
GPIO_Mode_IPU 上拉输入
GPIO_Mode_Out_OD 开漏输出
GPIO_Mode_Out_PP 推挽输出
GPIO_Mode_AF_OD 复用开漏输出
GPIO_Mode_AF_PP 复用推挽输出
下面将具体介绍GPIO的这八种工作方式：

1. **浮空输入模式(直接接受外部的电平信号)**


​     浮空输入模式下，I/O端口的电平信号直接进入输入数据寄存器。也就是说，I/O的电平状态是不确定的，完全由外部输入决定；如果在该引脚悬空（在无信号输入）的情况下，读取该端口的电平是不确定的。

2. **上拉输入模式（外部无信号默认高电平，外部高电平信号为高电平，低电平为低电平）**


​    上拉输入模式下，I/O端口的电平信号直接进入输入数据寄存器。但是在I/O端口悬空（在无信号输入）的情况下，输入端的电平可以保持在高电平；并且在I/O端口输入为低电平的时候，输入端的电平也还是低电平。

3. **下拉输入模式 （外部无信号默认低电平，外部高电平信号还是高电平，低电平还是低电平）**


​     下拉输入模式下，I/O端口的电平信号直接进入输入数据寄存器。但是在I/O端口悬空（在无信号输入）的情况下，输入端的电平可以保持在低电平；并且在I/O端口输入为高电平的时候，输入端的电平也还是高电平。

4. **模拟输入模式（ 将外部信号不导入芯片内部而是直接输入到外设）**


​     模拟输入模式下，I/O端口的模拟信号（电压信号，而非电平信号）直接模拟输入到片上外设模块，比如ADC模块等等。

5. **开漏输出模式（只能输出低电平，如果是高电平就相当于断路（也叫高阻态））**


​      开漏输出模式下，通过设置位设置/清除寄存器或者输出数据寄存器的值，途经N-MOS管，最终输出到I/O端口。这里要注意N-MOS管，当设置输出的值为高电平的时候，N-MOS管处于关闭状态，此时I/O端口的电平就不会由输出的高低电平决定，而是由I/O端口外部的上拉或者下拉决定；当设置输出的值为低电平的时候，N-MOS管处于开启状态，此时I/O端口的电平就是低电平。同时，I/O端口的电平也可以通过输入电路进行读取；注意，I/O端口的电平不一定是输出的电平。

6. **开漏复用输出模式 其实就是片上外设来提供信号**


​      开漏复用输出模式，与开漏输出模式很是类似。只是输出的高低电平的来源，不是让CPU直接写输出数据寄存器，取而代之利用片上外设模块的复用功能输出来决定的。

7. **推挽输出模式（既可以输出高电平也可以输出低电平）**


​      推挽输出模式下，通过设置位设置/清除寄存器或者输出数据寄存器的值，途经P-MOS管和N-MOS管，最终输出到I/O端口。这里要注意P-MOS管和N-MOS管，当设置输出的值为高电平的时候，P-MOS管处于开启状态，N-MOS管处于关闭状态，此时I/O端口的电平就由P-MOS管决定：高电平；当设置输出的值为低电平的时候，P-MOS管处于关闭状态，N-MOS管处于开启状态，此时I/O端口的电平就由N-MOS管决定：低电平。同时，I/O端口的电平也可以通过输入电路进行读取；注意，此时I/O端口的电平一定是输出的电平。

8. **推挽复用输出模式**

​     推挽复用输出模式，与推挽输出模式很是类似。只是输出的高低电平的来源，不是让CPU直接写输出数据寄存器，取而代之利用片上外设模块的复用功能输出来决定的。


### 关于Tim

​      首先我们需要想一想这个led呼吸灯为什么会产生呼吸一样的效果，**是因为在不同电压下led灯的亮度也不同**。所以我们需要在一个周期内，调控高低电平的比例，就能模拟出对应的电压，也就能产生led流水灯 的现象了

![image-20241011193049768](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241011193049768.png)

如图所示：高电平占周期时间的份额越大，那么输出的电压也就会越高，这里我们就需要采用TIM的输出比较模式来实现PWM方波信号了

​      接下来我来简单阐述一下原理，RCC时钟的频率到TIm上，TIM 存在一个预分频器，对这个频率分频，这个就是TIM外设的工作频率了，TIM内部存在一个时基单元（**可以简单理解成一个有点高级的计数器，这个计数器会每次经过一个周期，加一或者减一。**

​    然后我们肯定不能让他们一直增加或者递减，所以就设置了一个界限（**自动重载寄存器**），我们会对这个自动重装载寄存器设定一个值，**当我们计数器递增到这个值的时候，计数器就会归零（这里讨论是递增模式，如果是递减模式，则计数器会在我们设定的这个界限下开始递减，递减到0 就 恢复成 我们最开始设定 的值 ）** 

​        ps(其实原理这些我们了解即可，知道有这么个东西，然后通过 不断的去实践，我们就能熟悉这整个工作流程了) 。

​    接下来，在我们设置的这个 0 - 界限值 这个范围内，我们还可以设置一个界限（CCR），可以实现当计数器小于这个界限时候，输出高电平，大于这个界限的时候，输出低电平，

​    然后我们在最后程序里，不断调用这个 **设置CCR的操作**  的函数 ，不就达到了 设置 不同电压的目的了嘛！！！



#### 以下是原理图

1. **时基单元**  

![image-20241011194834584](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241011194834584.png)

​       **图中 U 和 UI 代表 更新事件和中断**，也就是当计数器到达界限的时候，会触发一次中断和跟新事件



2. **TIM的时钟来源**

   ![image-20241011195123097](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241011195123097.png)



![image-20241011195134240](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241011195134240.png)

![image-20241011195151901](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241011195151901.png)

3. TIM 输出比较 和输入捕获模式 （这里PWM采用输出比较模式,因为要输出电平信号嘛）

![image-20241011195325320](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241011195325320.png)





