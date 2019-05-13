# math_deg简介

**math_deg**是lua库函数**math.deg**的具体实现。**math_deg**的目的是将以弧度为单位的参数转换成以角度为单位的值。**math_deg**的声明如下：

```c
static int math_deg (lua_State *L);
```

---

# math_deg实现

**math_deg**的源码：

```c
static int math_deg (lua_State *L) {                                                  
    lua_pushnumber(L, luaL_checknumber(L, 1) * (l_mathop(180.0) / PI));
    return 1;
}
```

**math_deg**的实现代码是一目可以了然的。**math_deg**能正确求得角度依赖的数学公式：
$$
角度 = 弧度 \times \frac{180^{\circ}}{\pi}
$$
**math_deg**通过上述的数学公式求得角度值，然后将结果入栈。

# 一些说明

1个单位的弧度大约等于$57.3^{\circ}$.

一个单位的角度等于$\frac{180^{\circ}}{\pi}$弧度.

math_**deg**中的**deg**是**degree**（角度）的缩写。