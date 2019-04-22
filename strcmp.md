函数[**strcmp**](http://www.cplusplus.com/reference/cstring/strcmp/)对两个C字符串按**字典序**进行比较。其声明如下：

```c
int strcmp (const char * str1, const char * str2);
```

strcmp的返回值类型是int，不同返回值有不同含义：

+ 返回值小于0时，str1小于str2；
+ 返回值等于0时，str1等于str2;
+ 返回值大于0时，str1大于str2。

一定要注意，并不一定返回-1、0、1这三个值，我就曾经被人误导过。使用strcmp有一点需要注意：必须保证传入的str1和str2是合法的C字符串（以'\0'结尾的字符串），否则可能导致**overflow**。

---

让我们看其实现（string/strcmp.c）:

```c
/* Copyright (C) 1991, 1996, 1997, 2003 Free Software Foundation, Inc.
   This file is part of the GNU C Library.
*/

/* Compare S1 and S2, returning less than, equal to or
   greater than zero if S1 is lexicographically less than,
   equal to or greater than S2.  */
int strcmp (p1, p2)
    const char *p1;
    const char *p2; {
    register const unsigned char *s1 = (const unsigned char *) p1;
    register const unsigned char *s2 = (const unsigned char *) p2;
    unsigned reg_char c1, c2;

    do {
        c1 = (unsigned char) *s1++;
        c2 = (unsigned char) *s2++;
        if (c1 == '\0')
            return c1 - c2;
    } while (c1 == c2);

    return c1 - c2;
}
```

首先声明并定义了s1和s2。因为s1和s2由**register**修饰，在存取s1和s2时可能会更加高效。

接下来是一个`do{}while();`循环，对传入的两个字符串的字符一一做比较。这里的代码十分简单，没什么好详细说明的，要注意的就是：这里两处语句返回的都是`c1 - c2`，在使用strcmp的返回值时要注意。

---

一些说明：

+ **reg_char**：已在[**《strcpy》**](https://coderdeepwater.cn/c_cpp/stdlibc/string_h/strcpy/)中解释，不再赘叙。