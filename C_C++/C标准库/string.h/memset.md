函数[**memset**](http://www.cplusplus.com/reference/cstring/memset/)对指定的数组进行填充，其声明如下：

```c
void* memset(void* ptr, int value, size_t num);
```

memset的使用倒也简单，但是仍然有需要注意的地方：

+ ptr指向的数组能否存储num个字节需要调用者保证，memset不做任何检查；
+ 虽然指定的填充值value类型是int,但是memset是按**字节**为单位进行填充的。当value的值大于**255**时，一个字节就无法存储value了，此时value就会被截断，也就是说你指定的value和实际填充的value一定不同。我想当初设计memset的人是为了让memset更容易调用，毕竟int是我们使用得最多的整数类型，不然每次指定填充值还需要进行强制转换，还是有些麻烦。暂且不去评价这样的做法是否合理，既然其实现已是如此，作为调用者，我们就应当清楚明了memset的陷阱。

让我们分析其源码(string/memset.c):

```c
/* Copyright (C) 1991, 1997, 2003 Free Software Foundation, Inc.
   This file is part of the GNU C Library.
 */
void* memset(dstpp, c, len)
    void *dstpp;
    int c;
    size_t len; 
{
    long int dstp = (long int) dstpp;
    if (len >= 8) {
        size_t xlen;
        op_t cccc;

        cccc = (unsigned char) c;
        cccc |= cccc << 8;
        cccc |= cccc << 16;
        if (OPSIZ > 4) {
            /* Do the shift in two steps to avoid warning if long has 32 bits.  */
            cccc |= (cccc << 16) << 16;
        }

        /* There are at least some bytes to set.
           No need to test for LEN == 0 in this alignment loop.  */
        while (dstp % OPSIZ != 0) {
            ((byte *) dstp)[0] = c;// 如果c大于255，赋的值就会被截断
            dstp += 1;
            len -= 1;
        }

        /* Write 8 `op_t' per iteration until less than 8 `op_t' remain.  */
        xlen = len / (OPSIZ * 8);
        while (xlen > 0)
        {
            ((op_t *) dstp)[0] = cccc;
            ((op_t *) dstp)[1] = cccc; ((op_t *) dstp)[2] = cccc;
            ((op_t *) dstp)[3] = cccc;
            ((op_t *) dstp)[4] = cccc;
            ((op_t *) dstp)[5] = cccc;
            ((op_t *) dstp)[6] = cccc;
            ((op_t *) dstp)[7] = cccc;
            dstp += 8 * OPSIZ;
            xlen -= 1;
        }
        len %= OPSIZ * 8;// 求得剩余的字节长度

        /* Write 1 `op_t' per iteration until less than OPSIZ bytes remain.  */
        xlen = len / OPSIZ;
        while (xlen > 0)
        {
            ((op_t *) dstp)[0] = cccc;
            dstp += OPSIZ;
            xlen -= 1;
        }
        len %= OPSIZ;
    }

    /* Write the last few bytes.  */
    while (len > 0)
    {
        ((byte *) dstp)[0] = c;
        dstp += 1;
        len -= 1;
    }
    return dstpp;
}
```

先看函数第一行代码：

```c
long int dstp = (long int) dstpp;
```

这里将dstpp强制转换成long int并赋值给dstp。为什么不是强制转换成int并赋值呢？是为了确保dstp能完整的保存指针dstpp的所有信息。

在看`if (len >= 8){}`，这里做了一个判断：如果len的小于8，直接将写入剩下的字节;如果len>=8则进入if语句执行。继续往下看：

```c
size_t xlen;
op_t cccc;

cccc = (unsigned char) c;// c会被截断
cccc |= cccc << 8;
cccc |= cccc << 16;
if (OPSIZ > 4) {
    /* Do the shift in two steps to avoid warning if long has 32 bits.  */
    cccc |= (cccc << 16) << 16;
}
```

上述代码很好理解：为了提升写入的效率，以**word**为单位进行写入，所以这里声明了变量cccc。通过位运算，将被截断的c（一个字节）填充进到cccc的每一个字节中（所以才叫cccc，很直观的）。至于这里的`if (OPSIZ > 4){}`的作用是处理64位操作系统（word占8字节）下的word填充。这里为什么不是直接位移32位呢？是因为在32系统中出现位移32位的代码，编译器会发出警告，所以这里分两次进行位移。

接着往下看：

```c
while (dstp % OPSIZ != 0) {
    ((byte *) dstp)[0] = c;// 如果c大于255，赋的值就会被截断
    dstp += 1;
    len -= 1;
}
```

这个`while (dstp % OPSIZ != 0){}`循环的目的又是什么呢？看起来很奇怪，dstp代表的不是内存地址吗，为什么会对dstp求余呢？其实这个很好解释：咱们的计算机系统在寻址时，为了提升寻址速度，一般都会按**aligned address**(即对齐的地址)寻址。举个例子，64位操作系统以8个字节对齐地址。对于64位操作系统来说，寻址0、8、16、24、32等等是非常方便的。所以呢，为了加快寻址速度，`while (dstp % OPSIZ != 0){}`就将未对齐的部分先写入被截断的c。

再看：

```c
xlen = len / (OPSIZ * 8);
while (xlen > 0)
{
    ((op_t *) dstp)[0] = cccc;
    ((op_t *) dstp)[1] = cccc; 
    ((op_t *) dstp)[2] = cccc;
    ((op_t *) dstp)[3] = cccc;
    ((op_t *) dstp)[4] = cccc;
    ((op_t *) dstp)[5] = cccc;
    ((op_t *) dstp)[6] = cccc;
    ((op_t *) dstp)[7] = cccc;
    dstp += 8 * OPSIZ;
    xlen -= 1;
}
len %= OPSIZ * 8;// 求得剩余的字节长度
```

这段代码进一步地做了优化。同样的思想在[**strncpy**](https://coderdeepwater.cn/c_cpp/stdlibc/string_h/strncpy/)中亦有采用,这里不再赘述。以8个op_t为一组进行拷贝，可减少7/8的`--len`和`len > 0`的判断，进一步提升执行效率。执行完这段代码，还可能剩余一些内容未写入。继续看：

```c
xlen = len / OPSIZ;
while (xlen > 0)
{
    ((op_t *) dstp)[0] = cccc;
    dstp += OPSIZ;
    xlen -= 1;
}
len %= OPSIZ;
```

这段代码将剩余的内容（一定小于8个op_t），以op_t为单位进行填充。最后求得剩余的字节数len。至此，`if (len >= 8) {}`就执行完毕。

最后看：

```c
while (len > 0)
{
    ((byte *) dstp)[0] = c;
    dstp += 1;
    len -= 1;
}
```

如果执行完`if (len >= 8) {}`,还有剩余的字节（len > 0）,则简单地执行`while(len>0){}`对剩余的字节进行填充。

最后，全部字节填充完毕，返回指针。

---

一些说明：

+ **op_t**和**OPSIZ**（sysdeps/generic/memcopy.h）：

  ```c
  /* Type to use for aligned memory operations.
     This should normally be the biggest type supported by a single load
     and store. */
  #define op_t    unsigned long int
  #define OPSIZ   (sizeof(op_t))
  ```

+ **byte**（sysdeps/generic/memcopy.h）：

  ```c
  /* Type to use for unaligned operations.  */
  typedef unsigned char byte;
  ```

  

