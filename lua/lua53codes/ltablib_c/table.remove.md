# 简介

lua库函数**table.remove**的声明如下：

```lua
table.remove (list [, pos])
```

**table.remove**的目的是移除list中位于pos处的元素，并返回这个元素：

+ 当pos在$[1,\#list]$范围内时，**table.remove**将list[pos+1]，list[pos+2]，...，list[#list]向前移动，并移除list[#list]；
+ 当#list等于0时，pos可以是0，此时**table.remove**并不移除元素，直接返回nil；
+ 当pos等于#list+1时，**table.remove**并不移除元素，直接返回nil。
+ 除此之外，pos就是非法的。

使用**table.remove**不传递pos时，默认删除位置#list处的元素，也就是删除最后一个元素。

**table.remove**对应的具体实现是C函数**tremove**，**tremove**的声明如下：

```c
static int tremove (lua_State *L);
```

# 分析

注解**tremove**：

```c
static int tremove (lua_State *L) {
    // aux_getn()获取序列当前的长度
    lua_Integer size = aux_getn(L, 1, TAB_RW);
    // 获取位置pos，默认设置为size
    // 这意味着默认移除表中的最后一个元素
    lua_Integer pos = luaL_optinteger(L, 2, size);
    if (pos != size) { /* validate 'pos' if given */
        // 对位置pos的合法性做一次检查
        // 注意pos可以是size+1 此时不会移除序列中的元素
        luaL_argcheck(L, 1 <= pos && pos <= size + 1, 1, "position out of bounds");
    }
    // 将被移除的元素入栈 这个值就是tremove的返回值
    // 当pos == size+1时，这个值就是nil
    lua_geti(L, 1, pos);  /* result = t[pos] */
    // 依次将元素向前移动一个位置
    for ( ; pos < size; pos++) {
        // 获取下标[pos+1]对应的元素，将其入栈
        lua_geti(L, 1, pos + 1);
        // 将通过lua_geti()函数获取的[pos+1]对应的元素，设置到[pos]处
        lua_seti(L, 1, pos);  /* t[pos] = t[pos + 1] */
    }
    // 将nil入栈
    lua_pushnil(L);
    // 将指定位置pos处的元素，设置为nil
    // 达到最终的移除效果
    lua_seti(L, 1, pos);  /* t[pos] = nil */
    // 返回值的数量 1个
    return 1;
}
```

由上述源码可知：

+ #list等于0时，如果移除t[0]，此时**tremove**的时间复杂度是$O(1)$；
+ pos等于#list或者pos等于#list+1时，**tremove**的时间复杂度是$O(1)$；
+ pos在$[1,\#list)$范围内时，**tremove**会移动表中的元素，其时间复杂度是$O(n)$。

注意，在调用**table.remove**时，如果每次都移除序列前面的元素，其效率是非常低下的，例如调用**table.remove(list,1)**将移动#list-1个元素。

# 引用

[**lua5.3参考手册之table.remove**](https://www.lua.org/manual/5.3/manual.html#pdf-table.remove)