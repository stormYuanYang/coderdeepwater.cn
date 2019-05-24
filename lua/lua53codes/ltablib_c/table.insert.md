# 简介

**table.insert**声明如下：

```lua
table.insert(list, [pos,] value)
```

**table.insert**的目的是在list的位置pos处插入元素value，并后移元素list[pos]，list[pos+1]，...，list[#list]。pos的默认值就是#list+1，因此调用table.insert(t, x)会将x插在列表t的末尾（类似C++中的vector::push_back()）。

lua库函数**table.insert**对应的具体实现是C函数**tinsert**。**tinsert**声明如下：

```c
static int tinsert(lua_State* L);
```

# 分析

**tinsert**源码：

```c
static int tinsert (lua_State *L) {
    /*
     * aux_getn()的作用是获取序列当前的实际长度,
     * aux_getn()+1表示第一个空元素
     */
    lua_Integer e = aux_getn(L, 1, TAB_RW) + 1;  /* first empty element */
    lua_Integer pos;  /* where to insert new element */
    // lua_gettop()获取栈上的参数数量
    switch (lua_gettop(L)) {
        // 只有两个参数，说明一定是在末尾插入元素
        case 2: {  /* called with only 2 arguments */
                    // 确定元素插入位置
                    pos = e;  /* insert new element at the end */
                    break;
                }
        // 有3个参数，说明调用者指定了插入位置
        case 3: {
                    lua_Integer i;
                    // 获取指定的插入位置 pos
                    pos = luaL_checkinteger(L, 2);  /* 2nd argument is the position */
                    // 对插入位置的合法性做一次检查
                    luaL_argcheck(L, 1 <= pos && pos <= e, 2, "position out of bounds");
                    // 将位于插入位置以及其后面的元素，依次向后移动一个位置
                    for (i = e; i > pos; i--) {  /* move up elements */
                        // 取出元素放入栈顶
                        lua_geti(L, 1, i - 1);
                        // 将栈顶元素放入表中
                        lua_seti(L, 1, i);  /* t[i] = t[i - 1] */
                    }
                    break;
                }
        default: {
                     return luaL_error(L, "wrong number of arguments to 'insert'");
                 }
    }
    // 将指定元素赋值到指定位置
    lua_seti(L, 1, pos);  /* t[pos] = v */
    return 0;
}
```

关于代码的解释已由注释说明，不在赘述。现在分析一下**tinsert**的时间复杂度：

1. 当插入位置**pos**就是**e**时，**tinsert**只有常量时间操作；
2. 当插入位置**pos**不是**e**时，**tinsert**会将范围$[pos,e-1]$的元素依次移动到$[pos+1,e]$，需要线性时间操作；
3. 不论插入位置**pos**在何处，执行**lua_seti()**时都可能会导致表扩容；表扩容时，**lua_seti()**的时间复杂度是$O(n)$；表未扩容时，**lua_seti()**的时间复杂度是$O(1)$；但是依据**摊还分析**，可以认为**lua_seti**的时间复杂度就是$O(1)$。

由以上3点，我们可以得出以下结论：

1. 使用**table.insert**在序列末尾插入元素时，**table.insert**的时间复杂度是$O(1)$；
2. 使用**table.insert**在序列内部插入元素时，**table.insert**的时间复杂度是$O(n)$。

# 引用

[**lua5.3参考手册之table.insert**](https://www.lua.org/manual/5.3/manual.html#pdf-table.insert)