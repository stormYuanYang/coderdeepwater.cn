# 简介

lua库函数**table.unpack**：

```lua
table.unpack(list, [i [, j]])
```

**table.unpack**的目的是返回list中范围在$[i,j]$内的元素，这个函数等价于`return list[i],list[i+1],...,list[j]`。

**table.unpack**支持为**负数**的i和j，但是必须满足$j-i+1<=INT\_MAX$并且栈空间足以容纳$j-i+1$个元素。使用**table.unpack**时，可以不传递参数i和j：

+ i的默认值为1；
+ j的默认值为#list；

下文对**table.unpack**的具体实现C函数**unpack**进行分析。

# 分析

注解**unpack**：

```c
static int unpack (lua_State *L) {
    lua_Unsigned n;
    // i默认为1
    lua_Integer i = luaL_optinteger(L, 2, 1);
    // e默认就是序列的长度
    lua_Integer e = luaL_opt(L, luaL_checkinteger, 3, luaL_len(L, 1));
    // 范围不合法，没有range
    if (i > e) return 0;  /* empty range */
    // 如果i==INT_MIN，e==INT_MAX；此时e-i+1就会越界
    // 所以让n=e-i
    n = (lua_Unsigned)e - i;  /* number of elements minus 1 (avoid overflows) */
    // 对n的值做一次检测,n必须要小于INT_MAX且，栈空间能容纳n+1个元素
    if (n >= (unsigned int)INT_MAX  || !lua_checkstack(L, (int)(++n))) {
        // 实际上这里永远不会返回,luaL_error()会处理错误
        return luaL_error(L, "too many results to unpack");
    }
    // 这里需要注意，终止条件不能是i<=e；不然就会在e==INT_MAX时，
    // 导致i在等于e之后的下次自增时越界
    // 这样的后果就是死循环
    for (; i < e; i++) {  /* push arg[i..e - 1] (to avoid overflows) */
        // 从表中取出元素，并入栈
        lua_geti(L, 1, i);
    }
    // 单独处理最后一个元素
    lua_geti(L, 1, e);  /* push last element */
    // 返回n表示，unpack的返回值一共有n个
    return (int)n;
}
```

**unpack**的实现并不复杂，不过要注意**unpack**是如何处理边界情况的，这一点值得我们学习。最后分析一下**unpack**的时间复杂度和空间复杂度。

###### 时间复杂度

**unpack**除for外均是常量时间操作；for循环内的lua_geti()的时间复杂度是$O(1)$；故，**unpack**的时间复杂度是：
$$
O(n)
$$

###### 空间复杂度

**unpack**需要$j-i+1$的栈空间，所以**unpack**的空间复杂度是：
$$
O(n)
$$

# 引用

[**lua5.3参考手册之table.unpack**](https://www.lua.org/manual/5.3/manual.html#pdf-table.unpack)