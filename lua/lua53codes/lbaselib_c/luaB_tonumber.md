# 简介

C函数**luaB_tonumber**是lua库函数**tonumber**的具体实现。**luaB_tonumber**的目的是将传入的参数转换为一个数字。**luaB_tonumber**的声明如下：

```c
static int luaB_tonumber (lua_State *L);
```

**luaB_tonumber**接受两个参数e和base，由**L**的栈携带。

# 分析

先看**luaB_tonumber**源码：

```c
static int luaB_tonumber (lua_State *L) {
    if (lua_isnoneornil(L, 2)) {  /* standard conversion? */
        luaL_checkany(L, 1);
        if (lua_type(L, 1) == LUA_TNUMBER) {  /* already a number? */
            lua_settop(L, 1);  /* yes; return it */
            return 1;
        }
        else {
            size_t l;
            const char *s = lua_tolstring(L, 1, &l);
            if (s != NULL && lua_stringtonumber(L, s) == l + 1)
                return 1;  /* successful conversion to number */
            /* else not a number */
        }
    }
    else {
        size_t l;
        const char *s;
        lua_Integer n = 0;  /* to avoid warnings */
        lua_Integer base = luaL_checkinteger(L, 2);
        luaL_checktype(L, 1, LUA_TSTRING);  /* no numbers as strings */
        s = lua_tolstring(L, 1, &l);
        luaL_argcheck(L, 2 <= base && base <= 36, 2, "base out of range");
        if (b_str2int(s, (int)base, &n) == s + l) {
            lua_pushinteger(L, n);
            return 1;
        }  /* else not a number */
    }  /* else not a number */
    lua_pushnil(L);  /* not a number */
    return 1;
}
```

**luaB_tonumber**首先执行代码`if (lua_isnoneornil(L, 2))`判断是否传入了转换的基数**base**，如果不传入参数**base**，则是标准转换，即将参数转换成十进制：

+ 执行`luaL_checkany(L, 1);`检查第一个参数**e**是否存在；
+ 然后执行`if (lua_type(L, 1) == LUA_TNUMBER)`判断参数**e**是否已经是数字，如果是就直接将其入栈，函数执行完毕，否则继续执行；
+ 执行`const char *s = lua_tolstring(L, 1, &l);`尝试将参数**e**转换成字符串；然后执行`if (s != NULL && lua_stringtonumber(L, s) == l + 1)`判断是否转换成功，如果成功，函数执行完毕。

不是标准转换：

+ 执行`lua_Integer base = luaL_checkinteger(L, 2);`获取**base**;
+ 然后执行`luaL_checktype(L, 1, LUA_TSTRING);`进行检查：如果参数**e**不是数字，则只能是字符串；
+ 注意`luaL_argcheck(L, 2 <= base && base <= 36, 2, "base out of range");`，这里对传入的**base**做了一次检查，转换的基数只能是$[2,36]$内的整数；
+ 然后执行`if (b_str2int(s, (int)base, &n) == s + l)`尝试将字符串**s**转换成以**base**为基数的数字；其中**b_str2int**就是**lbaselib.c**中的一个内部方法；如果转换成功就将结果入站。

最后，如果无法转换，**luaB_tonumber**调用`lua_pushnil(L);`将**nil**入站，表示无法转换成数字。