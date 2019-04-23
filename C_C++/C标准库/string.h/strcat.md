函数**strcat**的作用是将两个**C字符串**连接起来。其声明如下：

```c
char * strcat (char * dest, const char * src);
```

将src指向的C字符串连接到dest指向的C字符串后面,会覆盖掉dest处的C字符串的终止符'\0'。调用strcat需要调用者保证：

+ src指向合法的C字符串；
+ dest指向的字符数组已经有一个合法C字符串；
+ dest指向的字符数组的容量足够容纳两个C字符串（dest指向的字符数组容量必须大于等`strlen(dest)+strlen(src)+1`）。

当连接完成后，返回dest。

---

分析strcat的实现（string/strcat.c）：

```c
/* Copyright (C) 1991, 1997, 2003 Free Software Foundation, Inc.
   This file is part of the GNU C Library.
*/
/* Append SRC on the end of DEST.  */
char* strcat (dest, src)
    char *dest;
    const char *src;
{
    char *s1 = dest;
    const char *s2 = src;
    reg_char c;

    /* Find the end of the string.  */
    do {
        c = *s1++;
    } while (c != '\0');

    /* Make S1 point before the next character, so we can increment
       it while memory is read (wins on pipelined cpus).  */
    s1 -= 2;

    do {
        c = *s2++;
        *++s1 = c;
    }
    while (c != '\0');

    return dest;
}
```

strcat实现没什么复杂的地方：

+ 第一步，遍历dest，将s1移动到原字符串的末尾；
+ 第二部，遍历src，依次将s2指向的字符拷贝到s1。

这里需要注意的是`s1 -= 2;`，这样做的目的，上述代码注释已经解释：是为了在执行流水线方式指令的CPU时获得更好的读取性能。但是具体是如何提升的性能，我就不太清楚了，如果你知道，还请告知:-)。

分析strcat的时间复杂度。根据上文很容易得出，strcat的最坏时间复杂度为:
$$
O(n)
$$

---

一些说明：

- **reg_char**：已在[**《strcpy》**](https://coderdeepwater.cn/c_cpp/stdlibc/string_h/strcpy/)中解释，不再赘述。