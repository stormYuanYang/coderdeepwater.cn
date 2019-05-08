**math_fmod**是lua库函数**math.fmod**的具体实现，**math_fmod**的声明如下：

```c
static int math_fmod (lua_State *L);
```

**math_fmod**接受两个参数，命名这两个参数为x、y，**math_fmod**求x除以y的余数。

---

**math_fmod**的实现：

```c
static int math_fmod (lua_State *L) {
    if (lua_isinteger(L, 1) && lua_isinteger(L, 2)) {
        lua_Integer d = lua_tointeger(L, 2);
        if ((lua_Unsigned)d + 1u <= 1u) {  /* special cases: -1 or 0 */
            luaL_argcheck(L, d != 0, 2, "zero");
            lua_pushinteger(L, 0);  /* avoid overflow with 0x80000... / -1 */
        }
        else
            lua_pushinteger(L, lua_tointeger(L, 1) % d);
    }
    else
        lua_pushnumber(L, l_mathop(fmod)(luaL_checknumber(L, 1),
                    luaL_checknumber(L, 2)));
    return 1;
}
```

**math_fmod**首先判断传入参数的类型。如果两个参数都是整数类型，则单独处理求余；否则调用C语言的库函数fmod对浮点数进行余数求解。

当参数x、y都是整数时，lua对第二个参数即y做了安全判断，避免算术运算的异常。`luaL_argcheck(L, d != 0, 2, "zero");`对y等于0做了检查；当y等于-1时，如果x等于**0x8000...**（LUA_MININTEGER），而**0x8000...**除以**-1**的结果是无法用当前的整数表示的（因为这个值实际等于**LUA_MAXINTEGER+1**）；为避免溢出，`lua_pushinteger(L, 0);`对此情况做了一个简单处理：直接将余数设置为0。

当参数x、y不都是整数时，lua先调用**luaL_checknumber**尝试将其转换成浮点数；转换成功后，调用C标准库方法**fmod**求x除以y的余数。注意，求得的余数结果可能并不是你想要的，因为浮点数是存在误差的。看一个例子：

```c
// test.c
#include <stdio.h>      
#include <math.h>       
int main ()
{
    printf( "%lf\n%lf\n", 0.6 / 0.2, fmod(0.6,0.2));
    return 0;
}
```

这里顺便提醒一下：如果你是用gcc编译上述代码，记得在编译时加上参数**-lm**，表示链接数学库，否则gcc可能会报告**fmod**是未定义的引用。完整的编译并执行程序命令可参考：

```bash
gcc -lm -o test test.c && ./test
```

在我的PC上，**fmod(0.6,0.2)**的打印结果是**0.2**，这个结果明显是错误的，因为0.6除以0.2能够整数，其商为3；那为什么**fmod(0.6,0.2)**的结果是0.2呢？根本原因就是浮点数是有精度限制的，0.6除以0.2理应等于3，但是在计算机中的结果可能并不是这样；对test.c代码进行修改：

```c
//test1.c
#include <stdio.h>      
#include <math.h>       
int main ()
{
    printf( "%0.17lf\n%0.17f\n", 0.6 / 0.2, fmod(0.6,0.2));
    return 0;
}
```

运行此程序，得到打印结果：

**2.99999999999999956**

**0.19999999999999996**

0.6除以0.2实际等于**2.99999999999999956**（近似于**3**），这就导致了0.6除以0.2的余数是**0.19999999999999996**（近似于**0.2**）。所以求两个浮点数的余数时，要特别小心，实际的结果和你预期的结果可能是南辕北辙的。