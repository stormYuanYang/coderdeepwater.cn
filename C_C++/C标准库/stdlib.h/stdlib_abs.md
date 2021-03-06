函数[**abs**](http://www.cplusplus.com/reference/cstdlib/abs/)返回参数的绝对值。

其声明如下：

```c
int abs(int n);
```

这个函数的实现本身非常简单，让我们直接看源代码。abs的实现放在stdlib/abs.c中：

```c
/* Copyright (C) 1991, 1997 Free Software Foundation, Inc.
   This file is part of the GNU C Library.
*/
#include <stdlib.h>
#undef	abs
/* Return the absolute value of I.  */
int 
abs (int i) {
    return i < 0 ? -i : i;
}
```

abs的实现就一行代码。简单地判断i和0的大小，如果i小于0，直接返回-i，否则返回i本身。

有一个陷阱需要注意：当参数i是[**INT_MIN**](http://www.cplusplus.com/reference/climits/)时，即int的最小值时，这个函数就无法正确获取i的绝对值了。

```c
#include <stdio.h>
#include <limits.h>
#include <stdlib.h>

int main() {
    int a = 1 << 31;//如果int是32位的话，1<<31就是最小值
    int b = INT_MIN;
    int c = INT_MIN + 1;
    printf("abs(a):%d\tabs(b):%d\tabs(c):%d\n", abs(a), abs(b), abs(c));
    return 0;
}
```

执行上述代码，在我的计算机上打印结果如下：

```c
abs(a):-2147483648   abs(b):-2147483648  abs(c):2147483647
```

是不是感到很奇怪，怎么a、b的绝对值打印出来还是个负数而且就是其本身。**signed**类型的整数在计算机内存中采用2进制的**补码**表示。其首位是**符号位**：

| 符号位值 | 整数类型 |
| :------: | :------: |
|    0     |   正数   |
|    1     |   负数   |

剩下的位数则表示整数的值。以[**int8_t**](http://www.cplusplus.com/reference/cstdint/)(一个字节，8位)为例构造几个例子：

|          10进制          | 2进制补码 |
| :----------------------: | :-------: |
|            1             | 0000,0001 |
| 255(int8_t类型的最大值)  | 0111,1111 |
|            0             | 0000,0000 |
|            -1            | 1111,1111 |
|           -255           | 1000,0001 |
| -256(int8_t类型的最小值) | 1000,0000 |

由上表，你大概能推测出，为什么abs(INT_MIN)不能得到正确的绝对值了。因为在同一signed类型中，根本表示不了INT_MIN的绝对值。如果是int8_t，那么int8_t的最小值的绝对值256没有办法在int8_t中表示出来。至于为什么int8_t表示不了256，如果你记不清了，可以复习一下大学里的相关课程，特别是《数字电子技术基础》。

既然表示不了INT_MIN的绝对值，为什么返回值还是INT_MIN本身呢？这就涉及到计算机内部的求相反数的运算规则了(取决于你计算机的具体实现,也有其他的方法)：

+ 步骤1：首先将变量所有位**取反**(比如对1000,000取反，得到0111,1111)；
+ 步骤2：然后用1加上取反后的值;
+ 步骤3：步骤2的结果即是相反数。

同样的，我构造几个例子:

| 步骤/整数(二进制表示) |       -255(即1000,0001)        |       255(即0111,1111)       |      -256(即1000,0000)       |
| :-------------------: | :----------------------------: | :--------------------------: | :--------------------------: |
|         步骤1         | 对-255取反得到254(即0111,1110) | 对255取反得到-256(1000,0000) | 对-256取反得255(即0111,1111) |
|         步骤2         |    1+254得255(即0111,1111)     | 1+(-256)得-255(即1000,0001)  |   1+255得-256(即1000,0000)   |
|         步骤3         |         得到结果：255          |        得到结果:-255         |        得到结果：-256        |

由上表可知，传给abs的参数是INT_MIN时，返回的结果必然还是INT_MIN。