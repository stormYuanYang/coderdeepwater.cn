话说都半年没写博客了呢😁。哈哈哈哈哈，没办法，项目上线以来，自己太过匆忙（其实是自己变懒了）。

昨天晚上呢，建东问我lua提供的table最多能放下多少个元素。这个问题呢，其实之前通过看lua的源代码就已知晓，不过长时间不温习，已经忘却😢。正好乘着为建东解惑的机会，做一个笔记。

table由数组部分和哈希表部分组成，table的容量也就等于数组容量与哈希表容量之和。table的一切秘密都可以到ltable.c中挖掘，table的极限容量也是如此:

```c
/*
 ** Maximum size of array part (MAXASIZE) is 2^MAXABITS. MAXABITS is
 ** the largest integer such that MAXASIZE fits in an unsigned int.
 */
#define MAXABITS	cast_int(sizeof(int) * CHAR_BIT - 1)
#define MAXASIZE	(1u << MAXABITS)
```

由上述宏定义可知，table的数组部分最大容量是 **2的MAXABITS次方**，这个值的大小取决于int的字节数：int是32位，table的数组部分最大容量就是**2的31次方**；int是64位，table的数组部分最大容量就是**2的63次方**。

```c
/*
 ** Maximum size of hash part is 2^MAXHBITS. MAXHBITS is the largest
 ** integer such that 2^MAXHBITS fits in a signed int. (Note that the
 ** maximum number of elements in a table, 2^MAXABITS + 2^MAXHBITS, still
 ** fits comfortably in an unsigned int.)
 */
#define MAXHBITS	(MAXABITS - 1)

```

由上述宏定义和注释可知，table的哈希表部分最大容量只有数组部分最大容量的一半。注意，这里说的哈希表容量并不代表哈希表可以存储元素的数量，而是指实现哈希表的数组的最大容量，哈希表的实现方法的不同，哈希表存储元素的数量上限也就不同（链接法、开放寻址法）。

table的数组最大容量与哈希表最大容量之和能够用unsigned int表示。可以这样简单地记忆table的最大容量：最大unsigned int的一半就是数组部分的最大容量，数组部分最大容量的一半就是哈希表部分的最大容量。