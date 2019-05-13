# math_random简介

**math_random**是lua库函数**math.random**的具体实现。**math_random**的目的是获取一个伪随机数。**math_random**允许你通过传入的参数来控制产生的随机数的范围。**math_random**的声明如下：

```c
static int math_random (lua_State *L);
```

# math_random实现

**math_random**的实现源码：

```c
static int math_random (lua_State *L) {
    lua_Integer low, up;
    double r = (double)l_rand() * (1.0 / ((double)L_RANDMAX + 1.0));
    switch (lua_gettop(L)) {  /* check number of arguments */
        case 0: {  /* no arguments */
                    lua_pushnumber(L, (lua_Number)r);  /* Number between 0 and 1 */
                    return 1;
                }
        case 1: {  /* only upper limit */
                    low = 1;
                    up = luaL_checkinteger(L, 1);
                    break;
                }
        case 2: {  /* lower and upper limits */
                    low = luaL_checkinteger(L, 1);
                    up = luaL_checkinteger(L, 2);
                    break;
                }
        default: return luaL_error(L, "wrong number of arguments");
    }
    /* random integer in the interval [low, up] */
    luaL_argcheck(L, low <= up, 1, "interval is empty");
    luaL_argcheck(L, low >= 0 || up <= LUA_MAXINTEGER + low, 1,
            "interval too large");
    r *= (double)(up - low) + 1.0;
    lua_pushinteger(L, (lua_Integer)r + low);
    return 1;
}
```

现在分析**math_random**的实现：

```c
double r = (double)l_rand() * (1.0 / ((double)L_RANDMAX + 1.0));
```

**math_random**直接通过**l_rand**先求得一个随机数，然后将其缩小到$[0,1)$之间。注意，这里使用**double**类型来表示随机数，而不是**lua_Number：**考虑到兼容性，**lua_Number**可能是**float**类型；当**lua_Number**是**float**类型时，缩小随机数可能会丢失精度；缩小随机数时应该竟可能地保留精度，这样才能体现更好的随机性。这里(double)L_RANDMAX加1的目的是，保证缩小后的随机数**r**小于1。

接下来是一个**switch**语句，对调用者传入的参数做判断，并根据参数的数量和值确定求取随机数范围的上界**up**和下界**low**：

+ 不传入任何参数调用lua函数**math.random**；此为默认情况，随机数范围在$[0,1)$之间；直接将随机数入栈即可；
+ 只传入一个参数调用lua函数**math.random**；此时，**math_random**认为传入的是上界，随机数范围在$[1,up]$内；
+ 传入两个参数调用lua函数**math.random**；此时，下界和上界都由调用者提供，随机数范围在$[low,up]$内。

接下来执行参数检查：

```c
luaL_argcheck(L, low <= up, 1, "interval is empty");
luaL_argcheck(L, low >= 0 || up <= LUA_MAXINTEGER + low, 1,
            "interval too large");
```

需要调用者保证：

+ **low**小于等于**up；**
+ 随机数范围要在合理范围内：**随机数范围**的大小（也可称之为长度）只能在**[0,LUA_MAXINTEGER]**内;随机数本身只能在**[low,LUA_MAXINTEGER+low]**之间。

然后执行：

```c
r *= (double)(up - low) + 1.0;
```

这里加1.0的目的是保证**r**在$[0,up-(low-1)]$内;

然后将**r**移动到$[low,high]$中：

```c
lua_pushinteger(L, (lua_Integer)r + low);
```

将**r**移动到$[low,high]$范围内，得到最终结果，然后将其入栈。

