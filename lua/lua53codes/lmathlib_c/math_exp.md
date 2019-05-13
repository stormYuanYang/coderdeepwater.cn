# math_exp简介

**math_exp**是lua标准库函数**math.exp**的具体实现。**math_exp**声明如下：

```c
static int math_exp (lua_State *L)；
```

**math_exp**接受一个参数，将其命名为n；**math_exp**的目的是求科学常数**e**（**e**=2.718281828459...）的n次幂，用数学公式可表示为：
$$
e^{n}
$$

---

# math_exp实现

**math_exp**的源代码：

```c
static int math_exp (lua_State *L) {                                                  
    lua_pushnumber(L, l_mathop(exp)(luaL_checknumber(L, 1)));
    return 1;
}
```

**math_exp**的实现非常地简单：先将传入参数转换成**lua_Number**类型的实数n，然后将实数n传递给C标准库函数**exp**；由**exp**求得**e**的n次幂；将**exp**求得的结果入栈。