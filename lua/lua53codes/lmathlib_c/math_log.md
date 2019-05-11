# math_log简介

**math_log**是lua库函数**math.log**的具体实现；一般而言，**math_log**接受两个参数，将这两个参数命名为x和base，math_log的目的是求x以base为底的对数。**math_log**的声明如下：

```c
static int math_log (lua_State *L);
```

使用lua方法**math.log**，需要调用者自己保证参数的合法性（一般规律：在lua中，一个方法需要的是数值，一般可以传入可以转换成数值的字符串，lua会在内部尝试转换）。

---

# math_log实现

**math_log**的源码：

```c
static int math_log (lua_State *L) {
    lua_Number x = luaL_checknumber(L, 1);
    lua_Number res;
    if (lua_isnoneornil(L, 2))
        res = l_mathop(log)(x);
    else {
        lua_Number base = luaL_checknumber(L, 2);
#if !defined(LUA_USE_C89)
        if (base == l_mathop(2.0))
            res = l_mathop(log2)(x); 
        else
#endif
                if (base == l_mathop(10.0))
                    res = l_mathop(log10)(x);
                else
                    res = l_mathop(log)(x)/l_mathop(log)(base);
    }
    lua_pushnumber(L, res);
    return 1;
}
```

**math_log**先获取第一个参数**x**，然后判断第二个参数**base**是否存在、是否为nil。

如果第二个参数**base**不存在或者是nil，则直接调用C标准库函数**log**（此函数的目的和C标准库函数**exp**的目的是相反的，**exp**求以**e**为底的指数）求**x**以科学常数**e**为底的对数；然后将结果入栈，然后函数返回。

如果第二个参数存在，则执行代码：

```c
	else {
        lua_Number base = luaL_checknumber(L, 2);
#if !defined(LUA_USE_C89)
        if (base == l_mathop(2.0))
            res = l_mathop(log2)(x); 
        else
#endif
                if (base == l_mathop(10.0))
                    res = l_mathop(log10)(x);
                else
                    res = l_mathop(log)(x)/l_mathop(log)(base);
    }
```

首先获取第二个参数的值并赋值给**base**；注意这里有一个宏判断`#if !defined(LUA_USE_C89)`，其目的是保持兼容：采用**C89**标准编译lua时，无法使用C标准库函数**log2**的，因为**log2**直到在1993年才被加入标准库。如果不是采用**C89**标准编译lua，则可以使用**log2**，这样直接求以2为底的对数，效率要高一些。当base不等与2.0时，执行代码：

```c
				if (base == l_mathop(10.0))
                    res = l_mathop(log10)(x);
                else
                    res = l_mathop(log)(x)/l_mathop(log)(base);
```

这里首先判断一下base是不是等于10；如果base等于10，则直接调用C标准库函数**log10**求x以10为底的对数；否则，根据数学公式（其中a、e均是常数）：
$$
log_{a}x = \frac{log_{e}x}{log_{e}a}
$$
可以通过先求得**x**以**e**为底的对数，再求得**base**以**e**为底的对数，前者除以后者就得到**x**以**base**为底的对数。

---

# 一些说明

1. 我们称以**10**为底的对数叫做**常用对数**(common logarithm)；
2. 称以无理数**e**（e=2.71828...）为底的对数称为**自然对数**(natural logarithm)；
3. **e**是一个无理数(**e**=2.71828...)，我们常常称其为**科学常数**；
4. **0**是没有对数的；
5. 在实数范围内，负数没有对数；在虚数范围内，负数有对数。

