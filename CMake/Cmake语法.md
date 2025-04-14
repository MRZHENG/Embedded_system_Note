## Cmake语法

cmake 命令工具由 5个可执行文件构成

- cmake
- ctest
- cpack
- cmake-gui
- ccmake



#### message 语法 获取变量信息

```cmake
使用 这个符号获得变量的值 ${}
```

eg:

```cmake
#变量名＋字符串
set(myVar cat)
message("myVar = ${myVar}")

set(pet my${myVar})

message("pet = ${pet}")
message("CMake_Version ${CMAKE_VERSION}")
```



#### 变量操作 Set,list

- CMake 变量分为两种  一种是Cmake提供，或者是自定义

- CMake 变量命名是区分大小写的

- CMake **中的变量在存储的时候都是字符串**

- CMake 获取变量

  ```cmake
  ${变量名}
  ```

- 变量的基础操作是Set( ) 与 unset( )，但是你也可以用list或是string操作变量 





#### Set方法

- set(<variable> <value> ... [PARENT_SCOPE])
- set可以给一个变量设置多个值
- 变量内部存储时使用 “ ; ” 分割，但显示时只进行连接处理

```cmake
cmake_minimum_required(VERSION 4.0)

set(Var1 YZZY)
message(var: ${Var1})

#设置存在空格的变量 有两种方式
#1.加[[ ]] 
set([[My Var]] ZZZ)
message(My Var: ${My\ Var})

#2.使用“ ”
set("My Var2" WWW)
message(My Var2: ${My\ Var2})

#设置多个值
set(Listvalue a1 a2)
message(${Listvalue})
#说明可以重复设置
set(Listvalue a1;a3)
message(${Listvalue})


# $path 如何获取系统的一些变量
message($ENV{PATH})
#message($ENV{CXX})

# 给环境变量创建一些值
set(ENV{CXX} g++)
message($ENV{CXX})

#unset 对于一些没有值的变量，获取该变量会报错
unset(ENV{CXX})
message($ENV{CXX})
```

#### List方法

![image-20250412193629804](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250412193629804.png)

##### example

```cmake
cmake_minimum_required(VERSION 4.0)

#两种方式创建Val
message("----------------------------")
set(ListValue1 a1 a2 a3)
message(ListValue1: ${ListValue1})

list(APPEND ListValue2 p1 p2 p3 p2)
message(ListValue2: ${ListValue2})

message("----------------------------")
#排序
list(SORT ListValue2)
message(ListValue2: ${ListValue2})

message("----------------------------")
#获取长度
list(LENGTH ListValue1 len)
message(${len})

message("----------------------------")
#获取索引
list(FIND ListValue1 a1 index)
message( ${index})

message("----------------------------")
#删除元素
list(REMOVE_ITEM ListValue1 a1)
message(ListValue1: ${ListValue1})

message("----------------------------")
#添加元素
#尾插法
list(APPEND ListValue2 p4)
message(ListValue2: ${ListValue2})

#指定位置添加
list(INSERT ListValue2 3 p5)
message(ListValue2: ${ListValue2})
```

