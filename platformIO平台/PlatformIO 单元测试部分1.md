### PlatformIO 单元测试部分1（在主机进行 单元测试）

#### 原版文档链接

[使用 PlatformIO 进行单元测试：第 1 部分。基础知识 |PlatformIO 实验室 --- Unit Testing with PlatformIO: Part 1. The Basics | PlatformIO Labs](https://piolabs.com/blog/insights/unit-testing-part-1.html)



#### 使用单元测试所需要的库函数

![image-20250327215207661](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250327215207661.png)

只需要包含<unity>头文件即可进行单元测试



#### 以下是关于unity的函数官方测试

[安装 ·throwtheswitch/Unity – PlatformIO 注册表 --- Installation · throwtheswitch/Unity – PlatformIO Registry](https://registry.platformio.org/libraries/throwtheswitch/Unity/installation)

![image-20250327225734112](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250327225734112.png)

**注意：仅在Windows上严格要求setup和tearDown函数 为空定义，并且这个函数必须实现**



#### 开始测试

![image-20250328171338695](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250328171338695.png)

首先介绍一下PlatformIO 下的lib 文件夹是用来干什么的？

回答： 这个目录是用于项目特定的（私有的 ）库文件，PlatformIO 会将它们编译成静态库并链接到可执行文件中 ，**需要特别强调的是 每个库的源代码都应该放在自己的单独目录中Example:（lib/your_library_name/[这里是源文件]）**, 在使用库文件的时候，PlatformIO的库依赖查找器会自动扫描项目源文件，找到并识别出所需的依赖库



概念补充：

#### 静态库（Static Library）

静态库是一种将多个目标文件（.o 或 .obj 文件）打包成一个文件（通常是 `.a` 或 `.lib` 扩展名）的文件格式。静态库通常用于在编译时链接到程序中，而不是在运行时动态加载。它的主要特点是：

1. **编译时链接**：当你编译一个程序时，静态库的代码被复制到最终的可执行文件中。这意味着，在程序执行时，静态库的代码已经被包含在程序内部。
2. **不可修改**：静态库一旦被编译，库中的代码就不可再更改。如果需要更新功能或修复bug，必须重新编译和链接静态库。
3. **文件格式**：静态库通常是 `.a` 或 `.lib` 文件。它是一个包含了多个目标文件的归档文件。

#### 链接（Linking）

链接是将编译好的目标文件（.o 文件）和库文件（包括静态库和动态库）组合成一个完整的可执行文件的过程。链接过程有两种类型：

1. **静态链接**：在编译时，将所有依赖的代码（如静态库）复制到最终的可执行文件中。静态链接的优点是执行时不需要依赖外部库文件，但缺点是生成的可执行文件通常较大，而且无法在运行时更新库文件。
2. **动态链接**：在程序运行时，动态链接器（loader）将所需的动态库（如 `.so` 或 `.dll` 文件）加载到内存中。动态链接的优点是可以在不重新编译程序的情况下更新库文件，缺点是需要在运行时加载库文件，可能会增加启动时间，并且需要确保库文件存在。

![image-20250328172140637](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250328172140637.png)

然后在 **test目录下 放入需要测试的文件，然后点击编译，编译完成后，才能进行单元代码测试**

以下是需要测试的函数库，以及测试函数文件

![image-20250328172355051](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250328172355051.png)

![image-20250328172413726](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250328172413726.png)

![image-20250328172423367](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250328172423367.png)



#### 最后测试

点击左侧 小蚂蚁的图标 

![image-20250328172530072](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250328172530072.png)

点击Test 就可以进行单元测试了

![image-20250328173248947](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250328173248947.png)

Good Luck !!! 最后结果通过了