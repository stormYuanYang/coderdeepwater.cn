C函数**math_abs是**lua函数**math.abs**的具体实现。math_abs本身是对C函数**fabs**的一次封装。math_abs的声明如下：

```c
static int math_abs (lua_State *L);
```

math_abs只受一个lua虚拟机作为参数，实际被操作的参数存储在虚拟机的栈上。math_abs本身是比较简单的，直接看其源代码：

```c
static int math_abs (lua_State *L) {                                             
    if (lua_isinteger(L, 1)) {
        lua_Integer n = lua_tointeger(L, 1);
        if (n < 0) n = (lua_Integer)(0u - (lua_Unsigned)n);
        lua_pushinteger(L, n);
    }
    else
        lua_pushnumber(L, l_mathop(fabs)(luaL_checknumber(L, 1)));
    return 1;
}
```

math_abs先对栈上的参数进行检查；如果参数是整数，则不调用fabs，直接对参数做一次简单的绝对值转换，并将绝对值压栈即可；如果参数是浮点数，则调用fabs求参数的绝对值，并将绝对值压栈。

注意，在lua中调用math.abs求math.mininteger的绝对值：

```lua
print(math.abs(math.mininteger))-- 等价于
print(math.mininteger)
```

math.mininteger的绝对值仍然是math.mininteger，在我的PC上是：
$$
-2^{63} = -9223372036854775808
$$
使用math.abs要小心，对math.mininteger求绝对值不能获得正确结果。其原因已在[**abs**](https://coderdeepwater.cn/c_cpp/stdlibc/stdlib_h/abs/)详细说明，不再赘述。