# 零、前言

大名鼎鼎的**GCC**在glibc中，为原子操作提供了一系列**built-in functions**。这些函数以较底层的汇编代码编写，减轻了系统、应用级别的加锁带来的性能损耗。因为是内建的函数，在使用时不需要包含标准库的头文件；下面简单介绍这些函数。

# 一、n++类操作

```c
type __sync_fetch_and_add (type *ptr, type value, ...)
type __sync_fetch_and_sub (type *ptr, type value, ...)
type __sync_fetch_and_or (type *ptr, type value, ...)
type __sync_fetch_and_and (type *ptr, type value, ...)
type __sync_fetch_and_xor (type *ptr, type value, ...)
type __sync_fetch_and_nand (type *ptr, type value, ...)
```

上述函数用伪代码可以表示为：

```c
{ // add sub or and xor
    tmp = *ptr; 
    *ptr op= value; //op表示操作，比如+、-、&、|、^等
    return tmp; 
}
{ // nand 即 not and，对and之后的值取反
    tmp = *ptr; 
    *ptr = ~(tmp & value); 
    return tmp; 
}   
```

对其一一说明：

+ **__sync_fetch_and_add**：原子加法，\*ptr + value获得新值，但返回旧的\*ptr；
+ **__sync_fetch_and_sub**：原子减法，*ptr - value获得新值，但返回旧的\*ptr；
+ **__sync_fetch_and_or**：原子或运算，\*ptr | value获得新值，但返回旧的*ptr；
+ **__sync_fetch_and_and**：原子与运算，*ptr & value获得新值，但返回旧的\*ptr;
+ **__sync_fetch_and_xor**：原子亦或运算，*ptr ^ value获得新值，但返回旧的\*ptr;
+ **__sync_fetch_and_nand**：原子not and运算，先与运算再取反获得新值，但返回旧的\*ptr。

测试代码（非多线程环境，只是演示如何使用原子操作）：

```c
#include <stdio.h>
int main() {
    int a = 0;
    // __sync_fetch_and_add
    printf("%d ", __sync_fetch_and_add(&a, 1));
    printf("%d\n", a);

    // __sync_fetch_and_sub
    printf("%d ", __sync_fetch_and_sub(&a, 1));
    printf("%d\n", a);

    // __sync_fetch_and_or
    printf("%d ", __sync_fetch_and_or(&a, 1));
    printf("%d\n", a);
    
    // __sync_fetch_and_and
    printf("%d ", __sync_fetch_and_and(&a, 0));
    printf("%d\n", a);
 
    // __sync_fetch_and_xor
    printf("%d ", __sync_fetch_and_xor(&a, 1));
    printf("%d\n", a);

    // __sync_fetch_and_nand
    // 1 & 1 --> 1
    // not 1 --> 1111...1110
    // 最终得到-2
    printf("%d ", __sync_fetch_and_nand(&a, 1));
    printf("%d\n", a);
    return 0;
}
```

# 二、++n类操作

```c
type __sync_add_and_fetch (type *ptr, type value, ...)
type __sync_sub_and_fetch (type *ptr, type value, ...)
type __sync_or_and_fetch (type *ptr, type value, ...)
type __sync_and_and_fetch (type *ptr, type value, ...)
type __sync_xor_and_fetch (type *ptr, type value, ...)
type __sync_nand_and_fetch (type *ptr, type value, ...)
```

上述函数用伪代码表示：

```c
{// add sub or and xor nand
    *ptr op= value; 
    return *ptr; 
}
{// nand 即not and
    *ptr = ~(*ptr & value); 
    return *ptr; 
}
```

对其一一说明：

+ **__sync_add_and_fetch**：原子加法，\*ptr + value获得新值，并返回新值；
+ **__sync_sub_and_fetch**：原子减法，\*ptr - value获得新值，并返回新值；
+ **__sync_or_and_fetch**：原子或运算，\*ptr | value获得新值，并返回新值；
+ **__sync_and_and_fetch**：原子与运算，*ptr & value获得新值，并返回新值;
+ **__sync_xor_and_fetch**：原子亦或运算，*ptr ^ value获得新值，并返回新值;
+ **__sync_nand_and_fetch**：原子not and运算，先与运算再取反获得新值，并返回新值。

测试代码：

```c
#include <stdio.h>

int main() {
    int a = 0;
    // __sync_add_and_fetch
    printf("%d ", __sync_add_and_fetch(&a, 1));
    printf("%d\n", a);

    // __sync_sub_and_fetch
    printf("%d ", __sync_sub_and_fetch(&a, 1));
    printf("%d\n", a);

    // __sync_or_and_fetch
    printf("%d ", __sync_or_and_fetch(&a, 1));
    printf("%d\n", a);
    
    // __sync_and_and_fetch
    printf("%d ", __sync_and_and_fetch(&a, 0));
    printf("%d\n", a);
 
    // __sync_xor_and_fetch
    printf("%d ", __sync_xor_and_fetch(&a, 1));
    printf("%d\n", a);

    // __sync_nand_and_fetch
    // 1 & 1 --> 1
    // not 1 --> 1111...1110
    // 最终得到-2
    printf("%d ", __sync_nand_and_fetch(&a, 1));
    printf("%d\n", a);
    return 0;
}
```

# 三、CAS

```c
bool __sync_bool_compare_and_swap (type *ptr, type oldval type newval, ...)
type __sync_val_compare_and_swap (type *ptr, type oldval type newval, ...)
```

上述函数，执行原子的比较和交换（即**CAS**）：如果当前的\*ptr等于oldval,则将newval写入到\*ptr。

说明：

+ **__sync_bool_compare_and_swap**：返回真，如果比较成功，且newval被写入；
+ **__sync_val_compare_and_swap**：返回进行CAS操作之前的*ptr。

测试代码：

```c
#include <stdio.h>
int main() {
    int a = 0;
    // a: 0 --> 9(can)
    printf("%s ", __sync_bool_compare_and_swap(&a, 0, 9) ? "yes" : "no");
    printf("%d\n", a);
    //a: 9 --> 0 (can not, 因为a != 5)
    printf("%s ", __sync_bool_compare_and_swap(&a, 5, 0) ? "yes" : "no");
    printf("%d\n", a);
 
    a = 0;
    // success
    printf("old:%d ", __sync_val_compare_and_swap(&a, 0, 9)); 
    printf("new:%d\n", a);
    // fail,因为a != 5
    printf("old:%d ", __sync_val_compare_and_swap(&a, 5, 0)); 
    printf("new:%d\n", a);

    return 0;
}
```

# 四、lock_test_and_set

```c
type __sync_lock_test_and_set (type *ptr, type value, ...)
```

**__sync_lock_test_and_set**将value写入\*ptr，返回旧的\*ptr。

测试代码：

```c
#include <stdio.h>
int main() {
    int a = 0;
    // a:0 --> 6
    printf("%d\n", __sync_lock_test_and_set(&a, 6));
    printf("%d\n", a);
    return 0;
}
```

# 五、lock_release

```c
void __sync_lock_release (type *ptr, ...)
```

**__sync_lock_release**可以释放**__sync_lock_test_and_set**获得的锁。一般将0写入\*ptr。

测试代码：

```c
#include <stdio.h>

int main() {
    int a = 0;
    // a:0 --> 6
    printf("%d\n", __sync_lock_test_and_set(&a, 6));
    printf("%d\n", a);
    // release 
    __sync_lock_release(&a);
    printf("%d\n", a);
    return 0;
}
```

# 注意

上文提到的原子操作在新的标准库版本中支持C++11的内存模型。

上文提到的原子操作在较新的版本中，前缀由**__sync**变更为**__atomic**，并且去掉了中间的and，比如**__sync_fetch_and_add**变为**__atomic_fetch_add**；并且在使用上有细微差别。

上文提到的原子操作支持大小为1,2,4或者8字节的完整的变量和指针（如果__int128被支持，原子操作可以支持16字节的变量）。

上文提供的测试代码，请使用gcc进行编译。

在**gcc4.4**之前的版本，**__sync_fetch_and_nand**是先对\*ptr的tmp值取反，再执行与运算（步骤是相反的）：

```c
{ // GCC4.4之前 nand 即 not and
    tmp = *ptr; 
    *ptr = ~tmp & value;// 先取反，再求与 
    return tmp; 
}   
```

在**gcc4.4**之前的版本，**__sync_nand_and_fetch**是先对\*ptr的tmp值取反，再执行与运算（步骤是相反的）：

```c
{// GCC4.4之前 nand 即 not and
    *ptr = ~*ptr & value;// 先取反，再求与 
    return *ptr;
}
```

# 引用

[**GCC原子操作说明文档**](https://gcc.gnu.org/onlinedocs/gcc-9.1.0/gcc/_005f_005fatomic-Builtins.html#g_t_005f_005fatomic-Builtins)