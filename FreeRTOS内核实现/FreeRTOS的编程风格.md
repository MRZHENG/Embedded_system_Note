### 数据类型

​	在**FreeRTOS**当中，使用的数据虽然都是标准C的数据类型，但是针对不同的处理器，对于标准C的数据类型进行了重定义，给它们取了一个新的名字，比如**char**重新 定义了一个名字**portCHAR**，这里面的**port**表示接口的意思，就是**FreeRTOS**要移植到这些 处理器上需要这些接口文件来把它们连接在一起，但是用户在写程序的时候并非一定要遵 循**FreeRTOS** 的风格，我们还是可以直接用C语言的标准类型。在**FreeRTOS**中，**int型从不使用** ，只使用 **short**和 **long**型。在 **Cortex**-M内核的MCU中，**short**为 16位，**long**为 32 位。

​	**FreeRTOS中的数据类型重定义在 portmacro.h 这个 文件实现** (port 是接口的意思，macro 是宏定义的意思)

![image-20250429142932168](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250429142932168.png)

![image-20250429142939257](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250429142939257.png)

![image-20250429143028468](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250429143028468.png)

​	在编程的时候，如果用户没有明确指定 char 的符号类型，那么编译器会默认的指定 char 型的变量为无符号或者有符号。正是因为这个原因，在 FreeRTOS 中，我们都需要明 确的指定变量char是有符号的还是无符号的。在keil中，默认 char是无符号的，但是也可 以配置为有符号的

![image-20250429143137405](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250429143137405.png)

### 变量的命名

​	在 FreeRTOS 中，定义变量的时候往往会把变量的类型当作前缀加在变量上，这样的 好处是让用户一看到这个变量就知道该变量的类型。比如 char型变量的前缀是 c，short型 变量的前缀是s，long型变量的前缀是l， portBASE_TYPE类型变量的前缀是x。还有其他 的数据类型，比如数据结构，任务句柄，队列句柄等定义的变量名的前缀也是x。  如果一个变量是无符号型的那么会有一个前缀 u，如果是一个指针变量则会有一个前 缀 p。因此，当我们定义一个无符号的 char 型变量的时候会加一个 uc前缀，当定义一个 char 型的指针变量的时候会有一个pc前缀。

| 变量类型     | 前缀 |
| ------------ | ---- |
| char         | c    |
| short        | s    |
| long         | I    |
| 其他数据类型 | X    |
| 指针         | p    |



### 函数的命名

 在FreeRTOS当中，函数的命名包含的了如下内容

- 函数返回值类型
- 函数所在的文件名
- 函数的功能
- 如果是私有函数，会加上prv(private)的前缀   

具体的举例如下：  

1. vTaskPrioritySet()函数的返回值为void 型，在task.c这个文件中定义。  
2.  xQueueReceive()函数的返回值为portBASE_TYPE型，在queue.c这个文件中定义。
3.   vSemaphoreCreateBinary()函数的返回值为void型，在semphr.h这个文件中定义。 

### 宏定义的命名

宏均是由大写字母表示，并配有小写字母的前缀，前缀用于表示该宏在哪个头文件定 义

![image-20250429143917845](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250429143917845.png)

**这里有个地方要注意的是信号量的函数都是一个宏定义，但是它的函数的命名方法是 遵循函数的命名方法而不是宏定义的方法。**