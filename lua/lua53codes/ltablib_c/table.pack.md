# 简介

lua库函数**table.pack**：

```lua
table.pack(...)
```

**table.pack**的目的是返回所有参数以键$1,2,...,n$填充的新表，并将"n"这个域设置为参数的总数。注意，因为是按参数的逆序填充参数，所以导致这张返回的表不一定是一个序列。

**table.pack**的具体实现是C函数**pack**，接下来对**pack**进行分析。

# 分析

对**pack**进行注解：

```c
static int pack (lua_State *L) {
    int i;
    // 获取参数数量n
    int n = lua_gettop(L);  /* number of elements to pack */
    // 创建一个表,并且重设其大小
    lua_createtable(L, n, 1);  /* create result table */
    // 将这个表插入到 index 1
    lua_insert(L, 1);  /* put it at index 1 */
    // 将所有参数按倒序依次放入表中
    // 因为这里是按倒序将参数放入表中
    // 所以pack返回的表不一定是一个序列
    for (i = n; i >= 1; i--) { /* assign elements */
        lua_seti(L, 1, i);
    }
    // 往栈上压入n
    lua_pushinteger(L, n);
    // 在表中设置t["n"]=n
    lua_setfield(L, 1, "n");  /* t.n = number of elements */
    // 返回1表明，pack只有一个返回值
    return 1;  /* return table */
}
```

**pack**的实现是比较简单的。现在对**pack**的时间复杂度和空间复杂度进行分析：

###### 时间复杂度

**pack**除for循环外，均是常量时间操作；for循环内部的lua_seti()也可认为是常量时间操作；所以**pack**的时间复杂度是：
$$
O(n)
$$

###### 空间复杂度

**pack**会在创建表时，设置其大小；最后会新增一个"n"域；表的大小是和参数数量成正比的；**pack**的空间复杂度是：
$$
O(n)
$$

# 引用

[**lua5.3参考手册之table.pack**](https://www.lua.org/manual/5.3/manual.html#pdf-table.pack)