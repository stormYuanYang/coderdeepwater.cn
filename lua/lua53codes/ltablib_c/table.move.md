# 简介

lua库函数**table.move**声明如下：

```lua
table.move(a1, f, e, t [,a2])
```

**table.move**的目的是将表a1中$[f,e]$范围内的元素拷贝到表a2的$[t,t+(e-(f-1))]$范围内。**table.move**需要调用者保证：

+ **e**大于等于**f**，否则**table.move**并不会拷贝元素，当然也不会报错；
+ **$e-(f-1)$**即拷贝的元素数量，不能超过**LUA_MAXINTEGER**；
+ $t+(e-(f-1))$不能大于**LUA_MAXINTEGER**,这样a2才可能存放下被拷贝的元素。

当调用**table.move**时，可以不传递参数a2，此时a2默认设置为a1，意味着源表和目标表是同一个表，也就是说在同一个表类进行元素的拷贝。当a1和a2是同一个表时，并且拷贝的范围$[f,e]$和$[t,t+(e-(f-1))]$之间有重叠，此时就可能会造成数据错乱，不过不用担心，**table.move**已经正确处理此情况，就算拷贝范围有重叠也能得到正确结果。下文会说明如何处理拷贝数据重叠造成的数据错乱。

# 分析

注解**table.move**：

```c
static int tmove (lua_State *L) {
    // 获取copy的首位置f
    lua_Integer f = luaL_checkinteger(L, 2);
    // 获取copy的结束位置e
    lua_Integer e = luaL_checkinteger(L, 3);
    // 获取目标起始位置t
    lua_Integer t = luaL_checkinteger(L, 4);
    // 如果没有目标table，则默认就是源表自己
    int tt = !lua_isnoneornil(L, 5) ? 5 : 1;  /* destination table */
    // 确认操作的数据是table或者具有table的功能
    checktab(L, 1, TAB_R);
    checktab(L, tt, TAB_W);
    // 判断拷贝的范围是否合法
    if (e >= f) {  /* otherwise, nothing to move */
        lua_Integer n, i;
        // 对拷贝的元素数量做一次检查，拷贝的元素数量不能超过LUA_MAXINTEGER
        luaL_argcheck(L, f > 0 || e < LUA_MAXINTEGER + f, 3,
                "too many elements to move");
        n = e - f + 1;  /* number of elements to move */
        // 检查目的位置能否容纳被拷贝的元素
        luaL_argcheck(L, t <= LUA_MAXINTEGER - n + 1, 4,
                "destination wrap around");
        // 因为可能就是在同一个table中进行元素的拷贝
        // 可能会涉及元素重叠的问题，所以这里做了一些处理
        if (t > e || t <= f || (tt != 1 && !lua_compare(L, 1, tt, LUA_OPEQ))) {
            // 元素不会重叠时，直接按下标递增的方式将元素拷贝到对应位置
            // 或者元素会重叠，但是按下标递增方式拷贝元素能正确得到结果时
            // 就按下标递增的方式拷贝元素
            for (i = 0; i < n; i++) {
                // 读取元素，入栈 
                lua_geti(L, 1, f + i);
                // 将栈顶元素设置到对应位置
                lua_seti(L, tt, t + i);
            }
        }
        // 元素会重叠，并且按下标递增方式拷贝元素时会造成数据错落
        // 就必须按下标递减的方式将元素拷贝到对应位置
        // 这样才不会造成数据错乱
        else {
            for (i = n - 1; i >= 0; i--) {
                lua_geti(L, 1, f + i);
                lua_seti(L, tt, t + i);
            }
        }
    }
    // 将目标table压入栈中，作为函数的返回值
    lua_pushvalue(L, tt);  /* return destination table */
    return 1;
}
```

接下来分析**tmove**是如何处理拷贝元素重叠问题的：

```lua
--此伪代码是lua代码
t = [1,2,3,4,5,6]-- 即是源也是目标
按下标递增进行拷贝，此时源范围[3,5]和目标范围[4,6]重叠
[.,.,3,4,5,.]-- 源范围[3,5] = {3,4,5}
     f   e
[.,.,.,4,5,6]-- 目标范围[4,6]
       t
期望得到
[1,2,3,3,4,5]
但实际会得到
[1,2,3,3,3,3]
```

这是为什么呢？在拷贝t[3]到t[4]时，t[4]变成了3；原来的t[4]等于4，但是t[4]已经被3覆盖了；也就是说在进行拷贝时，尚未拷贝的元素被修改了，这就是因为源范围和目标范围重叠导致的；为解决此问题，就需要在源范围和目标范围重叠并且**源范围在目标范围前**时，按下标递减的方式进行拷贝，可以看到**tmove**也是这样处理的：

```c
for (i = n - 1; i >= 0; i--) {
    lua_geti(L, 1, f + i);
    lua_seti(L, tt, t + i);
}
```

还有另外一种重叠的情况：源范围和目标范围重叠并且**源范围在目标范围后**，此时就必须按下标递增的方式进行拷贝；此方式也是**tmove**采取的默认方式，所以**tmove**没有单独对其进行处理:

```c
for (i = 0; i < n; i++) {
    lua_geti(L, 1, f + i);
    lua_seti(L, tt, t + i);
}
```

最后分析一下**tmove**的时间复杂度：

处理在拷贝时会进行表的遍历，其他操作均是常量时间消耗，所以**tmove**的时间复杂度是：
$$
O(n)
$$

# 引用

[**lua5.3参考手册之table.move**](https://www.lua.org/manual/5.3/manual.html#pdf-table.move)