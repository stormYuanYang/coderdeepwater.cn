# 简介

lua库函数**table.concat**的声明如下：

```lua
table.concat(list [, sep [, i [, j]]])
```

顾名思义，**table.concat**的目的是将序列list中指定的的元素连接成一个字符串。**table.concat**允许你传递1到4个参数：

+ 只传入list，表明将序列中全部元素连接成一个字符串（范围：[1,#list]），以空字符串""做分隔符；
+ 传入list、sep，表明将序列中全部元素连接成一个字符串（范围：[1,#list]），以**sep**做分隔符；
+ 传入list、sep、i，表明将序列中范围为[i,#list]的元素连接成一个字符串，以**sep**做分隔符；
+ 传入list、sep、i、j，表明将序列中范围为[i,j]的元素连接成一个字符串，以**sep**做分隔符；

注意，可选项参数都可以赋nil值，此时**table.concat**会启用默认值：

+ **sep**默认就是""；
+ **i**默认就是1；
+ j默认就是#list。

使用**table.concat**需要你保证被连接的元素都可以转换成字符串（常见的就是字符串和数字），否则**table.concat**会报告错误。

实现**table.concat**的C函数就是**tconcat**，下文将对**tconcat**进行分析。

# 分析

首先注解**tconcat**和其辅助函数**addfield**。

**addfield**:

```c
// 将t[i]转换成字符串并追加到Buffer b中
static void addfield (lua_State *L, luaL_Buffer *b, lua_Integer i) {
    // 获取下标为i对应的元素,并将其入栈
    lua_geti(L, 1, i);
    // 判断栈顶元素是否是字符串或者能否转换成字符串 
    // 如果不能，就报错
    if (!lua_isstring(L, -1)) {
        luaL_error(L, "invalid value (%s) at index %d in table for 'concat'",
                luaL_typename(L, -1), i);
    }
    // 将字符串追加到b中
    // 或者将能转换成字符串的元素转换成字符串再追加到b中
    luaL_addvalue(b);
}
```

**addfield**的目的就是将序列中指定下标的元素转换成字符串后，追加到Buffer中。

**tconcat**：

```c
static int tconcat (lua_State *L) {
    luaL_Buffer b;
    // 获取序列中最后一个元素的下标，也是整个序列的长度
    lua_Integer last = aux_getn(L, 1, TAB_R);
    size_t lsep;
    // 从栈上获取分隔符，默认分隔符是空字符串-->""
    const char *sep = luaL_optlstring(L, 2, "", &lsep);
    // 获取concat的第一个元素下标，默认是1
    lua_Integer i = luaL_optinteger(L, 3, 1);
    // 获取last的值，默认就是最后一个元素的下标
    last = luaL_optinteger(L, 4, last);
    // 初始化缓存b,luaL_buffinit实际上并没有分配内存
    // 只是设置了一些初始值
    luaL_buffinit(L, &b);
    // 遍历：将序列中的元素和分隔符添加到缓存b
    for (; i < last; i++) {
        // 添加序列中的元素
        // 实际上是将t[i]先转换成字符串再追加到b中
        // 当b的容量不够时，b会自动扩容
        addfield(L, &b, i);
        // 添加分割符
        luaL_addlstring(&b, sep, lsep);
    }
    // 因为有分隔符的存在，所以这里只能单独地添加t[last]
    // 因为t[last]后不会有分隔符了
    if (i == last)  /* add last value (if interval was not empty) */
        addfield(L, &b, i);
    // 将缓存b中的字符串入栈，得到concat的最终结果
    luaL_pushresult(&b);
    // 返回1，表明concat就一个返回值
    return 1;
}
```

**tconcat**采用缓存的思想：分配一块内存Buffer，不停地追加字符串到这个Buffer中；当前Buffer不足以容纳新的字符串时，就将开辟新的容量翻倍的NewBuffer（如果容量翻倍之后还是不足以容纳新追加的字符串，则开辟足以容纳新字符串的容量），将已存放到Buffer中的字符串，拷贝至NewBuffer，然后将新字符串拷贝至NewBuffer。

比起每连接一个字符串就分配新的内存，这样做的好处就是，避免连接的元素过多时分配大量的内存。例如在JAVA中使用String类型的字符串进行连接操作，都会分配至少两个字符串长度的内存大小（实际上肯定不止，因为String类头部还会占用若干字节）：如果你连接100个字符串，每个字符串的大小都为1MByte，那么你在循环连接这100个字符串时，总共会分配$2+3+...+100$MByte的内存，也就是$5049$MByte（大概$5$GByte）的内存。JAVA也考虑到了这个问题，所以其推出了StringBuffer类；StringBuffer在连接字符串时，就会采用**缓存机制**了。

最后来分析一下**tconcat**的时间复杂度和空间复杂度。

## 时间复杂度

**tconcat**会将元素转换成字符串并将其追加到缓存Buffer中：

+ 如果元素不是字符串，比如元素是数字，还得将数字转换成字符串，转换函数的时间复杂度是$O(n)$；
+ 如果元素是字符串，就无须转换；
+ 但是不论元素是不是字符串，最终将字符串追加到缓存Buffer时，都会调用C标准库函数memcpy，虽然说memcpy对其实现做了一定的优化（比如尽量按**字**拷贝、按操作系统的**分页**进行拷贝等），但是其时间复杂度仍认为是$O(n)$。

我们注意到**tconcat**是在一个for循环中追加元素和分隔符的；而**tconcat**的其他操作时间复杂度至多是$O(n)$；所以**tconcat**的时间复杂度就是：
$$
O(n^{2})
$$

## 空间复杂度

因为**tconcat**采用缓存机制处理被连接的字符串，缓存Buffer只会在新追加的字符串无法追加进Buffer时扩容；根据**摊还分析**思想，其空间复杂度应该是：
$$
O(n)
$$
同样的，JAVA中对String类字符串进行连接，其空间复杂度就应该是：
$$
O(n^{2})
$$

# 引用

[**lua5.3参考手册之table.concat**](https://www.lua.org/manual/5.3/manual.html#pdf-table.concat)