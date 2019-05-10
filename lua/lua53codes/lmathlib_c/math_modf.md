**math_modf**是lua库函数**math.modf**的具体实现，和math_ceil、math_floor不同，math_modf不使用任何的C标准库函数，独立地实现了和C标准库函数modf相同的功能。**math_modf**的声明如下：

```c
static int math_modf (lua_State *L);
```

**math_modf**的目的是获得一个浮点数的整数部分（向0取整）和小数部分。

---

**math_modf**的实现：

```c
/*
 ** next function does not use 'modf', avoiding problems with 'double*'
 ** (which is not compatible with 'float*') when lua_Number is not
 ** 'double'.
 */
static int math_modf (lua_State *L) {
    if (lua_isinteger(L ,1)) {
        lua_settop(L, 1);  /* number is its own integer part */
        lua_pushnumber(L, 0);  /* no fractional part */
    }
    else {
        lua_Number n = luaL_checknumber(L, 1);
        /* integer part (rounds toward zero) */
        lua_Number ip = (n < 0) ? l_mathop(ceil)(n) : l_mathop(floor)(n);
        pushnumint(L, ip);
        /* fractional part (test needed for inf/-inf) */
        lua_pushnumber(L, (n == ip) ? l_mathop(0.0) : (n - ip));
    }
    return 2;
}
```

为什么lua不直接使用C标准库的modf函数获取浮点数的整数副本和小数部分呢？先看modf的声明：

```c
double modf (double x, double* intpart);
```

当LUA_NUMBER不是double类型时(LUA_NUMBER可以宏定义为float类型)，double* 和float*并不兼容，这样可能会导致一些问题。

看代码：

```c
if (lua_isinteger(L ,1)) {
    lua_settop(L, 1);  /* number is its own integer part */
    lua_pushnumber(L, 0);  /* no fractional part */
}
```

math_modf先判断参数本身是不是整数；如果是的话，直接将其设置到栈顶作为整数部分，再压入0作为小数部分；否则继续执行：

```c
else {
    lua_Number n = luaL_checknumber(L, 1);
    /* integer part (rounds toward zero) */
    lua_Number ip = (n < 0) ? l_mathop(ceil)(n) : l_mathop(floor)(n);
    pushnumint(L, ip);
    /* fractional part (test needed for inf/-inf) */
    lua_pushnumber(L, (n == ip) ? l_mathop(0.0) : (n - ip));
}
```

math_modf取整数部分，采用向0舍入的策略；然后用浮点数减去整数部分得到小数部分。例如：

+ math.modf(-2.9) --> 整数部分：-2 小数部分：-0.9
+ math.modf(2.9) --> 整数部分：2 小数部分：0.9