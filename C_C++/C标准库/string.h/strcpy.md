[**strcpy**](http://www.cplusplus.com/reference/cstring/strcpy/)即stringcopy的缩写，自如其名，其作用是对**C字符串**(以'**\0'**结尾的字符串)进行拷贝(也会拷贝'\0')。其声明在string.h中：

```c
char * strcpy (char * dest, const char * src);
```

strcpy将str指向的C字符串拷贝到dest。strcpy在使用时需要注意：

+ strcpy并未对dest指向的内存大小做判断，拷贝C字符串到dest时可能会导致**overflow**，所以需要调用者保证dest指向的内存大小足够放置C字符串；
+ strcpy只能对C字符串做安全的拷贝。如果拷贝的字符串不是以'\0'结尾，则可能会导致**overflow**，因为strcpy只有拷贝了'\0'才会停止。

现在我们看strcpy的源码：string/strcpy.c

```c
/* Copyright (C) 1991, 1997, 2000, 2003 Free Software Foundation, Inc.
   This file is part of the GNU C Library.
  */
/* Copy SRC to DEST.  */
    char *
strcpy (dest, src)
    char *dest;
    const char *src;
{
    reg_char c;
    char *__unbounded s = (char *__unbounded) CHECK_BOUNDS_LOW (src);
    const ptrdiff_t off = CHECK_BOUNDS_LOW (dest) - s - 1;
    size_t n;

    do {
        c = *s++;
        s[off] = c;
    } while (c != '\0');

    n = s - src;
    (void) CHECK_BOUNDS_HIGH (src + n);
    (void) CHECK_BOUNDS_HIGH (dest + n);

    return dest;
}
```

现在对上述源码进行分析。排除宏定义的干扰，分析strcpy还是挺轻松的。strcpy先求得`dest`和`s`之间的偏移量`off`，然后使用do{}while();将`src`中的每一个字符拷贝拷贝到`dest`中，直到拷贝完字符'\0'，循环停止。然后返回`dest`。

其实strcpy可以简单的写成:

```c
char* strcpy(char* dest, const char* src) {
    char* tmp = dest;
    while (*tmp++ = *src++)
        ;
    return dest;
}
```

这样写，可能会多做一些运算和存储。但是对于现代C编译器来说，这不是问题，C编译器会尽可能的优化的。

使用strcpy很容易造成overflow，应尽量使用strncpy替代。

---

一些说明：

+ reg_char意指register char，即

  ```c
  // 具体是哪个宏定义取决于你的编译环境
  #define reg_char char
  // 或者
  #define reg_char register char
  ```

  使用register修饰char，目的是尽可能地加快存取变量c的速度。

+ `__unbounded`是一个宏定义，其定义可能是

  ```c
  // nothing
  #define __unbounded
  ```

  也可能是其他文本，这取决于你的编译器。

+ `CHECK_BOUNDS_LOW`和`CHECK_BOUNDS_LOW`均是宏定义。不同的平台有不同实现，这里给出一般实现(sysdeps/generic/bp-checks.h)：

  ```c
  /* Verify that pointer's value >= low.  Return pointer value.  */                  
  # define CHECK_BOUNDS_LOW(ARG)                  \
    (((__ptrvalue (ARG) < __ptrlow (ARG)) && BOUNDS_VIOLATED),    \
     __ptrvalue (ARG)) 
  /* Verify that pointer's value < high.  Return pointer value.  */
  # define CHECK_BOUNDS_HIGH(ARG)             \
    (((__ptrvalue (ARG) > __ptrhigh (ARG)) && BOUNDS_VIOLATED),   \
     __ptrvalue (ARG))
  ```

  `CHECK_BOUNDS_LOW`和`CHECK_BOUNDS_LOW`检查`ARG`指针的值。具体是如何检查和校验，我也不太清楚:-)，在glibc目录下也未搜索到__ptrlow的宏定义。如果你知道，还请告知。