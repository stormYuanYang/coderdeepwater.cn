函数[**strncmp**](http://www.cplusplus.com/reference/cstring/strncmp/)类似strcmp，可以视作strcmp的升级版本，按字典序对字符串进行大小比较。其声明如下：

```c
int strncmp ( const char * str1, const char * str2, size_t num );
```

strncmp的返回值类型是int，不同返回值有不同含义（同strcmp一致）：

- 返回值小于0时，str1小于str2；
- 返回值等于0时，str1等于str2;
- 返回值大于0时，str1大于str2。

strncmp比较str1和str2这两个字符串，但是不超过num个字符：

+ num大于str1和str2中任意一个的长度（包括'\0'）时；strncmp就等价于strcmp；
+ 否则，就只会比较部分字符，而不是完整地比较str1和str2。

---

其实现如下（string/strncmp.c）：

```c
/* Copyright (C) 1991, 1996, 1997, 2003 Free Software Foundation, Inc.
   This file is part of the GNU C Library.*/
/* Compare no more than N characters of S1 and S2,
   returning less than, equal to or greater than zero
   if S1 is lexicographically less than, equal to or
   greater than S2.  */
int strncmp (s1, s2, n)
    const char *s1;
    const char *s2;
    size_t n;
{
    unsigned reg_char c1 = '\0';
    unsigned reg_char c2 = '\0';

    if (n >= 4) {
        size_t n4 = n >> 2;
        do {
            c1 = (unsigned char) *s1++;
            c2 = (unsigned char) *s2++;
            if (c1 == '\0' || c1 != c2)
                return c1 - c2;
            c1 = (unsigned char) *s1++;
            c2 = (unsigned char) *s2++;
            if (c1 == '\0' || c1 != c2)
                return c1 - c2;
            c1 = (unsigned char) *s1++;
            c2 = (unsigned char) *s2++;
            if (c1 == '\0' || c1 != c2)
                return c1 - c2;
            c1 = (unsigned char) *s1++;
            c2 = (unsigned char) *s2++;
            if (c1 == '\0' || c1 != c2)
                return c1 - c2;
        } while (--n4 > 0);
        n &= 3;
    }

    while (n > 0) {
        c1 = (unsigned char) *s1++;
        c2 = (unsigned char) *s2++;
        if (c1 == '\0' || c1 != c2)
            return c1 - c2;
        n--;
    }

    return c1 - c2;
}
```

strncmp的实现还是挺简单的。

首先看`if (n >= 4) {}`，这里做了一定优化：以4个字符为一组进行比较，这样可以减少3/4的`--n`和`n==0`判断，可以提升一定性能（其思想已在[**《strncpy》**](https://coderdeepwater.cn/c_cpp/stdlibc/string_h/strncpy/)中详细分析，不再赘述)。执行完`if(n>=4){}`，可能还有剩余的字符尚未比较，所以用`while (n>0){}`循环比较即可。

---

一些说明：

+ **reg_char**：已在[**《strcpy》**](https://coderdeepwater.cn/c_cpp/stdlibc/string_h/strcpy/)中解释，不再赘述。