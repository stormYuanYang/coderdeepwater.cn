# math_rad简介

**math_rad**是lua库函数**math.rad**的具体实现。**math_rad**是[**math_deg**](https://coderdeepwater.cn/lua/lua53codes/lmathlib_c/math_deg/)的逆运算：**math_rad**的目的是将以角度为单位的参数转换成以弧度为单位的值。**math_rad**的声明如下：

```c
static int math_rad (lua_State *L);
```

# math_rad实现

**math_rad**的源码：

```c
static int math_rad (lua_State *L) {                                                  
    lua_pushnumber(L, luaL_checknumber(L, 1) * (PI / l_mathop(180.0)));
    return 1;
}
```

**math_rad**的实现和[**math_deg**](https://coderdeepwater.cn/lua/lua53codes/lmathlib_c/math_deg/)的实现同样简单。**math_rad**依赖的数学公式：
$$
角度 = 弧度 \times \frac{\pi}{180^{\circ}}
$$
**math_rad**通过上述的数学公式将单位为弧度的参数转换为角度值。

# 一些说明

**math_rad**和**math_deg**互为逆运算：**math_rad**将弧度变为角度，**math_deg**却将角度变为弧度。

math_**rad**中的**rad**是**radian**的缩写。