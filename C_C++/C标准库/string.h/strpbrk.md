**strpbrk**是一个字符串查找函数，其声明如下：

```c
char * strpbrk ( const char * s, const char * accept); 
```

strpbrk接受两个合法的C字符串。strpbrk的目的是找到accept的任意一个字符在s中首次出现的位置。

---

strpbrk的实现还是挺简单的（string/strpbrk.c）：

```c
/* Copyright (C) 1991, 1994, 1996, 1997, 2003 Free Software Foundation, Inc.
   This file is part of the GNU C Library.*/
/* Find the first occurrence in S of any character in ACCEPT.  */
char * strpbrk (s, accept)
    const char *s;
    const char *accept;
{
    while (*s != '\0') {
        const char *a = accept;
        while (*a != '\0')
            if (*a++ == *s)
                return (char *) s;
        ++s;
    }
    return NULL;
}
```

strpbrk的代码非常直观，就只有两层嵌套循环：外层循环遍历s中的每一个字符，内循环遍历accept中的字符，如果能在accept中找到当前字符***s**，则返回s；否则继续检查s中的下一个字符。如果遍历完s都找不到一个字符是accept中的字符，则返回NULL。

显而易见，strpbrk的时间复杂度是平方级的：
$$
O(n^{2})
$$
当然啦，一般使用这个函数的时候accept中的字符都只有常数个（多数情况下只有1个）。从实际的运用来看，其时间复杂度可以认为是线性的：
$$
O(n)
$$
