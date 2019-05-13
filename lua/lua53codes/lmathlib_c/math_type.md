# math_type简介

**math_type**是lua库函数**math.type**的具体实现。**math_type**的作用是判断一个参数是整数还是浮点数：

+ 如果参数是整数，则返回“integer”；
+ 如果参数是浮点数，则返回“float”；
+ 如果参数不是数字，则返回nil。

**math_type**的声明如下：

```c
static int math_type (lua_State *L);
```

# math_type实现

**math_type**的实现源码：

```c
static int math_type (lua_State *L) {
    if (lua_type(L, 1) == LUA_TNUMBER) {
        if (lua_isinteger(L, 1))
            lua_pushliteral(L, "integer");
        else
            lua_pushliteral(L, "float");
    }
    else {
        luaL_checkany(L, 1);
        lua_pushnil(L);
    }
    return 1;
}
```

**math_type**判断参数是否是**LUA_TNUMBER**类型；如果是**LUA_TNUMBER**的话，则再判断参数是整数还是浮点数；否则，如果是传入了参数，则将nil入栈。