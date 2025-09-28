### 1.什么是TaskControlBlock(TCB)?

​	TCB结构体，其实就是在堆空间中，申请一段空间作为栈，还包含了任务名字，这个任务的状态，以及栈的地址，和栈顶指针的一个结构体

```c

/*任务控制块声明
  1.包涵栈顶指针
  2.任务节点
  3.任务栈起始地址
  4.任务名字
*/
typedef struct tskTaskControlBlock
{
    volatile StackType_t *pxTopOfStack;   /*任务栈顶指针*/

    ListItem_t  xStateListItem;     /*任务节点(不是链表类型)*/
    
    StackType_t          *pxStack;   /*任务栈起始地址 (不是栈底，只是规定了这一块区域属于栈)*/

    char pcTaskName[configMAX_TASK_NAME_LEN]; /*任务名字*/
} tskTCB;

typedef tskTCB TCB_t;
```

| TCB_t包含的元素 |                              |      |
| --------------- | ---------------------------- | ---- |
| pxTopOfStack    | 指向任务栈顶指针             |      |
| xStateListItem  | 任务节点(挂在任务就绪列表等) |      |
| pxStack         | 指向任务栈的内存块           |      |
| pcTaskName      | 任务名字                     |      |

---



### 2. 新建一个任务发生了什么？

​          我们来思考一下对于创建一个任务，我们应该要做什么，我们需要对其TCB结构体经行初始化操作，然后返回这个TCB结构体的句柄指针

#### 2.1 xTaskCreateStatic

```c++
/*任务创建函数*/
#if( configSUPPORT_STATIC_ALLOCATTION ==1 )
/// @brief 如果开启了静态分配创建任务的开关，则可以调用这个函数创建静态任务
/// @param pxTaskCode     任务入口函数
/// @param pcName         任务名称 字符串形式
/// @param ulStackDepth   任务栈大小，单位为字
/// @param pvParameters   任务形参
/// @param puxStackBuffer 任务栈的起始地址
/// @param pxTaskBuffer   TCB_T 结构体的指针
/// @return TaskHandle_t  返回任务类指针
/// @details  该函数新建了两个临时变量，指向TCB控制块，以及一个任务句柄，对控制块的栈指针进行赋值，
///          然后调用prvInitialiseNewTask函数进行初始化，最后返回任务句柄。
TaskHandle_t xTaskCreateStatic( TaskFunction_t pxTaskCode,  
                                const char *const pcName,
                                const uint32_t ulStackDepth,
                                void * const  pvParameters,
                                StackType_t *const puxStackBuffer,
                                TCB_t * const pxTaskBuffer)
{
    TCB_t *pxNewTCB;
    TaskHandle_t xReturn;/*临时变量,用于返回任务句柄*/

    /*检查参数合法性 看两个buffer是否都不为空*/
    if(( pxTaskBuffer != NULL ) && ( puxStackBuffer != NULL ))
    {
        pxNewTCB = (TCB_t *) pxTaskBuffer;
        pxNewTCB->pxStack = (StackType_t *) puxStackBuffer;  //任务栈底 指向puxStackBuffer
        
        /*创建新的任务,会将TCB结构体初始化，并对传入的句柄进行初始化操作*/
        prvInitialiseNewTask( pxTaskCode,   /*任务入口函数*/
                              pcName,       /*任务名称 字符串形式*/
                              ulStackDepth, /*任务栈大小，单位为字*/
                              pvParameters, /*任务形参*/
                              &xReturn,     /*任务句柄*/
                              pxNewTCB      /*任务指针*/
                            );
    }else{
        xReturn = NULL;
    }
    return xReturn;
}
#endif /* configSUPPORT_STATIC_ALLOCATTION */

```

1. configSUPPORT_STATIC_ALLOCATTION是一个宏定义开关，意为是否允许开启任务静态创建函数
2. 该函数新建了两个临时变量，指向TCB控制块的指针，以及一个任务句柄，对控制块的栈指针进行赋值， 然后调用**prvInitialiseNewTask**函数进行初始化，也就是对**TCB**结构体填充内容，以及在堆内设置一些ARM内核规定的任务栈设置方式，最后返回**任务句柄**。
3. 任务的栈，任务的函数实体，任务控制块都需联系起来，这个联系工作就是由任务创建函数实现
4. 返回的 **TaskHandle_t** 实际上就是一个  **void*** 的指针



#### 2.2 prvInitialiseNewTask(xTaskCreateStatic调用)

```c
///@brief    私有初始化任务函数
///@param    pxTaskCode    任务入口函数
///@param    pcName        任务名称 字符串形式
///@param    ulStackDepth  任务栈大小，单位为字
///@param    pvParameters  任务形参
///@param    pxCreatedTask 任务句柄(用于指向任务控制块)
///@param    pxNewTCB      任务控制块指针
///@details  该函数用于初始化任务控制块，设置任务栈，设置任务名称
static void prvInitialiseNewTask(TaskFunction_t pxTaskCode,
                                 const char *const pcName,
                                 const uint32_t ulStackDepth,
                                 void * const pvParameters,
                                 TaskHandle_t *const pxCreatedTask,
                                 TCB_t *pxNewTCB)  
{
    StackType_t *pxTopOfStack;
    UBaseType_t x;

    /*获取栈顶地址*/
    pxTopOfStack = pxNewTCB->pxStack + (ulStackDepth - (uint32_t)1);
    /*向下做8字节对齐 例子假设是 57字节 那么经过换算后就是56字节*/
    pxTopOfStack = ((StackType_t *)\
                    (((uint32_t)pxTopOfStack) & (~( (uint32_t)0x0007 )))); 
                    
    /*将任务的名字存储在TCB中
       那么就会有两种情况 
       1.任务名称长度小于等于configMAX_TASK_NAME_LEN 
       2.任务名称长度大于 configMAX_TASK_NAME_LEN (需要做截断处理)
    */
    for( x = (UBaseType_t) 0; x < configMAX_TASK_NAME_LEN; x++ ){
        pxNewTCB->pcTaskName[x] = pcName[x];
        if(pcName[x] == 0x00){
            break;
        }
    }
    /*任务名字长度不能超过 configMAX_TASK_NAME_LEN 这种是截断情况
      打印字符是遇到第一个\0 就会停止，有多个 \0 的话，一个 \0 后面的字符会丢失*/ 
    pxNewTCB->pcTaskName[configMAX_TASK_NAME_LEN - 1] = '\0';

    /*初始化TCB结构体的 xStateListItem 节点*/
    vListInitialiseItem(& pxNewTCB->xStateListItem);
    /*设置xStateListItem的拥有者*/
    listSET_LIST_ITEM_OWNER(&pxNewTCB->xStateListItem,pxNewTCB);

    /*初始化任务栈,调用硬件底层接口*/  
    pxNewTCB->pxTopOfStack = pxPortInitialiseStack(
                            pxTopOfStack, /*任务栈顶指针*/
                            pxTaskCode,   /*任务入口函数*/
                            pvParameters  /*任务形参*/
                            );
    
    /*将任务句柄指向 任务控制块中*/
    if( (void*) pxCreatedTask != NULL ){
        *pxCreatedTask = (TaskHandle_t) pxNewTCB;
    }

}
```

**描述:**

这个函数由 **xTaskCreateStatic** 调用，也就是对传入的TCB任务控制块的指针所指向的控制块进行了赋值，初始化操作，也就是干了4件事情

1. 找到还没有对任务栈初始化的栈顶指针
2. 对任务名字经行了赋值
3. 初始化了xStateListItem，任务节点，让他当前还不属于任何一个就绪列表
4. **调用了底层的pxPortInitialiseStack 来初始化这个任务控制块的堆栈，最后返回其栈顶指针，然后对TCB的栈顶指针进行了赋值操作**
5. 最后将传入的 **TaskHandle_t *** 类型，也就是指向任务句柄的指针解引用，使得任务句柄赋值为 **TCB_t *** 类型的任务控制块指针



#### 2.3 pxPortInitialiseStack(由prvInitialiseNewTask调用来初始化栈)

```c
/// @brief 任务栈初始化函数，返回栈顶指针
/// @param pxTopOfStack   栈顶指针 
/// @param pxCode         任务入口函数
/// @param pvParameters   任务入口函数的参数
/// @return 栈顶指针
/// @details 对任务栈进行初始化操作
StackType_t *pxPortInitialiseStack( StackType_t* pxTopOfStack,
                                    TaskFunction_t pxCode,
                                    void* pvParameters)
{
    /*异常发生时，自动加载到CPU寄存器的内容*/
    pxTopOfStack--;
    *pxTopOfStack = portINITIAL_XPSR; /*xPSR 的 Bit24 位为1 为Thumb模式*/
    pxTopOfStack--;
    *pxTopOfStack = ((StackType_t) pxCode) & portSTART_ADDRESS_MASK; /*PC 任务入口函数*/
    pxTopOfStack--;
    *pxTopOfStack = (StackType_t)prvTaskExitError; /*LR: 任务退出返回地址,但是一般不会退出*/
    pxTopOfStack-= 5; /*R12, R3,R2 ,R1 默认初始化为 0*/
    *pxTopOfStack = (StackType_t)pvParameters; /*R0: 函数参数*/

    /*异常发生时，手动加载到CPU寄存器的内容*/
    pxTopOfStack-=8; /*R11,R10,R9,R8,R7,R6,R5,R4 默认初始化为0*/

    /*返回栈顶指针，此时pxTopOfStack 指向空闲的栈空间*/
    return pxTopOfStack;
}                                    


```

具体这段代码干了什么如图所示

![image-20250513105849263](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250513105849263.png)

**为什么寄存器都是 0？**

因为这里只是任务创建，任务还没有运行，所以是0



#### 总结：

 **关于创建任务**，流程如下

1.  调用 **xTaskCreateStatic** ，检查两个buffer的合法性，生成一个指向 TCB_t的指针，以及一个用于指向 TCB结构体的  **TaskHandle** ,然后调用 **prvInitialiseNewTask** 去对任务块初始化
2. 进入 **prvInitialiseNewTask**，对 **TCB** 的成员进行初始化，其中对于 **pxTopOfStack(栈顶指针)** 初始化，调用了 **接口函数pxPortInitialiseStack**
3. 进入 **pxPortInitialiseStack**，将寄存器，任务形参，压入栈中,更新 **pxTopOfStack** 返回
4. 最后 **prvInitialiseNewTask**  把 **pxTopOfStack**更新后，将 传入的句柄指针(**TaskHandle_t ***)，解引用后指向 **TCB**控制控制块



**关于任务第一次运行**

​          任务第一次运行时，就是从这个栈指针开始手动加载8个字的内容到CPU寄存器：R4、R5、R6、R7、 R8、R9、R10和R11，当退出异常时，栈中剩下的8个字的内容会自动加载到CPU寄存器： R0、R1、R2、R3、R12、R14、R15和xPSR的位24。此时PC指针就指向了任务入口地址， 从而成功跳转到第一个任务



---





### 3. 关于任务优先级

任务创建好后，我们需要将任务添加到就绪列表中

就绪列表的组成？

![image-20250513115807047](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250513115807047.png)

就绪列表实际就是一个类型 为 **List_t**类型的数组，**数组的下标就是代表在这个下标的存储的元素的任务优先级**， 这数组的大小，由决定 **任务最大优先级的宏 configMAX_PRIORITIES** 决定，同一个优先级的任务统一插入到就绪列表的同一条链表中

![image-20250513120209404](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250513120209404.png)



------



### 4. 启动第一个任务

#### 4.1实现调度器

​	调度器的功能：**负责任务切换，具体是在就绪列表中找到优先级最高的任务然后再去执行该任务从代码上来看，调度器无非也就是由几个全局变 量和一些可以实现任务切换的函数组成，全部都在task.c文件中实现。**  



#### 4.2 vTaskStartScheduler()函数

![image-20250513150254932](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250513150254932.png)

     1. 由于当前还是简单的实现，还不支持查找最高优先级，所以先手动指定一个需要运行的任务
     1. 调用 函数**xPortStartScheduler** 启动调度器，**调度器启动成功，则不会返回，这个函数在port.c中实现**



#### 4.3 xPortStartScheduler()函数(由vTaskStartScheduler调用)

```C
BaseType_t xPortStartScheduler(void)
{
    /*配置PendSV 和 SysTick 的中断优先级为最低*/
    portNVIC_SYSPRI2_REG |= portNVIC_PENDSV_PRI;
    portNVIC_SYSPRI2_REG |= portNVIC_SYSTICK_PRI;

    /*启动第一个任务后，不再返回*/
    prvStartFirstTask();

    /*不应该运行到这*/
    return 0;
}

```

     1. 配置PendSV 和 SysTick 的中断优先级为最低。**SysTick和 PendSV都会涉及到系统调度，系统调度的优先级要低于系统的其它硬件中断优先级**，即优 先相应系统中的外部硬件中断，所以SysTick和PendSV的中断优先级配置为最低
     1. 调用 prvStartFirstTask() 来启动第一个 任务，启动完成后，不再返回，**这个函数由汇编编写**



#### 4.4 prvStartFirstTask()函数(由xPortStartScheduler()函数调用)

描述：**prvStartFirstTask**这个函数用于开启第一个任务，**这个任务做了两件事情**

1. 更新 MSP (**主栈指针**) 的值
2. 产生 **SVC** 系统调用 （SVC 异常类型的一种）**会去执行 SVC 中的中断服务函数**

```c
/* 
   * 参考资料《STM32F10xxx Cortex-M3 programming manual》4.4.3，百度搜索“PM0056”即可找到这个文档 
   * 在Cortex-M中，内核外设SCB的地址范围为：0xE000ED00-0xE000ED3F 
   * 0xE000ED008为SCB外设中SCB_VTOR这个寄存器的地址，里面存放的是向量表的起始地址，即MSP的地址 
   */ 
__asm void prvStartFirstTask(void)
{
    PRESERVE8
		/*在 Cortex-M 中，0xE000ED08 是 SCB_VTOR 这个寄存器的地址
			里面存放的是中断向量表的起始地址，即MSP的地址*/
		ldr r0, =0xE000ED08    /*寄存器地址存储到 R0*/
		ldr r0, [r0]		   /*内容有 为 0x00000000,MSP指针的地址*/
		ldr r0, [r0]		   /*将主栈指针指向的值，也就是0x00000000的内容读取出来*/
	
		/*设置主堆栈指针 msp的地址*/
		msr msp , r0
	
		/*使能全局中断和异常*/
		cpsie i			
		cpsie f
		dsb             /*它的作用是确保在 dsb 指令执行之前的所有数据访问（包括读写操作）都完成之后，才允许继续执行后面的指令。*/
		isb             /*确保前面的指令已经完成以及所有流水线都刷新*/
	
		/*调用 SVC 去启动第一个任务 也就是SVC_Handler*/
		svc 0
		nop
		nop
}

```

关于CPS指令用法

```assembly
CPSID I ;PRIMASK=1     ;关中断 
CPSIE I ;PRIMASK=0     ;开中断 
CPSID F ;FAULTMASK=1   ;关异常 
CPSIE F ;FAULTMASK=0   ;开异常 
```

描述 ：
       更新 Msp 的值，开启全局中断和异常，调用SVC 去启动第一个任务

#### 4.5 vPortSVCHandler 函数（由 prvStartFirstTask 调用）

```assembly
/*这个由 portYIELD() 实现*/
__asm void vPortSVCHandler(void)
{
    extern pxCurrentTCB;

    PRESERVE8

    ldr r3 , =pxCurrentTCB  /*将 pxCurrentTCB的地址存入 R3*/
    ldr r1 ,[r3]            /*加载pxCurrentTCB到r1*/
    ldr r0 ,[r1]            /*加载pxCurrentTCB的内容，指向的任务控制块到 r0，任务控制块的第一个
                              成员就是栈顶指针，所以此时r0等于栈顶指针。*/
    ldmia r0!, {r4-r11}     /*以r0为基地址，将栈中向上增长的8个字的内容加载到CPU寄存
                              器r4~r11，同时r0也会跟着自增。 */
    msr psp , r0            /*将任务栈指针传递给PSP */
    isb                     /*确保前面的指令已经完成以及所有流水线都刷新*/
    mov r0 ,#0              /*将R0设置为0*/
    msr basepri, r0         /*打开所有中断，关于这个寄存器见 ARM 架构篇 */
    orr r14,#0xd            /*当从SVC中断服务退出前，通过向r14寄存器最后4位按位或上0x0D，使得硬件在退出时使用进程堆栈指针PSP完成出栈操作并返回后进入任务模式、回Thumb状态。在SVC中断服务里面，使用的是MSP堆栈指针，是处在ARM状态。*/
    bx  r14                 /*异常返回，此时使用的是链接寄存器*/
}
```

描述：

1. 这个函数读取当前**pxCurrentTCB**的指向的的 **TCB** （加载pxCurrentTCB指向的任务控制块到 r0，任务控制块的第一个 成员就是栈顶指针，所以此时r0等于栈顶指针）

2. 将 R4 - R11 寄存器 8 个字的内容，**手动加载到 CPU** ，同时 R0 也会自动自增

   ![image-20250513160135873](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250513160135873.png)

3. 再将改变后的R0，也就是新的栈顶指针刷新到PSP当中，**也就是上图指向R0的位置**

4. 打开所有中断

5. 从SVC中断服务退出前，通过向r14**寄存器**最后4位按位或上 0x0D，**使得硬件在退出时使用进程堆栈指针PSP完成出栈操作并返回后进入任务模式、返 回Thumb状态。在SVC中断服务里面，使用的是MSP堆栈指针，是处在ARM状态**

6. 最后，异常返回，由于已经对 **R14** 做了操作，切换指针 为 **PSP**，**自动将栈中的剩下 内容加载到CPU寄存器： xPSR，PC（任务入口地址），R14，R12，R3，R2，R1，R0 （任务的形参）同时PSP的值也将更新**



#### 4.6 总结

启动一个任务的流程

1. 先调用**vTaskStartScheduler** ，再调用位于 port.c中的 接口 **xPortStartScheduler**
2. **xPortStartScheduler** 设置了 SysTick 和 PendSV 优先级，然后调用 **prvStartFirstTask** 开启第一个任务
3. **更新 Msp 的值，开启全局中断和异常**，调用**SVC** 去**启动第一个任务**，前往 **vPortSVCHandler**
4. 在  **vPortSVCHandle** 中 恢复 所有的存入的寄存器加载到CPU![image-20250513162058172](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250513162058172.png)
5. 然后 对 **R14** 进行赋值,切换内核模式，将指针切换 成 **PSP**



---



### 5.任务切换

#### 5.1 任务切换函数 taskYIELD()

```C
/*portmacro.h定义*/
 /* 中断控制状态寄存器：0xe000ed04 
 * Bit 28 PENDSVSET: PendSV 悬起位 
 */ 
#define portNVIC_INT_CTRL_REG         (*((volatile uint32_t*)(0xE000ED04)))
#define portNVIC_PENDSVSETT_BIT       (1UL << 28UL)

#define portSY_FULL_READ_WRITE        (15)

/* 将PendS 的悬起位置置1，在没有其他中断发生的情况下，触发PendSV中断，去执行PendSV 服务函数*/
#define portYIELD()\
{\
 /*触发PendSV 中断实现上下文切换*/\
 portNVIC_INT_CTRL_REG = portNVIC_PENDSVSETT_BIT;\
 __dsb(portSY_FULL_READ_WRITE);\
 __isb(portSY_FULL_READ_WRITE);\
}


/*task.h定义*/
#define taskYIELD()   portYIELD()

```

**描述：**

portYIELD 的 实现很简单，实际就是将 **PendSV的悬起位置 置1 ** 在没有其他中断响应的时候，去运行 **PendSV 中断** 去执行 写好的 **PendSV 中断服务函数**



#### 5.2  xPortPendSVHandler()函数 (PendSV 中断服务函数)

代码如图所示

![image-20250513165754982](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250513165754982.png)

**描述** 

#### 5.2.1 保存上文

当 进入 **PendSV 中断**， **上一个任务 运行的环境即： xPSR，PC（任务入口地址），R14，R12，R3，R2，R1，R0（任务的形 参）这些CPU寄存器的值会自动存储到任务的栈中**，PSP 的值也就跟着跟新了 此时 PSP  指向如图所示

![image-20250513165418206](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250513165418206.png)

将**当前的PSP 保存到 R0 寄存器中** ,加载 **pxCurrentTCB**的地址到 R3，将 **pxCurrentTCB** 的内容，也就是 **pxCurrentTCB所指向的 TCB 的地址** 赋值给 R2 ，**然后 R0 向下递减 存储 R4 -R11 的内容 ， 此时 R0 就是当前任务栈顶指针 ， 再将 R0 赋值 给 R2**

#### 5.2.2 切换下文

将 **R3和R14 临时压入堆栈** ( 在 CortexM3-M4 中， 整个系统的所有中断都是 使用的 主栈指针 **MSP**)，

**为什么要压入 R3 和 R14？**

1. 因为接下来要调用 **上下文切换函数 vTaskSwitchContext** , 返回地址会保存到 R14 中，会被覆盖，而且 **在PendSV 中断服务函数执行完毕 后，返回时，会根据 R14 来决定 返回处理器模式还是任务模式，出栈时用的是 PSP 还是 MSP ，所以需要入栈保护** 
2. 至于 R3 ，因为他指向的 当前正在运行的任务 (**准确来说是上文**) ，在进入**上下文切换函数 vTaskSwitchContext** 时，我们不确定会不会 使用 R3 寄存器作为中间变量，为了保险起见也将R3保存起来

将 configMAX_SYSCALL_INTERRUPT_PRIORITY 的值存储到 r0，该宏在 FreeRTOSConfig.h 中定义，用来配置中断屏蔽寄存器 BASEPRI 的值,  **configMAX_SYSCALL_INTERRUPT_PRIORITY配置为191 高4位有效，实际值等于11，即优先级高于或等于11的中断都被关掉**

关中断，进入**临界段**，**进入 上下文切换函数 跟新 pxCurrentTCB**

退出临界段，开中断，恢复 R3 和 R14 的值，此时 SP 使用的是 MSP，**R3 为 pxCurrentTCB这个指针的地址，将其指向的新的任务的TCB地址 赋值给 R1 ，然后 将 新的任务的 TCB的地址的内容 赋值给R0，（这个内容就是 新的任务的栈顶指针）** 将下一个要运行的任务的任务栈的内容加载到CPU寄存器r4~r11。 **将 R0的值 赋值给PSP， 待会PendSV异常结束，PSP指针会自动自增，将自动加载到CPU的内容加载到CPU **

最后 当调用 bx r14指令后，系统以PSP作为SP指针出栈，把接下来要运行的新任务 的任务栈中剩下的内容加载到CPU寄存器：R0（任务形参）、R1、R2、R3、R12、R14 （LR）、R15（PC）和xPSR，从而切换到新的任务。 

#### 5.3 总结

1. 任务切换的接口是 taskYIELD 函数，内部实现细节是挂起 **PendSV 中断, 然后进入中断服务函数中 去处理**
2. **PendSV 中断函数 中（任何中断服务程序，所使用的指针 均是 MSP），先将 上一个运行任务的状态 需要手动保存的部分保存起来 （PSP还没有切换，所以依旧指向上一个任务的栈顶），自动保存的部分 CPU 在进入这个中断函数时就已经保存，然后保存 R14，以及R3（存储pxCurrentTCB的地址**）然后进入 vTaskSwitchText 函数,切换 pxCurrentTCB 的 函数 ，退出后，恢复在 中断服务函数中，保存的R3 和 R14, 然后将PSP的值更新到 下一个任务，退出中断，进入任务模式 将下一个任务的自动加载的部分，加载到CPU



---





