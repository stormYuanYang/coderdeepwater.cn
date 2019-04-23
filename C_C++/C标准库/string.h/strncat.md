**strncat**和strcat功能一致，对字符串进行连接。其声明如下：

```c
char * strncat ( char * str1, const char * str2, size_t num );
```

strncat将str2中不超过num个字符**追加**到str1字符串的后面(str1的'\0'会被覆盖掉，然后在拼接最后添加'\0')：

+ num < strlen(str2),只会拷贝部分str2中的字符到str1的末尾，最后添加上终止符'\0'；
+ num >= strlen(str2),会拷贝完整的str2中的字符到str1的末尾，最后同样会加上终止符'\0'。

总体来说，strncat比strcat功能更强大：可以选择拷贝部分字符，而不是全部字符。和strcat一样，strncat也要保证str1指向的字符数数组的容量要足够大。

---

strncat的实现（string/strncat.c）：

```c
/* Copyright (C) 1991, 1997 Free Software Foundation, Inc.
   This file is part of the GNU C Library.
*/
char * strncat (s1, s2, n)
    char *s1;
    const char *s2;
    size_t n;
{
    reg_char c;
    char *s = s1;

    /* Find the end of S1.  */
    do
        c = *s1++;
    while (c != '\0');

    /* Make S1 point before next character, so we can increment
       it while memory is read (wins on pipelined cpus).  */
    s1 -= 2;

    if (n >= 4) {
        size_t n4 = n >> 2;
        do {
            c = *s2++;
            *++s1 = c;
            if (c == '\0')
                return s;
            c = *s2++;
            *++s1 = c;
            if (c == '\0')
                return s;
            c = *s2++;
            *++s1 = c;
            if (c == '\0')
                return s;
            c = *s2++;
            *++s1 = c;
            if (c == '\0')
                return s;
        } while (--n4 > 0);
        n &= 3;
    }

    while (n > 0) {
        c = *s2++;
        *++s1 = c;
        if (c == '\0')
            return s;
        n--;
    }

    if (c != '\0')
        *++s1 = '\0';

    return s;
}
```

strncat先遍历s1,找到s1的末尾。接下来是一个优化，以4个字符为一组进行拷贝，这样可以减少3/4的`--n`操作和`n>0`的判断（其思想已在[**《strncpy》**](https://coderdeepwater.cn/c_cpp/stdlibc/string_h/strncpy/)中详细分析，不再赘述）。4个字符一组拷贝完后，还剩下字符的话，继续拷贝剩下的字符。最后的if判断：

```c
if (c != '\0')
    *++s1 = '\0';
```

这段代码的目的是：当`n <= strlen(s2)`时，在s1字符串的最后添加上'\0'，以保证其正确性。

---

一些说明：

- **reg_char**：已在[**《strcpy》**](https://coderdeepwater.cn/c_cpp/stdlibc/string_h/strcpy/)中解释，不再赘述。