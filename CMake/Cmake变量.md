### CMake 变量

#### 普通变量

##### 设置普通变量

```cmake
set(varName value... [PARENT_SCOPE])
```

- varName ：变量名
- value ： 变量值 
- PARENT_SCOPE: (父级作用域) 可选项

##### 普通变量名

1. 建议使用字母，数字， “_” 或 “-”
2. 区分大小写
3. 不要使CMAKE_ 开头定义自己的变量

![image-20250414110121004](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250414110121004.png)

**在这个存储列表** 例子中 ，由于CMake 存储的所有变量都是字符串，存储列表时，可以使用空格，或者;

（'';“ 是专门用于列表的分隔符）要是不需要 ; 作为列表的分隔符 , 就需要使用转义字符

![image-20250414110355964](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250414110355964.png)

**当我们在变量中需要使用空格，就要采用字符串形式 或者 [[ ]] 形式**

![image-20250414110513592](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250414110513592.png)

![image-20250414111109943](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250414111109943.png)

**当我们需要使用或者引用一个变量时 则使用**

```cmake
${}
```

![image-20250414110639349](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250414110639349.png)



CMake 可以对变量定义 和 取消定义

![image-20250414111223325](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250414111223325.png)

**如果CMake 变量是未被定义，那么如果引用这个 变量 则会得到一个 空串 但是他和下面这个是有区别的**

![image-20250414111409079](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250414111409079.png)

#### 缓存变量 

缓存变量存储在 **CMakeCache.txt** 中

```cmake
set(
	varName
	value...
	CACHE
	type
	"helpstring"
	[FORCE]
)
```

1. **`varName`**：

   - 这是你要设置的变量的名称。例如，`CMAKE_BUILD_TYPE` 或自定义变量名 `MY_VAR`。

2. **`value...`**：

   - 这是你希望为变量 `varName` 设置的值。你可以设置一个或多个值，通常是字符串、数字等。例如，`"Debug"`、`"Release"` 或 `10`。

3. **`CACHE`**：

   - 关键字 `CACHE` 指示你正在设置一个缓存变量。CMake 中的缓存变量会被保存到 `CMakeCache.txt` 文件中，使得这些变量在多次构建过程中能够保持不变（即使 CMake 配置被重新运行）。

4. **`type`**：

   - 这是缓存变量的类型。常见的类型包括：

     - `BOOL`：布尔类型，值为 `ON` 或 `OFF`  **不区分大小写**。

     ![image-20250414112335495](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250414112335495.png)

     ​        这些也可以是BOOL 类型存储的值

     ​		对于 BOOL 类型的缓存变量 可以使用 option 命令来定义

     ```cmake
     option(optVar helpString [初始值])  #初始值默认为OFF
     #option 命令的等价形式 不使用 FORCE 选项 使用 set 来定义
     set(optVar [初始值] CACHE BOOL helpString)
     ```

     

     - `STRING`：字符串类型，值为字符串。
     - `PATH`：路径类型。
     - `FILEPATH`：文件路径类型。
     - `INTERNAL`：用于内部变量，CMake 会忽略该变量的值并避免用户修改。

   - 类型用于定义该变量的行为和预期数据类型。

5. **`"helpstring"`**：

   - 这是一个可选的帮助信息，用于描述变量的用途或含义。当你在 CMake GUI 或命令行界面中查看缓存变量时，帮助字符串会显示出来，帮助用户了解变量的意义和作用 **相当于注释**。

6. **`[FORCE]`**：

   - `FORCE` 是一个可选参数。表示是否是 **强制更新缓存**  ，如果不使用 **FORCE** , **则只有当缓存中没有这个变量时，才会写入，如果已经有这个变量，那么不会更新缓存中的值**

#### 环境变量

##### 设置环境变量 （这种方式只是临时添加环境变量，而不会改变系统的环境变量）

```
set(ENV{varName} value)
```



### 对于同时定义的普通变量和缓存变量相同的情况

- 会优先使用普通变量的值
- 只有当定义一个缓存变量，而没有定义同名的变量，才会使用缓存变量的值

##### EXAMPLE:

![image-20250414113438378](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250414113438378.png)

![image-20250414113448587](https://mrzhb.oss-cn-chengdu.aliyuncs.com/image-20250414113448587.png)



### 使用命令行定义变量

```
cmake -D<VAR_NAME>=<VALUE> <path_to_source>
```

- `<VAR_NAME>` 是变量名。
- `<VALUE>` 是你希望为该变量设置的值。
- `<path_to_source>` 是项目源代码的路径，通常是 CMakeLists.txt 文件所在的目录。



### 使用命令行删除变量

```
cmake -U<VAR_NAME> <path_to_source>
```

- `<VAR_NAME>` 是你希望取消定义的变量名。
- `<path_to_source>` 是项目源代码的路径，通常是 CMakeLists.txt 文件所在的目录。



### 作用域

CMake 内变量作用域如下

- 全局：缓存变量
- 局部： 函数，以及子目录 当中都有自己的作用域
- 在进入局部作用域的时候，他会复制一份父作用域的变量吗

```cmake
#定义作用域
block()
...
end_block()
```



##### 全局作用域和局部作用域的细节

###### 1. **局部作用域与全局作用域**

CMake 的作用域分为全局作用域和局部作用域。在全局作用域中定义的变量可以在其所有子作用域中访问，但如果在局部作用域中修改了这些变量，它们的变化不会影响到父作用域中的变量，除非显式使用 `set()` 或 `unset()` 等命令。

###### 2. **变量的作用域继承**

在进入局部作用域（比如通过 `function()` 或 `macro()` 等构造）时，**局部作用域会继承父作用域的变量**。**但这并不复制**，**而是引用**。这意味着如果局部作用域修改了这些变量的值，父作用域中的相应变量不会受到影响。

###### 3. **局部作用域中的变量修改**

如果你希望在局部作用域中修改父作用域中的变量值，可以使用 `PARENT_SCOPE` 选项：

```
cmakeCopy Codeset(MY_VAR "New value" PARENT_SCOPE)
```

这样，`MY_VAR` 会被修改并在父作用域中生效。

###### 4. **局部作用域内的变量不会影响父作用域**

在 CMake 中，局部作用域内的变量会继承父作用域的值，但如果在局部作用域中修改了这些变量，它们不会自动更新父作用域中的变量。这是因为 CMake 会在局部作用域内为变量创建**局部副本**。

###### 示例：

```
cmakeCopy Code# 全局作用域定义变量
set(MY_VAR "Global value")

# 进入局部作用域
function(my_function)
  # 局部作用域继承父作用域的变量
  message("In function, MY_VAR = ${MY_VAR}")
  
  # 修改局部作用域中的变量
  set(MY_VAR "Local value")
  message("In function, after change MY_VAR = ${MY_VAR}")
endfunction()

# 调用函数
my_function()

# 查看父作用域中的变量
message("Outside function, MY_VAR = ${MY_VAR}")
```

###### 输出：

```
Copy CodeIn function, MY_VAR = Global value
In function, after change MY_VAR = Local value
Outside function, MY_VAR = Global value
```

###### 解释：

- **父作用域的 `MY_VAR`** 值没有改变，依然是 `Global value`。
- 在局部作用域（`function` 内部）修改的 `MY_VAR` 不会影响父作用域中的 `MY_VAR`。

###### 5. **函数和宏的区别**

- **函数** (`function()`) 中的变量是局部的，修改时不会影响到父作用域。
- **宏** (`macro()`) 中的变量同样是局部的，并且在宏内修改变量时也不会影响父作用域。

