# math_randomseed简介

**math_randomseed**是lua库函数**math.randomseed**的具体实现。**math_randomseed**的目的是设定随机数生成器的“种子”，相同的“种子”会产生相同的伪随机数序列。**math_randomseed**的声明如下：

```c
static int math_randomseed (lua_State *L);
```

# math_randomseed实现

**math_randomseed**的实现源码：

```c
static int math_randomseed (lua_State *L) {                                           
    l_srand((unsigned int)(lua_Integer)luaL_checknumber(L, 1));
    (void)l_rand(); /* discard first value to avoid undesirable correlations */
    return 0;
}
```

**math_randomseed**通过调用**l_srand**将传入的参数设为随机数生成器的“种子”。注意接下来的`(void)l_rand();`，执行一次随机数函数的调用的目的是丢弃当前“种子”产生的第一个随机数，避免产生不好的随机数关联，以便提高伪随机性。

# 一些说明

**l_srand**和**l_rand**均是宏定义(都定义在**lmathlib.c**中)：

```c
#if !defined(l_rand)        /* { */                                                   
	#if defined(LUA_USE_POSIX)
		#define l_rand()    random()
		#define l_srand(x)  srandom(x)
		#define L_RANDMAX   2147483647  /* (2^31 - 1), following POSIX */
	#else
		#define l_rand()    rand()
		#define l_srand(x)  srand(x)
		#define L_RANDMAX   RAND_MAX
	#endif
#endif                      /* } */
```

