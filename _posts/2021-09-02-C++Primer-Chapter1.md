---
layout: post
title: C++ Primer|Chapter 1
date: 2021-09-02
Author: 小猫猫
categories: 
toc: true
tags: [sample, document]
comments: true
--- 

C++ Primer|Chapter 1|预备

## 编译

进入vs的本机工具命令提示（Windows+cmd的命令行界面要另外配置环境参数）

```
D:\vs2013\VC>cd D:\ConsoleApplication1\ConsoleApplication38//进入该路径

D:\ConsoleApplication1\ConsoleApplication38>cl prog.cpp  //编译源文件

/out:prog.exe      //可执行文件
prog.obj           //目标文件（缺少链接处理）

D:\ConsoleApplication1\ConsoleApplication38>start prog.exe //运行可执行文件

windows: echo %ERRORLEVEL% //获得程序状态
UNIX: echo $?
```

*如果没有后缀，可以*

*文件---查看---√文件扩展名*

头文件：类的类型一般存储在头文件中，标准库的头文件使用`<>`，非标准库的头文件使用`""`。声明写在`.h`文件，定义实现写在`.cpp`文件。



```
ls
//list command

gcc hello.c
gcc -o hello hello.c//重命名生成的exe文件

gcc hello.c -lmath//链接math.h
gcc -o hello hello.c -lcs50//重命名exe文件&链接cs50.h
```

## IO

```
#include <iostream>`
std::cout << "hello"`<< std::endl;
std::cin >> v1`;
```

- #include <iostream> **iostream**里面有cin、cout、cerr、clog

- `>>`和`<<`是**operator** 

`<<`左operand是`cout`这个ostream object  右operand是`“hello”`这个value

而`std::cout << "hello"`作为一个expression整体的result是左值`cout`

所以接下来相当于`(std::cout << "hello") << std::endl;`就是把endl输入到cout了

- **endl**：被称为manipulator的特殊值

  效果：end the current line + 将相关缓冲区（buffer）中的内容flush到外部设备中 *而不是waiting in the memory*

  程序员要注意 写print statement不能忘记用endl去 flush stream 否则if the program crashed, the output may be left in the buffer, leading to the incorrect inferences about where the program crashed.

- UNIX和Mac下键盘输入文件结束符：`ctrl+d`，Windows下：`ctrl+z`


- `std::cout `代表 cout这个name is defined in the namespace ’std‘   而不是自己define的一个cout  `::`是作用域scope运算符

  如果要简化 用using statement—— `using std::cout;`之后就可以直接写cout了
  
- *unistd.h——sleep()函数 可以暂停（）秒*

**成员函数（类方法）**：使用`.`调用。

****：使用``调用。

## 使用文件重定向

```
./main <infile >outfile
```
