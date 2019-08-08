###### profile

**index2addr**是lapi.c中的内部函数，其声明可视作:

```c
static TValue *index2addr (lua_State *L, int idx);
```

index2addr的目的是通过传入的`int`类型的idx下标得到idx对应的`TValue*`类型的指针。

###### 代码分析

index2addr的实现如下：

```c
static TValue *index2addr (lua_State *L, int idx) {
    CallInfo *ci = L->ci;
    if (idx > 0) {
        TValue *o = ci->func + idx;
        api_check(L, idx <= ci->top - (ci->func + 1), "unacceptable index");
        if (o >= L->top) return NONVALIDVALUE;
        else return o;
    }
    else if (!ispseudo(idx)) {  /* negative index */
        api_check(L, idx != 0 && -idx <= L->top - (ci->func + 1), "invalid index");
        return L->top + idx;
    }
    else if (idx == LUA_REGISTRYINDEX)
        return &G(L)->l_registry;
    else {  /* upvalues */
        idx = LUA_REGISTRYINDEX - idx;
        api_check(L, idx <= MAXUPVAL + 1, "upvalue index too large");
        if (ttislcf(ci->func))  /* light C function? */
            return NONVALIDVALUE;  /* it has no upvalues */
        else {
            CClosure *func = clCvalue(ci->func);
            return (idx <= func->nupvalues) ? &func->upvalue[idx-1] : NONVALIDVALUE;
        }
    }
}
```

3到8行代码。当idx大于0时，则直接通过第4行代码`TValue *o = ci->func + idx;`，求得idx对应的`TValue*`指针o。通过执行第5行代码`api_check(L, idx <= ci->top - (ci->func + 1), "unacceptable index");`对idx值的合法性做一次断言判断，即idx必须满足**0 < idx <= (ci->top - 1) - ci->func**。注意第6到7行的`if else`判断,如果**idx>=L->top**，此时则一定满足**L->top <= o < ci->top**，那么此时o指向的地址上是没有有效值的，所以应当返回常量NONVALIDVALUE;否则，直接返回o。

9到12行代码。此时idx是一个负数，并且idx满足`ispseudo(idx)`执行结果为假，这意味着idx是指向栈上的一个负数索引。类似地，通过执行第10行代码`api_check(L, idx != 0 && -idx <= L->top - (ci->func + 1), "invalid index");`对idx值的合法性做一次断言判断，即idx必须满足**0 < -idx <= (L->top - 1) - ci->func**；注意，当传入负数idx时，idx的绝对值$|idx|$是小于**(L->top - 1) - ci->func**的，此时执行第11行代码`return L->top + idx;`得到的指针指向的地址都是拥有有效值的，所以无须像idx大于0时那样，再判断idx的范围。

13到14行代码。如果idx直接等于LUA_REGISTRYINDEX：直接返回**&G(L)->l_registry**即可。

15到24行代码。剩下的这一种情况，就说明idx索引的就只能是upvalue了。通过执行第16行代码`idx = LUA_REGISTRYINDEX - idx;`，计算得到新的idx；通过执行第17行代码`api_check(L, idx <= MAXUPVAL + 1, "upvalue index too large");`对idx的值的合法性做断言判断，此时idx必须满足**0 < idx <= MAXUPVAL + 1**。18到23行代码，对当前栈上的函数类型做判断，如果`ci->func`是light c函数，则直接返回常量NONVALIDVALUE，因为light c函数没有upvalues，仅仅是一个存粹的过程函数，不会保存额外的信息；否则，`ci->func`一定是一个c闭包（可以保存额外信息），执行第22行代码`return (idx <= func->nupvalues) ? &func->upvalue[idx-1] : NONVALIDVALUE;`将idx和func->nupvalues做一次比较，如果idx<=func->nupvalues，则idx能索引到有效值，那么返回&func->upvalue[idx-1]；否则返回常量NONVALIDVALUE。

###### 总结

index2addr是一个简单函数，只在lapi.c中被使用。index2addr最坏时间复杂度为：
$$
O(1)
$$

###### 备注

代码中出现的一些宏定义：

\#define ispseudo(i)     ((i) <= LUA_REGISTRYINDEX)

\#define LUA_REGISTRYINDEX   (-LUAI_MAXSTACK - 1000)

\#define MAXUPVAL  255