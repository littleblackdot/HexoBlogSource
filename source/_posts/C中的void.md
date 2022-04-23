---
title: C中的void
date: 2022-04-23 19:15:12
tags:
---

&nbsp;

<!--more-->

参考文章:https://www.iteye.com/blog/zhengdl126-1739165

https://stackoverflow.com/questions/7016861/why-are-null-pointers-defined-differently-in-c-and-c

void最初接触当然是在学C的时候，见到的用法就是指明函数无需返回指，在学校计算机系统基础课的malloc_lab(CSAPP里的)里，大量的用到了void类型的指针，在做实验的时候了解了一下void的用法；

首先当然是百度后看前辈们的分享的知识；

#### VOID用法:

1. 如果函数没有返回值，那么应声明为void类型;
2. 如果函数无参数，那么应声明其参数为void；
3. 如果函数的参数可以是任意类型指针，那么应声明其参数为void * ；
4. void不能代表一个真实的变量；

------

#### VOID*使用注意事项：

1. 任何类型的指针都可以直接赋值给void* 类型变量；但是，void * 类型的变量作为右值赋给任意类型指针变量时要做类型转换。因为“无类型”可以包容“有类型”，而“有类型”则不能包容“无类型”；（C++适用，C中不适用）

e.g.

```
int a = 0;
int *p = &a;
void *vp = NULL;
p = (int *)vp;
vp = p;
```

在网上百度看一种说法，NULL是((void *) 0),的宏,依据是stdio.h里的声明：

```
#ifndef NULL
#ifdef __cplusplus
#define NULL    0
#else  /* __cplusplus */
#define NULL    ((void *)0)
#endif  /* __cplusplus */
#endif  /* NULL */
```

上面可以看出在C++中NULL是 0，在C中NULL被定义为 (（void*)0 )，在c中void* 类型的值可以直接赋值给任意类型的指针变量，在dev C++分别在.c和.cpp文件里比编译下面这个语句：

```
int *p = (void*)0
```

前者编译后没有任何提示信息，后者会报错…….

至于为什么c和c++的NULL不一样，stackoverflow里看到了一个问答，里面给出了C和C++对空指针常量(NULL)的不同定义:

Back in C++03, a null pointer was defined by the ISO specification (§4.10/1) as

> A null pointer constant is an integral constant expression (5.19) rvalue of integer type that evaluates to zero.

In C, this rule is similar, but is a bit different (§6.3.2.3/3):

> An integer constant expression with the value 0, or such an expression cast to type `void *`, is called a null pointer constant.55) If a null pointer constant is converted to a pointer type, the resulting pointer, called a null pointer, is guaranteed to compare unequal to a pointer to any object or function.

看来在c中0和(void*)0都能表示空指针常量，所以在C和C++里都能这样写: `int *p = 0;`,

在C++里可以写看起来怪但没有错的语句：`int a = NULL;`,这里又引出几个nullptr之于NULL的优点。

如果再看到这里又想不起来就重新看看吧：https://stackoverflow.com/questions/7016861/why-are-null-pointers-defined-differently-in-c-and-c

1. 如果函数没有返回值，那么应声明为void类型;
   在C语言中，凡不加返回值类型限定的函数，就会被编译器作为返回整型值处理。
   如果函数无参数,那么应声明其参数为void
   在C++语言中声明一个这样的函数：

```
int function(void){	return 1;}
```

则进行下面的调用是不合法的：
function(2);
因为在C++中，函数参数为void的意思是这个函数不接受任何参数。
我们在Turbo C 2.0中编译：

```
#include "stdio.h"
fun(){
	return 1;
}
main(){
	printf("%d",fun(2));
	getchar();
}
```

编译正确且输出1，这说明，在C语言中，可以给无参数的函数传送任意类型的参数，但是在C++编译器中编译同样的代码则会出错。在C++中，不能向无参数的函数传送任何参数，出错提示“’fun’ : function does not take 1 parameters”。所以，无论在C还是C++中，若函数不接受任何参数，一定要指明参数为void。
\3. void不能代表一个真实的变量
void a; //错误
function(void a); //错误

#### 小结

正如博主所说，void体现了一种抽象,void*很类似面向对象里的泛型。面向过程语言也有面向对象的思想，面向过程也可以写面向对象的东西。语言只是工具，编程重要的是思想。