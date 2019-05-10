**math_sqrt**是lua库函数**math.sqrt**的具体实现。**math_sqrt**的声明如下：

```c
static int math_sqrt (lua_State *L);
```

注意调用**math.sqrt**需要你保证传入的参数是非负数（传入负数，一般会得到**-nan**，即not a number）。

---

**math_sqrt**的实现：

```c
static int math_sqrt (lua_State *L) {
    lua_pushnumber(L, l_mathop(sqrt)(luaL_checknumber(L, 1)));
    return 1;
}
```

**math_sqrt**本身并没有什么特殊的处理，直接调用C标准库函数**sqrt**求参数的平方根，并将求得的结果入栈。返回1表示，**math_sqrt**返回一个值。