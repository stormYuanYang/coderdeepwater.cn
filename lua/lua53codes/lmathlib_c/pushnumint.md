**pushnumint**是lmathlib.c中的一个辅助函数，其作用是向栈中压入一个整数或者整数：

+ 如果传入的浮点数参数能转换成整数就向栈中压入转换后的整数；
+ 否则将浮点数直接压入栈中。

pushnumint的声明如下：

```c
static void pushnumint (lua_State* L, lua_Number d);
```

---

现在来看看pushnumint的实现：

```c
static void pushnumint (lua_State *L, lua_Number d) {
    lua_Integer n;
    if (lua_numbertointeger(d, &n))  /* does 'd' fit in an integer? */
        lua_pushinteger(L, n);  /* result is integer */
    else
        lua_pushnumber(L, d);  /* result is float */
}
```

pushnumint的结构一目了然。调用**lua_numbertointeger**尝试将浮点数d转换成整数；如果d能转换成整数，则将其整数值n压入栈中；否则，直接将d压入栈中。

那这里的**lua_numbertointeger**又是如何判断d能否转换成一个整数的呢？

**lua_numbertointeger**并不是一个函数，而是一个宏定义（定义在luaconf.h中），其定义为：

```c
/*
@@ lua_numbertointeger converts a float number to an integer, or
** returns 0 if float is not within the range of a lua_Integer.
** (The range comparisons are tricky because of rounding. The tests
** here assume a two-complement representation, where MININTEGER always
** has an exact representation as a float; MAXINTEGER may not have one,
** and therefore its conversion to float may have an ill-defined value.)
*/
#define lua_numbertointeger(n,p) \
  ((n) >= (LUA_NUMBER)(LUA_MININTEGER) && \
   (n) < -(LUA_NUMBER)(LUA_MININTEGER) && \
      (*(p) = (LUA_INTEGER)(n), 1))
```

**lua_numbertointeger**对传入的浮点数n做两次条件判断，n必须要落在能被**LUA_INTEGER**表示的范围内，即n满足：
$$
LUA\_MININTEGER <= n <= LUA\_MAXINTEGER
$$
当n满足条件时，直接将n强制转换成**LUA_INTEGER**类型并复制给\*p，并返回真；否则返回假。

注意，这里并不是直接判断n是否小于等于**(LUA_NUMBER)LUA_MAXINTEGER**，而是选择判断n是否小于**-(LUA_NUMBER)LUA_MININTEGER**；这样做的目的，注释中已经给出了解释，但是并未说明为什么**LUA_MAXINTEGER**可能无法用**LUA_NUMBER**类型表示：

LUA_NUMBER实际上是double(也可能是float、long double)的别名。LUA_INTEGER实际上是long int(也可能是int、long long int)的别名。而double在内存中存储结构导致，double表示的浮点数是拥有精度限制的，可能无法向long int一样准确地表示一个整数。

double可以用公式表示为：
$$
精度 a\times 2^{指数e}
$$
对于double来说，大于等于$2^{53}$的无符号整数就**可能**无法被精确表示了；因为double用53个bit表示精度，在指数确定时，最多只能表示$2^{53}$个不同的数。特别地，当指数e为0时：
$$
[0, 2^{53}-1]
$$
对于有符号整数来说这个范围就是:
$$
[-2^{52},2^{52}-1]
$$
当超过double能精确表示的整数范围后，就只有2的整数幂一定能被double精确表示，其他的数字则**可能**无法精确被表示；所lua在这里用**-(LUA_NUMBER)LUA_MININTEGER**(即$2^{63}​$)来做判断。

举个例子：

```c
#include <stdio.h>
#include <limits.h>

int main() {
    printf("%u %u\n", sizeof(long long int), sizeof(double));
    printf("%ld\n", LLONG_MAX);
    double a = (double)LLONG_MAX;
    printf("%lf\n", a);
	printf("将a转换成整数:%ld\n", (long long int)a);
    printf("%ld\n", LLONG_MIN);
    a = (double)LLONG_MIN;
    printf("%lf\n", a);
    return 0;
}
```

在我的PC上，打印结果：

```c
8 8
9223372036854775807
9223372036854775808.000000
将a转换成整数:-9223372036854775808
-9223372036854775808
-9223372036854775808.000000
```

**LLONG_MAX**被强制转换成double类型后，其结果并不等于**LLONG_MAX**了，已经有大小为1的误差了；再将a强转回**long long int**，其结果已经变成**LLONG_MIN**，这就是真正的天差地别。**LLONG_MIN**被强制转换成double类型后，其结果仍然正确地等于**LLONG_MIN**。