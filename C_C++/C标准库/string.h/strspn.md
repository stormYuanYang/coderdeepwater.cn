**strspn**的声明如下：

```c
size_t strspn (const char * s, const char * accept);
```

strspn的目的，用标准库的注释来说明是再好不过了：

”**Return the length of the maximum initial segment of s which contains only characters in accept**。“

使用strspn需要注意：

+ 参数s和accept必须是合法的C字符串（必须有'\0'表示字符串的结束）。

---

分析strspn的实现（string/strspn.c）：

```c
/* Copyright (C) 1991, 1997, 2003 Free Software Foundation, Inc.
   This file is part of the GNU C Library.*/
/* Return the length of the maximum initial segment
   of S which contains only characters in ACCEPT.  */
size_t strspn (s, accept)
    const char *s;
    const char *accept;
{
    const char *p;
    const char *a;
    size_t count = 0;

    for (p = s; *p != '\0'; ++p) {
        for (a = accept; *a != '\0'; ++a)
            if (*p == *a)
                break;
        if (*a == '\0')
            return count;
        else
            ++count;
    }
    return count;
}
```

strspn的实现非常简单：两层嵌套循环。

strspn遍历字符数组s中的字符，对当前访问的字符进行判断；如果当前字符存在于字符数组accept中，则计数count加1；否则返回已有计数count。

strspn运行的时间复杂度是多少呢？就源代码而言，很容易得出strspn的时间复杂度为：
$$
O(n^{2})
$$
但一般使用此函数时，accept的长度是常数且比较小；比如你使用strspn跳过字符串中的前导空白符；此时strspn的时间复杂度可认为是：
$$
O(n)
$$
