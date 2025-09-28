### List命令

```cmake
list(操作关键字 <listvar> <其他参数>)
```

#### Example:

##### 1.获取长度

```cmake
set(listvar a b c)
list(LENGTH listvar listLength)
message("listLength : ${listLength}")
```

##### 描述：

​		当前代码，使用list 命令，获取 变量listvar 的 长度，并将长度保存到listLength 当中

##### 2.获取指定索引元素

```cmake
set(listvar a b c)
list(GET listvar 0 firstItem)
message("firstItem : ${firstItem}")
```

 		当前代码，使用list 命令，获取 变量listvar 的 首元素

##### 3.尾插法添加索引元素

```cmake
set(listvar a b c)
list(APPEND listvar d) #添加指定元素
list(JOIN listvar "-" outtStr)#将每个元素以 "-" 来连接
message("outtSte : ${outtStr}")

```

![image-20250419094408614](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250419094408614.png)





### String 命令

##### 描述：

​		可以提供 替换，子串，以及正则表达式匹配等 字符串的处理

##### 1.查找

```cmake
string(FIND interesting in fIndex) #正向查找元素 "in" 结果存放在新的变量 fIndex 中
string(FIND interesting in rIndex REVERSE) #反向查找元素“in” 结果存放在新的变量 rIndex 中
```





### math 命令

```cmake
set(myvar 1+2*6/2)
message( "myvar : ${myvar}")
```

在当前代码的运行结果是，打印这个字符串，而不是我们想要做的运算结果，**如果我们需要使用运算，就需要math**

```cmake
math(EXPR <variable> "<expr>" [OUTPUT_FORMAT <format>])
```

math 对数学运算表达式 <expr> 来求值，并将值，复制给新创建的变量variable，<format>可以是16进制

![image-20250419110716004](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250419110716004.png)

```cmake
set(a 1+2*6/2)
math(EXPR b a*3)
message("a = ${a}\n b = ${b}")

```

![image-20250419111722094](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250419111722094.png)

### file命令

![image-20250419111125955](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250419111125955.png)

##### Example:

```cmake
file(GLOB varFilelist LIST_DIRECTORIES false RELATIVE ${CMAKE_SOURCE_DIR} "*.cpp")
message("varFileslist ${varFilelist}")
```

作用：

- `GLOB` 是用于在目录中查找符合模式的文件并将其列表存储到指定变量中的命令。
- `varFilelist` 是用来存储文件列表的变量名。
- `LIST_DIRECTORIES false`：这是一个选项，表示不将目录添加到文件列表中。`false` 表示不会将目录包含在列表中，只会包含符合模式的文件。
- `RELATIVE ${CMAKE_SOURCE_DIR}`：这是一个选项，用于指定返回的文件路径相对于 `${CMAKE_SOURCE_DIR}`。`${CMAKE_SOURCE_DIR}` 是 CMake 工程的根目录。这样会将文件路径转换为相对于源目录的路径，而不是绝对路径。
- `"*.cpp"`：这是一个文件匹配模式，表示要匹配所有扩展名为 `.cpp` 的文件。



### 生成器表达式

生成器表达式是在生成目标时才进行求值的

也就是在 执行 cmake --build 命令时，才会

获得表达式的值

```cmake
$<...>
```

#### 条件判断

```cmake
$<condition:trueValue> #condition 为1 则得到trueValue,否则就是空串
$<IF:condition,trueVale,falseValue># condition为1 则得到trueValue 否则就是 falseValue

```

注意: condition 只能 是 0/1 其他值都会报错

#### 真值转化

```cmake
$<BOOL:value> #当value为真时得到1 否则 0
```

将 变量转换成 真值 0/1

#### 逻辑运算

```cmake
$<AND:...>
$<OR:...>
$<NOT:value>
#需要注意的是里面的条件取值也只能是0/1

```

