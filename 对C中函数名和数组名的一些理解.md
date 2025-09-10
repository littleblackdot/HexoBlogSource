---
title: 对C中函数名和数组名的一些理解
date: 2022-04-23 19:23:40
tags:
---

&nbsp;

<!--more-->

参考文章：

https://blog.csdn.net/lin37985/article/details/43020621

https://www.zhihu.com/question/293674445

https://www.cnblogs.com/liqiuhao/p/7737539.html

#### 前言：

 在想实现一个通用性高的链表的时候想用回调函数，但是只知道可以用函数指针来实现，而且自己的函数指针也只是停留在老师教的层次，没有怎么用过，在百度了用法之后，感觉有些不正常，记录下，以后可能会再用到；

#### 正篇：

 我接触C的函数指针时学到的用法如下：

```
void func(int a){
    printf("%d \n", a);
}
int main(){
    void (*fp)(int) = &func;
    (*fp)(2);
}
```

上面的例子定义了一个函数指针fp，并通过fp来调用函数func，这里的用法很正常，符合一般指针的使用方式

先把func的地址给fp，在通过*fp访问func。

再看回调函数的一般用法

```
typedef void (*FuncType)(int);//定义一个函数指针类型FunType
void MyFun1(int x);
void MyFun2(int x);
void MyFun3(int x);
void CallMyFun(int, FunType)
int main(){
   CallMyFun(MyFun1,10); 
   CallMyFun(MyFun2,20);
   CallMyFun(MyFun3,30);
}
void CallMyFun(FunType fp,int x) 
{
   fp(x);
}
void MyFun1(int x) 
{
   printf("函数MyFun1中输出：%d\n",x);
}
void MyFun2(int x)
{
   printf("函数MyFun2中输出：%d\n",x);
}
void MyFun3(int x)
{
   printf("函数MyFun3中输出：%d\n",x);
}
```

我感觉奇怪的地方在函数指针调用的时候（CallMyFun中)：

在上面基础的用法是：

```
fp = &func;(*fp)(1);
```

这里的用法是：

```
fp = funcfp(1);
```

甚至在看了博客后还有这样的用法（这个用法还是有惊艳到我的）

```
(*func)(1);//func是函数名
```

怪的地方来了，**可以通过函数名，函数名的地址，甚至是函数名指向的内容来引用函数**

难道说这三个值是一样的？于是去试了一下

```
void func(int a){ 
    printf("%d \n", a); 
}

int main(){
    void (*fp)(int) = &func;
    (*fp)(1);
    fp(1);
    fp = func;
    fp(1);
    (*func)(1);
    printf("func: %p\n", func);
    printf("&func: %p\n", &func);
    printf("*func: %p\n", *func);
    return 0;
}
```

上机运行后发现三个值确实一样，但为什么回事这个结果？会是编译器的特殊处理吗？函数名不是函数的入口地址吗？起始内容不应该是函数内容相关的指令吗？

看了看博客给的解释：

1）其实，MyFun的函数名与FunP函数指针都是一样的，即都是函数指针。MyFun函数名是一个函数指针常量，而FunP是一个函数数指针变量，这是它们的关系。
2）但函数名调用如果都得如(*MyFun)(10)这样，那书写与读起来都是不方便和不习惯的。所以C语言的设计者们才会设计成又可允许MyFun(10)这种形式地调用（这样方便多了并与数学中的函数形式一样，不是吗？）。
3）为统一起见，FunP函数指针变量也可以FunP(10)的形式来调用。
4）赋值时，即可FunP = &MyFun形式，也可FunP = MyFun。

他给的解释也没有提到具体的实现细节。而且描述的不是很清晰，不是那么的让人接受。。。

终于知乎上(参考文章第二篇)看到了一个让人满意的答案：

```
func 得到函数地址，是因为它是函数指示符。只有在作为 sizeof 或者单目 & 操作符的操作数时，它的类型才是函数；其它情况都会被转化为指向该函数的指针。

A function designator is an expression that has function type. Except when it is the operand of the sizeof operator or the unary & operator, a function designator with type ''function returning type'' is converted to an expression that has type ''pointer to function returning type''.

6.3.2.1 Lvalues, arrays, and function designators

&func 得到函数地址，是因为单目 & 操作符本来就是用来取操作数的地址的。而根据上一条，此处操作数的类型就是 func 函数，所以这个表达式可以得到 func 函数的地址。

The unary & operator yields the address of its operand.

6.5.3.2 Address and indirection operators

*func 得到函数地址，是因为本来就有相关的规定，表达式 *函数 的值是对应的函数指示符，于是参见第一条。

The unary * operator denotes indirection. If the operand points to a function, the result is a function designator;

6.5.3.2 Address and indirection operators

p.s. 以上引用内容为 C99

p.p.s. 数组同理，并且文档都在这些条目的周边。
```

数组的相关内容(参考文章第三篇)：

```
C99 6.3.2.1 Lvalues, arrays, and function designators 中第三段是这样说的：

Except when it is the operand of the sizeof operator or the unary & operator, or is a
string literal used to initialize an array, an expression that has type ‘‘array of type’’ is
converted to an expression with type ‘‘pointer to type’’ that points to the initial element of
the array object and is not an lvalue. If the array object has register storage class, the
behavior is undefined.

译：除了在使用sizeof和&运算符或者使用字符串字面量初始化数组之外，一个含有数组名的表达式会转化为含有指向首元素的表达式，并且转化后不是一个左值（这也是为什么我们不能修改这个标志符，例如val++，所以有的人也会说数组名是一个const指针，从本质上说这也是错的）。如果数组的存储类型是寄存器的话，行为是未定义的。（估计也没人这么做吧。。）
```

数组名不是首地址指针，数组名是一个标识符，它标识出我们之前申请的一连串内存空间，而且这个空间内的元素类型是相同的——**即数组名代表的是一个内存块及这个内存块中的元素类型** 。**只是在大多数情况下数组名会“退化”（C标准使用的decay和converted这两个词）为指向第一个元素的指针。**

这里也解释了这个：

```
void func(int* );
int main(){
	a[10];
	printf("%d \n", sizeof(a));//40(sizeof是运算符，不是函数)
	func(a);
}

func(int a[]){
	printf("%d \n", sizeof(a));//8(64位os下)
}
```

对于数组指针(int (*p) [5]的解释：

p表示的是一个**指向一个元素类型为整数，元素个数为5的内存块的指针**

到这里，虽然他们的数据内容一致，但还要注意一下数据的类型：
在数组上尤其是，因为数组指针的操做平时还是有用到的，函数指针除了用来引用函数，也不会去做一些类似数组指针的操作，而且有些操做也不允许。。。。。