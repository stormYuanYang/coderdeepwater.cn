# math_min简介

**math_min**是lua库函数**math.min**的具体实现。**math_min**的目的是找到传入的参数中，最小的那个参数。**math_min**的声明如下：

```c
static int math_min (lua_State *L);
```

# math_min实现

**math_min**的源码：

```c
static int math_min (lua_State *L) {
    int n = lua_gettop(L);  /* number of arguments */
    int imin = 1;  /* index of current minimum value */
    int i;
    luaL_argcheck(L, n >= 1, 1, "value expected");
    for (i = 2; i <= n; i++) {
        if (lua_compare(L, i, imin, LUA_OPLT))
            imin = i;
    }
    lua_pushvalue(L, imin);
    return 1;
}
```

**math_min**的实现并不复杂：**math_min**遍历所有的参数，找到其中最小的参数。找到最小参数后直接将其入栈。需要注意的是，调用lua函数**math.min**时需要你保证参数之间的可比较性：一个**数字**如何和一个**table**进行比较呢？

现在我们分析一下**math_min**的时间复杂度。显而易见地，因为**math_min**除了**for**循环，其他操作都花费常数时间，所以**math_min**的时间复杂度是：
$$
O(n)
$$
