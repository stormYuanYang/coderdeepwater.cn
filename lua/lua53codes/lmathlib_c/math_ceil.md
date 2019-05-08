**math_ceil**是lua库函数math.ceil的具体实现（**math_ceil**的实现和**math_floor**的实现非常类似），其声明为：

```c
static int math_ceil (lua_State *L);
```

**math_ceil**接受一个参数，这个参数必须是**number**类型（整数或者浮点数）或者字符串。**math_ceil**正确接受参数后，获取不小于参数的最小整数值；注意获取整数值不一定是lua内部的**LUA_INTEGER**类型，可能是**LUA_NUMBER**类型。

------

**math_ceil**的实现：

```c
static int math_ceil (lua_State *L) {
    if (lua_isinteger(L, 1))
        lua_settop(L, 1);  /* integer is its own ceil */
    else {
        lua_Number d = l_mathop(ceil)(luaL_checknumber(L, 1));
        pushnumint(L, d);
    }
    return 1;
}
```

**math_ceil**先判断栈上的参数类型：

- 如果是LUA_INTEGER类型，直接将参数设置到栈顶；
- 否则，调用**luaL_checknumber**尝试将参数转换成一个数；转换成功就将这个数传递给**ceil**，由**ceil**给出不小于参数的最小整数值，并调用**pushnumint**将结果压栈。
