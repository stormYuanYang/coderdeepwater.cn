print应该是你学习lua语言学会使用的第一个函数，因为你用lua写下的第一句代码一般就是:

```lua
print("Hello World!")
```

现在我们就来看看lua中的print函数到底是如何实现的。当我们调用lua函数print时，实际调用的是C函数luaB_print，其定义如下：

```c
static int luaB_print (lua_State *L) {
    int n = lua_gettop(L);  /* number of arguments */
    int i;
    lua_getglobal(L, "tostring");
    for (i=1; i<=n; i++) {
        const char *s;
        size_t l;
        lua_pushvalue(L, -1);  /* function to be called */
        lua_pushvalue(L, i);   /* value to print */
        lua_call(L, 1, 1);
        s = lua_tolstring(L, -1, &l);  /* get result */
        if (s == NULL)
            return luaL_error(L, "'tostring' must return a string to 'print'");
        if (i>1) lua_writestring("\t", 1);
        lua_writestring(s, l);
        lua_pop(L, 1);  /* pop result */
    }
    lua_writeline();
    return 0;
}
```

luaB_print只接受一个参数**lua_State* L**。读到这里，你应该大体知道lua和C是如何进行数据交换的：通过栈。

luaB_print第一行代码：

```c
int n = lua_gettop(L);  /* number of arguments */
```

源代码中的注释写得很清楚了，获得栈上的参数数量。lua_gettop只是简单地计算了栈顶到当前函数index的差值，当然这个差值就代表了函数的参数数量。

继续往下看：

```c
lua_getglobal(L, "tostring");// lua-->stack
```

这行代码又是在做什么呢？很简单，在lua的全局表中找到名为tostring的函数，并将此函数push到栈上。此时栈上的情形大概是这样：

栈底	......, 	function_index, 	arg1, 	......,		argn, 	tostring,		栈顶

接下来是一个for循环。这个循环的目的是依次将tostring函数和一个打印参数入栈，将打印参数转换成字符串并且出栈tostring和参数，然后打印字符串；打印完毕继续将tostring函数和下一个打印参数入栈，直到所有参数打印完毕。

继续分析代码：

```c
lua_pushvalue(L, -1);
```

这里将栈顶元素push到栈上，也就是说将tostring函数再次push到栈上。栈的示意图：

栈底	......, 	function_index, 	arg1, 	......,		argn, 	tostring,		tostring,		栈顶

那么现在栈的最上面就两个tostring了。

再看下一行代码：

```c
lua_pushvalue(L, i);
```

这里将第i个参数push到栈上。执行完此代码，栈就是这样了：

栈底	......, 	function_index, 	arg1, 	......,		argn, 	tostring,		tostring,		arg1, 	栈顶

继续看：

```c
lua_call(L, 1, 1);// 调用栈顶的函数，参数1个，返回值1个
```

调用lua_call,执行tostring函数将arg1转换成字符串。转换后，tostring和arg1将出栈，然后字符串将入栈，也就是说调用tostring的结果，占据了tostring和arg1原有的位置。执行完lua_call后，栈是这样：

栈底	......, 	function_index, 	arg1, 	......,		argn, 	tostring,		arg1的字符串, 	栈顶

然后执行代码：

```c
s = lua_tolstring(L, -1, &l);  /* get result */
if (s == NULL)
    return luaL_error(L, "'tostring' must return a string to 'print'");
```

将栈上的字符串取出，并检测是否转换成功。

然后执行代码：

```c
if (i>1) 
    lua_writestring("\t", 1);
lua_writestring(s, l);
```

这里判断了一下参数是否不止一个。不止一个时，从第二个参数开始，打印参数前先打印一个制表符，这样打印出比较美观。这里的lua_writestring是一个宏定义，其定义如下：

```c
#define lua_writestring(s,l)   fwrite((s), sizeof(char), (l), stdout)
```

将字符串写入到标准输出流stdout。

然后执行：

```c
lua_pop(L, 1);
```

参数已经打印完毕，参数就没用了，此时就需要将其出栈。所以执行lua_pop。执行玩lua_pop，进入下一次循环（如果能的话），直到所有参数打印完毕。

for循环执行完后，执行：

```c
lua_writeline();
```

lua_writeline()本身是一个宏定义：

```c
#define lua_writeline()        (lua_writestring("\n", 1), fflush(stdout))
```

写入一个换行符，并刷新缓冲区。

至此，luaB_print函数就分析完毕了。luaB_print就是简单地打印传入的所有参数，并用制表符对其分割，打印完所有参数后，换行。