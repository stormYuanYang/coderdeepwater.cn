上回说到[**strcpy**](http://www.cplusplus.com/reference/cstring/strcpy/)：对C字符串进行拷贝。今天介绍的[**strncpy**](http://www.cplusplus.com/reference/cstring/strncpy/)和strcpy功能一致，同样是对C字符串进行拷贝。你可以将strncpy视作strcpy的升级版，一般而言，strncpy本身比strcpy更快、更安全。strncpy的声明如下：

```c
char * strncpy(char * dest, const char * src, size_t num);
```

参数num限制了strncpy从src拷贝到dest的字符数量上限，即最多拷贝num个字符。有了num参数，strncpy比起strcpy就更加安全，但是strncpy和strcpy一样没有判断dest指向的字符数组的容量是否能容纳拷贝内容，这个需要调用者去负责。但是strncpy有一个问题却是strcpy没有的，strcpy能完整地拷贝C字符串，而当指定的字符串长度num小于字符串本身的长度（包括字符'\0'）时，拷贝的字符串就不是一个完整的C字符串了，这个时候就要小心了：dest指向的字符串可能并没有终止符'\0'。

简单地总结strncpy的作用：

+ 当`num < strlen(src) + 1`（即num小于整个字符串长度(包括'\0')）时：strncpy只会拷贝num个字符，未拷贝终止符'\0'；
+ 当`num == strlen(src) + 1`时：此时strncpy和strcpy等价，恰好将字符串拷贝到dest。
+ 当`num > strlen(src) + 1`时：此时strncpy会完整地将字符串拷贝到dest，并且填充`num - strlen(src) - 1`个'\0'到dest字符之后。

---

直接分析源码(string/strncpy.c)：

```c
/* Copyright (C) 1991, 1997, 2003 Free Software Foundation, Inc.
   This file is part of the GNU C Library.
*/
char* strncpy (s1, s2, n)
    char *s1;
    const char *s2;
    size_t n;
{
    reg_char c;
    char *s = s1;
    --s1;
    if (n >= 4)
    {
        // 如果以四个字符为单位,这个单位叫做a,则n4表示有多少个a
        size_t n4 = n >> 2;
        for (;;)
        {
            c = *s2++;
            *++s1 = c;
            if (c == '\0')
                break;
            c = *s2++;
            *++s1 = c;
            if (c == '\0')
                break;
            c = *s2++;
            *++s1 = c;
            if (c == '\0')
                break;
            c = *s2++;
            *++s1 = c;
            if (c == '\0')
                break;
            if (--n4 == 0)
                goto last_chars;
        }
        n = n - (s1 - s) - 1;
        if (n == 0) {
            return s;
        }
        goto zero_fill;
    }
last_chars:
    n &= 3;
    if (n == 0)
        return s;
    do
    {
        c = *s2++;
        *++s1 = c;
        if (--n == 0)
            return s;
    }
    while (c != '\0');
zero_fill:
    do {
        *++s1 = '\0';
    } while (--n > 0);
    return s;
}
```

我们先分析`if (n >= 4){}`语句：

```c
if (n >= 4) {
    // 如果以四个字符为单位,这个单位叫做a,则n4表示有多少个a
    size_t n4 = n >> 2;
    for (;;) {
        c = *s2++;
        *++s1 = c;
        if (c == '\0')
            break;
        c = *s2++;
        *++s1 = c;
        if (c == '\0')
            break;
        c = *s2++;
        *++s1 = c;
        if (c == '\0')
            break;
        c = *s2++;
        *++s1 = c;
        if (c == '\0')
            break;
        if (--n4 == 0)
            goto last_chars;
    }
    n = n - (s1 - s) - 1;
    if (n == 0) {
        return s;
    }
    goto zero_fill;
}
```

strcpy在拷贝字符串时，是一个一个字符拷贝的。strncpy虽然同样是一个一个字符地拷贝，但strncpy做了优化(正是因为有了参数n才能做此优化)：以4个字符为一组进行拷贝。注意，这句话并不意味着strncpy会一次拷贝4个字符，仍然是一个一个字符拷贝。以4个字符为一组的目的是减少参数n的判断次数：原本在最坏情况下，需要做n次`--n`操作和n次`n > 0`的判断；以4个字符一组进行拷贝，只在第四个字符拷贝时进行`--n4`操作和`n4 > 0`的判断，这样可以减少3/4的上述操作和判断。当拷贝的字符串较长时，上述优化提升的效率还是很可观的。这里的`for(;;){}`虽然写得比较冗余，但是考虑到性能上的提升，则是利大于弊的。

这里为什么要单独执行`--s1`？是为了使用`++s1`而不是`s1++`去移动指针。这样做，是为了进一步提升执行效率，因为执行`++s1`的效率高于`s1++`。为什么`++s1`的效率又要高于`s1++`呢？简单来说，`++s1`比起`s1++`少做做一次旧的s1存储，直接得到s1加1的结果。

读完`for(;;){}`，你可能会有这样的疑问：为什么不以8个字符甚至16个字符一组进行拷贝呢，这样的话可以减少7/8甚至15/16的上述操作和判断？是的，以8个字符甚至16个字符一组进行拷贝，执行效率肯定会更加高效，但是提升又有多少呢？看下表：

| 分组字符数 | 减少的--n和n>0执行次数百分比(以n为总数) |
| :--------: | :-------------------------------------: |
|     4      |                   %75                   |
|     8      |                  %87.5                  |
|     16     |                 %92.25                  |

4个字符一组的效率已经很高了，虽然以8个字符甚至16个字符一组效率能得到更好的提升，但是由上表结果来看，提升并不是很明显(%75和%87甚至%92.25比也差不了多少)，并且会导致代码的成倍增长、大量的冗余。这样肯定是不值得的，所以在执行效率和代码结构上做了一个权衡。有舍才有得，岂能面面俱到。

退出`for(;;){}`有两种情况：

1. 拷贝了**'\0'**，则退出循环；
2. `n4 == 0`即n4个字符组被拷贝完，则退出循环。

先看**1.**：此时，指定的n个字符可能还未被拷贝完就遇到了终止符'\0'，通过`n = n - (s1 - s) - 1;`获取还需拷贝的字符数量，如果`n == 0`，说明字符恰好拷贝完毕，则直接返回；如果`n > 0`，则代码跳转至`zero_fill`，继续拷贝n个'\0'到s1：

```c
zero_fill:
    do {
        *++s1 = '\0';
    } while (--n > 0);
    return s;
```



再看**2.**：此时，可能还有剩余的字符（一定不会超过4个字符），需要对剩余字符进行拷贝，所以代码跳转至`last_chars`：

```c
last_chars:
    n &= 3;
    if (n == 0)
        return s;
    do
    {
        c = *s2++;
        *++s1 = c;
        if (--n == 0)
            return s;
    }
    while (c != '\0');
```

在这里，`n &= 3;`等价于`n %= 4;`(注意，并不是说`n &= a`都等价与`n %= a`，这里a是一个整数；因为4是2的正整数幂，所以才能等价)，目的是通过求余运算，求得剩余的字符数量。至于为什么不写成容易理解的求余运算`n %= 4;`而是写成与运算`n &= 3;`？是出于效率上的考量，毕竟**与运算**比**求余运算**快很多。

如果剩余字符数量为0，则直接返回。如果剩余字符不为0，对剩余的字符进行拷贝：如果尚未拷贝完就遇到'\0'，在拷贝完'\0'后，仍然需要对剩余字符进行`zero_fill`；如果将剩余字符拷贝完都没有遇到'\0'，则直接返回。

---

一些说明：

+ **reg_char**：已在[**《strcpy》**](https://coderdeepwater.cn/c_cpp/stdlibc/string_h/strcpy/)中解释，不再赘叙。