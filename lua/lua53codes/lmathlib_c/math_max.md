# math_max简介

**math_max**是lua库函数**math.max**的具体实现。**math_max**的目的是获得参数中最大的那个参数。**math_max**的声明如下：

```c
static int math_max (lua_State *L);
```

# math_max实现

**math_max**的实现源码：

```c
static int math_max (lua_State *L) {
    int n = lua_gettop(L);  /* number of arguments */
    int imax = 1;  /* index of current maximum value */
    int i;
    luaL_argcheck(L, n >= 1, 1, "value expected");
    for (i = 2; i <= n; i++) {
        if (lua_compare(L, imax, i, LUA_OPLT))
            imax = i;
    }
    lua_pushvalue(L, imax);
    return 1;
}
```

**math_max**和[**math_min**](https://coderdeepwater.cn/lua/lua53codes/lmathlib_c/math_min/)的实现非常类似，只不过前者求最大值，后者求最小值。**math_max**通过遍历所有参数查找最大值，和**math_min**一样，需要注意参数之间的可比较性。**math_max**和**math_min**的时间复杂度完全一样，都是线性的：
$$
O(n)
$$
