头文件**atomic.h**定义了系列的原子操作宏定义以保证线程安全，这些原子操作宏定义是对GCC提供的内置原子操作api的一次封装。关于GCC的内置原子操作api已在[《GCC内建的原子操作api》](https://coderdeepwater.cn/c_cpp/gcc/%E5%86%85%E5%BB%BA%E7%9A%84%E5%8E%9F%E5%AD%90%E6%93%8D%E4%BD%9Capi/)中详细介绍，不再赘述。

atomic.h的实现：

```c
#ifndef SKYNET_ATOMIC_H
#define SKYNET_ATOMIC_H

#define ATOM_CAS(ptr, oval, nval) __sync_bool_compare_and_swap(ptr, oval, nval)
#define ATOM_CAS_POINTER(ptr, oval, nval) __sync_bool_compare_and_swap(ptr, oval, nval)
#define ATOM_INC(ptr) __sync_add_and_fetch(ptr, 1)
#define ATOM_FINC(ptr) __sync_fetch_and_add(ptr, 1)
#define ATOM_DEC(ptr) __sync_sub_and_fetch(ptr, 1)
#define ATOM_FDEC(ptr) __sync_fetch_and_sub(ptr, 1)
#define ATOM_ADD(ptr,n) __sync_add_and_fetch(ptr, n)
#define ATOM_SUB(ptr,n) __sync_sub_and_fetch(ptr, n)
#define ATOM_AND(ptr,n) __sync_and_and_fetch(ptr, n)

#endif
```

现在对atomic.h定义的原子操作宏定义一一说明。

CAS操作：

```c
#define ATOM_CAS(ptr, oval, nval) __sync_bool_compare_and_swap(ptr, oval, nval)
#define ATOM_CAS_POINTER(ptr, oval, nval) __sync_bool_compare_and_swap(ptr, oval, nval)
```

**ATOM_CAS**和**ATOM_CAS_POINTER**对应的文本是一致的，通过调用**__sync_bool_compare_and_swap**执行CAS操作。

自增操作：

```c
#define ATOM_INC(ptr) __sync_add_and_fetch(ptr, 1)
#define ATOM_FINC(ptr) __sync_fetch_and_add(ptr, 1)
```

注意**ATOM_INC**和**ATOM_FINC**的不同之处：**ATOM_INC**类似++n，**ATOM_FINC**类似n++。

自减操作：

```c
#define ATOM_DEC(ptr) __sync_sub_and_fetch(ptr, 1)
#define ATOM_FDEC(ptr) __sync_fetch_and_sub(ptr, 1)
```

注意**ATOM_DEC**和**ATOM_FDEC**的不同之处：**ATOM_DEC**类似--n，**ATOM_FDEC**类似n--。

加法操作：

```c
#define ATOM_ADD(ptr,n) __sync_add_and_fetch(ptr, n)
```

注意，这里的加法类似`{*ptr += n; return *ptr;}`；可以仿造**ATOM_FINC**定义一个新的原子操作宏定义**ATOM_FADD**：

```c
// 构造的例子（skynet源码中并未有此宏定义）
#define ATOM_FADD(ptr,n) __sync_fetch_and_add(ptr, n)
```

ATOM_FADD类似`{tmp = *ptr; *ptr += n; return *tmp;}`。

减法操作：

```c
#define ATOM_SUB(ptr,n) __sync_sub_and_fetch(ptr, n)
```

**ATOM_SUB**类似`{*ptr -= n; return *ptr;}`。

与操作：

```c
#define ATOM_AND(ptr,n) __sync_and_and_fetch(ptr, n)
```

**ATOM_AND**类似`{*ptr &= n; return *ptr;}`。

---

atomic.h并未使用到所有的内置原子操作函数，想对内置原子操作函数有更多了解，可以阅读[《GCC内建的原子操作api》](https://coderdeepwater.cn/c_cpp/gcc/%E5%86%85%E5%BB%BA%E7%9A%84%E5%8E%9F%E5%AD%90%E6%93%8D%E4%BD%9Capi/)。