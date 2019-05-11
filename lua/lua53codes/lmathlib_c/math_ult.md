**math_ult**是lua库函数**math.ult**的具体实现。**math_ult**的目的是将传入的两个参数转化成无符号整数之后，再进行大小的比较。请你注意：虽然**math_ult**不要求你传入的两个参数就是**lua_Integer**类型，但是只接受**lua_Integer**、**lua_Number**和字符串，并且需要你保证传入的参数能正确转换成**lua_Integer**类型。

**math_ult**的声明如下：

```c
static int math_ult (lua_State *L);
```

---

**math_ult**的实现：

```c
static int math_ult (lua_State *L) {
    lua_Integer a = luaL_checkinteger(L, 1);
    lua_Integer b = luaL_checkinteger(L, 2);
    lua_pushboolean(L, (lua_Unsigned)a < (lua_Unsigned)b);
    return 1;
}
```

由上述代码可知，**math_ult**先尝试将传入的2个参数转换成整数a、b；转换成功后，将a、b强制转换成**lua_Unsigned**类型后，进行比较：

+ (lua_Unsigned)a 小于 (lua_Unsigned)b，将真入栈；
+ 否则，将假入栈。

---

最后，为什么命名为**math_ult**？**math_ult**可理解为**“math unsigned less than”**。
