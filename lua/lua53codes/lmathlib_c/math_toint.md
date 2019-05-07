**math_toint**是lua函数**math.tointeger**的具体实现，其声明如下：

```c
static int math_toint (lua_State *L);
```

math_toint接受一个参数，将其转换成整数；转换成功，就将转换后的整数压栈；否则，就将nil压入栈中。

math_toint的实现：

```c
static int math_toint (lua_State *L) {
    int valid;
    lua_Integer n = lua_tointegerx(L, 1, &valid);
    if (valid)
        lua_pushinteger(L, n);
    else {
        luaL_checkany(L, 1);
        lua_pushnil(L);  /* value is not convertible to integer */
    }
    return 1;
}
```

math_toint首先调用lua_tointegerx（接受整数、浮点数、字符串）尝试将参数转换成整数：

```c
lua_Integer n = lua_tointegerx(L, 1, &valid);
```

转换成功，**valid**为真；否则valid为假。

math_toint随后对valid进行真假判断：

+ 如果valid为真，则直接将n压入栈中；
+ 否则，首先检查调用者是否传递了参数；如果调用者传递了参数，将nil压入栈中；否则`luaL_checkany(L, 1);`就会报错。

最后返回1，表明math_toint的返回值只有1个。

注意，调用lua函数math.tointeger时（实际调用math_toint），必须由调用者保证传递的参数能正确地表示为整数：

+ 太大或者太小的整数、浮点数都无法正确转换成整数；
+ 太长的字符串形式表示的整数或者浮点数无法正确转换成整数。