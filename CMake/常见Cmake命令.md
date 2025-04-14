### CMake命令

```
cmake .
```

告诉camke ，CMakeList.txt 在当前目录，**并在当前目录生成构建文件**

```
cmake ..
```

告诉camke ，CMakeList.txt 在上一级目录，并在当前目录生成构建文件 **通常这个命令在Build目录执行**

```
cmake -B build 
```

告诉cmake，将构建文件放在 当前目录的子目录**build**内

```
cmake -G ""
```

告诉cmake,	使用什么生成器，**来构建Cmake项目**

```
cmake --build <dir>
```

使用cmake, 来生成可执行文件，**注意<dir>是构建文件的目录**

```
add_library(name dir1 dir2... )
```

告诉cmake，生成一个静态库，name是这个静态库的名字，dir1是这个静态库包含的文件

```
target_sources(name dir1 dir2 ...)
```

 告诉cmake，在name下，有dir1。。。等文件

```
#链接库 等于GCC 中的 -l参数
target_link_libraries(name lib1 lib2)
```

告诉cmake，对于名字为name的可执行程序的链接库在 lib1 lib2

```cmake
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            [source1] [source2 ...])
```

添加名为name的库，库的源文件可指定，也可用 **target_sources**()后续指定。
库的类型是STATIC(静态库)/SHARED(动态库)/MODULE(模块库)之一。

name属性必须全局唯一
生成的library名会根据STATIC或SHARED成为name.a或name.lib
这里的STATIC和SHARED可不设置，通过全局的BUILD_SHARED_LIBS的FALSE或TRUE来指定
windows下，如果dll没有export任何信息，则不能使用SHARED，要标识为MODULE

添加的库会被输出到以下几个目录
ARCHIVE_OUTPUT_DIRECTORY, LIBRARY_OUTPUT_DIRECTORY和 RUNTIME_OUTPUT_DIRECTORY，详见cmake 常用设定及函数
设置EXCLUDE_FROM_ALL，可使这个library排除在all之外，即必须明确点击生成才会生成