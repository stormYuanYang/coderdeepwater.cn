**b_str2int**是一个内部函数，其声明为：

```c
static const char *b_str2int (const char *s, int base, lua_Integer *pn);
```

你是无法直接使用b_str2int的，b_str2int目前只被**luaB_tonumber**调用。b_str2int的目的是将字符串转换成一个整数。b_str2int接受3个参数：

+ 需要转换成整数的字符串s；
+ 整数的基数base；
+ pn用于存放结果。

b_str2int返回一个字符指针，这个指针可为调用者提供转换的部分信息。

---

让我们看b_str2int的实现：

```c
// 空格符的字符集合
#define SPACECHARS	" \f\n\r\t\v"

static const char *b_str2int (const char *s, int base, lua_Integer *pn) {
    lua_Unsigned n = 0;
    int neg = 0;
    s += strspn(s, SPACECHARS);  /* skip initial spaces */
    if (*s == '-') { s++; neg = 1; }  /* handle signal */
    else if (*s == '+') s++;
    if (!isalnum((unsigned char)*s))  /* no digit? */
        return NULL;
    do {
        int digit = (isdigit((unsigned char)*s)) ? *s - '0'
            : (toupper((unsigned char)*s) - 'A') + 10;
        if (digit >= base) return NULL;  /* invalid numeral */
        n = n * base + digit;
        s++;
    } while (isalnum((unsigned char)*s));
    s += strspn(s, SPACECHARS);  /* skip trailing spaces */
    *pn = (lua_Integer)((neg) ? (0u - n) : n);
    return s;
}
```

首先看变量的声明和定义：

```c
lua_Unsigned n = 0;
int neg = 0;
```

b_str2int首先定义n和neg，n用于暂存转换结果，neg用于标志结果的正负。

再往下看：

```c
s += strspn(s, SPACECHARS);
```

这里通过调用C标准库中的**strspn**方法，跳过所有的前置空格符。

继续看：

```c
if (*s == '-') { s++; neg = 1; }
else if (*s == '+') s++;
```

跳过所有空格符后，判断一下第一个字符是否是'-'或者'+'；如果是'-'，表明这是一个负数，将标志neg设置为1；如果是'+'，s只需要继续向后就好。

继续看代码：

```c
if (!isalnum((unsigned char)*s))
    return NULL;
```

如果第一个字符不是字母或者数字，说明字符串s根本无法转换成整数，直接返回NULL。

看接下来的`do{}while()`循环：

```c
do {
    int digit = (isdigit((unsigned char)*s)) ? *s - '0'
        : (toupper((unsigned char)*s) - 'A') + 10;
    if (digit >= base) return NULL;  /* invalid numeral */
    n = n * base + digit;
    s++;
} while (isalnum((unsigned char)*s));
```

计算出***s**对应的字符对应的数值，如果大于等于指定的base，则说明这是一个无效的数字，直接返回；否则更新n的值，s继续向后移动。一旦*s不是数字和字母，就停止循环。

注意，这里并没有考虑整数溢出的情况：当字符串代表的整数长度太长时，转换得到的整数可能是错误的。

继续看代码：

```c
s += strspn(s, SPACECHARS);
```

再次调用strspn，跳过后置的空格。

最后看：

```c
*pn = (lua_Integer)((neg) ? (0u - n) : n);
```

这里根据标志neg的值，得到最终的整数值\*pn。注意，就算在`do{}while()`中没有溢出，这里同样可能会溢出，因为n是unsigned类型(**unsigned LUA_INTEGER**)，\*pn却是signed类型(**LUA_INTEGER**)。

执行完后，返回s。