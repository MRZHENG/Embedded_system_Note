### CMake构建项目的流程

![image-20250413154429664](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250413154429664.png)



### 直接引入源文件的方式

![image-20250413151913874](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250413151913874.png)

### 常见用法示例：

1. **创建构建目录**： 通常，建议在源代码目录外创建一个单独的构建目录，这样可以保持源代码目录整洁。

   例如，假设源代码在 `project/` 目录下，通常的步骤如下：

   ```
   mkdir build   # 创建一个单独的构建目录
   cd build      # 进入构建目录
   cmake ..      # 使用 cmake 生成构建文件，.. 表示返回上一级目录，即源代码目录
   ```

   这样做的好处是将构建文件与源代码文件分开，避免它们混在一起。

2. **直接在源代码目录中运行 `cmake .`**： 如果你不想创建单独的构建目录，可以直接在源代码目录中运行 `cmake .`：

   ```
   cd project  # 进入源代码目录
   cmake .     # 在当前目录生成构建文件
   ```

   这样会在当前目录生成 `Makefile` 或其他构建文件。





### 调用子目录的CMake脚本

![image-20250413160210519](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250413160210519.png)

![image-20250413162125407](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250413162125407.png)

**尤其需要注意的是.cmake的文件路径，必须以项目根目录来延展开**

比如这个animal 就不能省略 ，**否则就会报错** ，因为CMake存储的变量都是字符串 ，所以需要以当前项目的根目录来书写路径



### CMakeLists 嵌套

- target_include_directories    头文件目录的声明
- target_link_libraries               连接库文件
- add_subdirectory                   添加子目录
- add_library                              生成库文件

**Example:**

文件目录：

![image-20250413183840857](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250413183840857.png)

根目录的CMakeLists.txt

![image-20250414104236639](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250414104236639.png)

animal目录下的CMakeList.txt

![image-20250414104111933](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250414104111933.png)



#### Object Libraries

- add_library OBJECT 

  Object Library 是一个特殊的库类型，**它将目标文件编译成一个库，但是不会生成最终的链接文件（即静态库文件 .a 或者 动态库文件 .so）**

​		这意味你可以在后续的 add_library( ) 或 add_executable( ) 命令中，将 Object Library 作为源文件		进行链接，从而生成最终的可执行文件或库文件 **(注意：这个有版本要求)**

​		理解：**Object Library** 可以理解为将多个 `.o` 文件（即目标文件）汇集在一起，但它并不会进行最		终的链接操作，也不会生成最终的静态库（`.a`）或动态库（`.so`）文件。

​		具体来说，**Object Library** 是一个包含多个编译后的目标文件（`.o` 文件）的集合。它的作用主要		有两个：

1. **组织多个目标文件**：你可以把多个编译过的 `.o` 文件通过 Object Library 组织在一起，方便后续管理。
2. **灵活的链接机制**：这些目标文件可以被传递给其他目标（比如可执行文件或其他库），在后续的构建中完成最终的链接过程。这样，你就可以避免重复编译相同的源文件。