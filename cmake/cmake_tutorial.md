# cmake

+ CMake 是一个跨平台的、开源的构建工具。
+ cmake 是 makefile 的上层工具，它们的目的正是为了产生可移植的makefile，并简化自己动手写makefile时的巨大工作量

### 项目目录结构

```
    |--src
        |--main.c   原代码
    |--inc
        |-- xxx.h   头文件
    |--ext
        |--第三方扩展
    |--build
        |--项目构建目录
    |--CMakeLists.txt
```

### 最小配置

+ 命令不区分大小写的

```
    # cmake 最低版本需求
    cmake_minimum_required(VERSION 3.13)

    # 工程名称
    project(Demo)

    # 设置
    set(CMAKE_CXX_STANDARD 11)

    # 将main.c编译成一个名为Demo的可执行文件
    add_executable(demo src/main.c)
```

### 执行过程

1. src开编写代原
2. 编写CMakeLists.txt文件
3. 进入build目录，执行sudo cmake ..
4. build目录下会自动生成Makefile文件，然后执行 sudo make
5. 生成最终可执行文件

### 多文件编译

#### 1、源文件头文件在同一目录下，且文件不多

+ src下有: main.c  add.h  add.c
+ 只需要添加src/add.c

 ``` c
    cmake_minimum_required(VERSION 3.13)
    project(Demo)
    add_executable(demo src/main.c src/add.c)
 ```

#### 2、源文件头文件在同一目录下，且文件很多

+ 使用 **aux_source_directory(dir var)**函数
+ 把dir目录下的所有源文件都储存在var变量中,变量名随便定
+ 需要用到源文件的地方用 变量${var}来取代

 ``` c
├── build
├── CMakeLists.txt
└── src
    ├── add.c
    ├── add.h
    └── main.c
 ```

``` c
    CMakeLists.txt:

    cmake_minimum_required(VERSION 3.13)
    project(Demo)

    aux_source_directory(./src SRC_LIST)

    set(CMAKE_CXX_STANDARD 11)
    add_executable(demo ${SRC_LIST})
 ```

#### 3、头文件在其它目录下

+ add.h在inc目录下，add.c、main.c在src目录下
+ 使用 **include_directories ( dir )**
+ 作用是自动去dir目录下寻找头文件，相当于 gcc中的 gcc -I dir
+ 在build中执行sudo cmake ..
+ 在build中执行sudo make

``` c
    ├── build
    ├── CMakeLists.txt
    ├── inc
    │   └── add.h
    └── src
        ├── add.c
        └── main.c
```

``` c
    CMakeLists.txt

    cmake_minimum_required(VERSION 3.13)
    project(Demo)

    aux_source_directory(src/. SRC_LIST)
    include_directories(inc/.)

    add_executable(demo ${SRC_LIST})
```

#### 4、头文件、源文件分离，且有多个文件夹

+ 在build中执行sudo cmake ..
+ 在build中执行sudo make

``` c
    ├── build
    ├── CMakeLists.txt
    ├── inc1
    │   └── add.h
    ├── inc2
    │   └── sub.h
    ├── main
    │   └── main.c
    ├── src1
    │   └── add.c
    └── src2
        └── sub.c
```

``` c
    CMakeLists.txt:

    cmake_minimum_required(VERSION 3.13)
    project(Demo1)

    aux_source_directory(./src1 SRC1)
    aux_source_directory(./src2 SRC2)
    aux_source_directory(./main MAIN_DIR)

    include_directories(./inc1 ./inc2)
    add_executable(demo ${MAIN_DIR} ${SRC1} ${SRC2})
```

### 生成动态库和静态库

#### 1、关于路径

``` c
    ├── build
    ├── CMakeLists.txt
    ├── inc
    │   └── func1.h
    ├── lib
    └── src
        └── func1.c
```

#### 说明

+ 不管在哪个目录下执行 cmake,在CMakeLists.txt文件中，当前目录都是CMakeLists.txt所在的目录
+ PROJECT_BINARY_DIR是cmake系统变量，意思是执行cmake命令的目录
+ set_target_properties重新定义了库的输出名称，如果不使用set_target_properties也可以，那么库的名称就是add_library里定义的名称
+ LIBRARY_OUTPUT_PATH 是cmake系统变量，项目生成的库文件都放在这个目录下。这里我指定库生成到lib目录

+ 开始编译
  + 在build里sudo cmake ..
  + 在build里sudo make
  + 自动在lib目录下生成库文件

```  C
    CMakeLists.txt:

    cmake_minimum_required(VERSION 3.10)
    project(Demo_lib)

    # 整合源文件 
    aux_source_directory(${PROJECT_BINARY_DIR}/../src SRC_LIST)

    # 引入头文件路徑
    include_directories(${PROJECT_BINARY_DIR}/../inc)

    # 生成静态库和动态库，参数一：生成的库名称 参数二：静态或动态  参数三：生成所需要的源文件
    add_library(func_shared SHARED ${SRC_LIST})
    add_library(func_static STATIC ${SRC_LIST})

    # 设置最终生成的库名称，这里把库名称统一为myfunc
    set_target_properties(func_shared PROPERTIES OUTPUT_NAME "myfunc")
    set_target_properties(func_static PROPERTIES OUTPUT_NAME "myfunc")

    # 设置库默认生成到哪里
    set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/../lib)
```

``` c
func1.c:

#include <stdio.h>
#include "func1.h"

void func1(){
    printf("func1 \r\n");
}
```

> add_library(lib_name STATIC/SHARED src)
>>函数作用：生成库。
>>参数lib_name：是要生成的库名称，
>>参数STATIC/SHARED：指定生成静态库或动态库，
>>参数src：指明库的生成所需要的源文件

### 使用生成的库文件

``` c
├── bin         存放生成的可执行文件
├── build       存放cmake的编译临时文件
├── CMakeLists.txt
├── inc         头文件目录
│   └── func1.h
├── lib         库文件目录
│   ├── libmyfunc.a
│   └── libmyfunc.so
└── main_src    程序入口
    └── main.c
```

``` c
main.c:

#include <stdio.h>
#include "func1.h"

int main(void){
    func1();
    return 0;
}
```

```c
CMakeLists.txt:

cmake_minimum_required(VERSION 3.10)
project(Demo_lib)

# 整合源文件
aux_source_directory(${PROJECT_BINARY_DIR}/../main_src MAIN_SRC)

# 引入头文件路径
include_directories(${PROJECT_BINARY_DIR}/../inc)

# 查找库文件
# 参数一：是一个变量，用于储存查找到的库文件
# 参数二：要查找的库文件
# 参数三：要在哪个目录下查找
find_library(FUNC_LIB myfunc ${PROJECT_BINARY_DIR}/../lib)

# 设置可执行文件在哪里生成
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/../bin)

# 生成可执行文件
add_executable(hello ${MAIN_SRC})

# 把库链接到可执行文件中
target_link_libraries(hello ${FUNC_LIB})
```

#### 说明

+ EXECUTABLE_OUTPUT_PATH是cmake系统变量，意思是生成的可执行文件的的目录

> target_link_libraries(target lib_name)
>> 函数作用：把库lib_name链接到target可执行文件中

> find_library(var lib_name lib_path1 lib_path2)
>> 函数作用：查找库，并把库的绝对路径和名称存储到第一个参数里
>> 参数var：用于存储查找到的库
>> 参数lib_name：想要查找的库的名称，默认是查找动态库，想要指定查找动态库或静态库,可以加后缀，例如 funcname.so 或 funcname.a
>> 参数lib_path：想要从哪个路径下查找库，可以指定多个路径

### CMake其他功能

#### 添加编译选项

+ 有时编译程序时想添加一些编译选项，如-Wall，-std=c++11等，可以使用add_compile_options来进行操作

``` c
add_compile_options(-std=c++11 -Wall -o2)
```
