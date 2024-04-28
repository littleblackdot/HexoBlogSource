---
title: CMake笔记
date: 2024-04-19 22:59:34
tag: 
---

&nbsp;

<!--more-->

平时项目中会涉及一些的CMake脚本的修改,但都是需要改的地方去现查现或者参考已经写好的部分,在开始系统的看一些入门要点,然后记录一下

## CMake常用指令

- #### cmake_minimum_required - 指定CMake的最小版本要求

  - 语法: **cmake_minimum_required(VERSION versionNumber [FATAL_ERROR])**

    ```
    #cmake最小版本要求3.0.1
    cmake_minimum_required(VERSION 3.0.1)
    ```

- #### project - 定义工程名称，并可指定工程支持的语言

  - 语法：**project(projectName [CXX] [C] [JAVA])**

    ```
    #指定工程名为XXController
    project(XXController)
    ```

- #### set -显示定义变量

  - 语法：**set(var [value] [cache type docstring [force]])**

    ```
    #定义SRC变量， 其值为main.cpp test.cpp
    set(SRC main.cpp test.cpp)
    ```


- #### include_directories -向工程添加多个特定的库文件搜索路径

  - 语法：**inclide_directories([AFTER|BEFOR] [SYSTEM] dir1 dir2 ...)**

    ```
    #将 /usr/include/myincludeFolder 和 ./include 添加到头文件搜索路径
    include_directories(/usr/include/myincludeFolder ./include)
    ```

- #### link_directories -向工程添加多个特定的库文件搜索路径

  - 语法：**link_directories(dir1 dir2 ...)**

    ```
    #将/usr/lib/mylibFolder和./lib添加到库文件搜索路径
    link_directories(/usr/lib/mylibFolder ./lib)
    ```

- #### add_library -生成库文件

  - 语法：**add_library(libname [SHARED|STATIC|MODULE] [EXCLUDE_FROM_ALL] source1 source2 ...)**

    ```
    #通过变量 SRC 生成 libHello.so 库
    add_libaray(libHello SHARED ${SRC})
    ```
- #### add_compile_options -添加编译选项

  - 语法：**add_compile_options(\<option\> ...)**

    ```
    #添加编译选项 -Wall -std=c++11 -o2
    add_compile_options(-Wall -std=c++11 -o2)
    ```
- #### add_exectuable -生成可执行文件

  - 语法：**add_exectuable(exeName source1 source2 ...)**

    ```
    #编译main.cpp 生成可执行文件
    add_exectuable(main main.cpp)
    ```
- #### target_link_libraries -为target添加需要链接的共享库

  - 语法：**target_link_libraries(target library1<debug | optimized> ...)**

    ```
    #将hello动态库文件链接到可执行文件main
    target_link_libraries(main hello)
    ```
- #### add_subdirectory -向当前工程添加存放源文件的子目录，并可以指定中间二进制和目标二进制存放的位置

  - 语法：**add_subdirectory(sourceDir [binaryDir] [EXCLUDE_FROM_ALL])**

    ```
    #添加src子目录
    add_subdirectory(src)
    ```
- #### aux_source_directory -发现一个目录下所有源代码文件并将列表存储在一个变量中，这个指令临时被用来自动构建源文件列表

  - 语法：**aux_source_directory(dir var)**

    ```
    #定义src变量，其值为当前目录下所有的源代码文件
    add_subdirectory(src)
    # 编译src变量所代表的源代码文件，生成main可执行文件
    add_executable(main ${src})
    ```

## CMake常用变量

- CMAKE_C_FLAGS: gcc编译选项

- CMAKE_CXX_FLAGS: g++编译选项

  ```
  #在CMAKE_CXX_FLAGS选项追加-sdt=c++11
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
  ```

- CMAKE_BUILD_TYPE: 编译类型(Debug Release)

  ```
  #设置编译类型为Debug
  set(CMAKE_BUILD_TYPE Debug)
  ```

- CMAKE_C_COMPILER: 指定C编译器

- CMAKE_CXX_COMPILER:指定c++编译器
- EXECUTEABLE_OUTPUT_PATH:指定可执行文件的输出路径
- LIBRARY_OUTPUT_PATH:库文件输出的存放路径