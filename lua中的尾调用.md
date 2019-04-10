什么是递归呢？一个函数直接或间接的调用自己，就可以称为递归。一般而言，递归得具备如下条件：

+ 子问题和原问题本质上是相同的，甚至问题复杂度更简单；
+ 递归必须有一个出口，不然就成了无限递归。

一个简单的递归函数(下文构造的例子一律不考虑处理异常的问题，仅仅是示例代码，无须面面俱到)：

```lua
--此函数计算1～n的和
local function func(n)
    if n == 1 then
        return 1
    else
        return n + func(n-1)
    end
end
print(func(1000000))
```

当n比较大时，就会出现stack overflow。在我的PC上设置n为1,000,000并执行func函数就会出现**stack overflow**的报错，因为在得到最终结果之前你得保存1,000,000个函数栈，这是一笔不小的开销，是非常容易出现stack overflow的。现在我们利用lua提供的**尾调用**(人们有时也称其为**尾递归**)特性对func函数进行改进：

```lua
--此函数计算1～n的和
local function func1(sum, n)
    if n == 0 then   
        return sum;
    else
        return func1(sum+n, n-1)
    end
end
print(func1(0, 100000000))
```

采用func1函数就不可能再出现stack overflow。当函数最后的操作是调用另一个函数时，lua就不会保存当前函数栈，而是直接复用当前函数栈，这样一来不管你n是1还是1,000,000甚至更大，都不可能出现stack overflow，毕竟在函数func1()执行过程中都只会有一个函数栈存在。你可以将其理解成，执行尾调用时执行了C/C++中的goto语句，跳转到func()第一行开始执行。但是请注意，lua能够进行尾调用的前提是，当前函数的最后操作只能是调用另一个函数，因为这样才能保证，lua不需要保存当前函数的信息。我们构造几个看起来很像尾调用但实际上却不是尾调用的例子：

```lua
-- 此函数简单地遍历n~1
local function func2(n)
    if n == 1 then
        return
    else
        func2(n-1)
    end
end
print(func2(1000000))
```

乍一看func2()十分的正常，是尾调用呀，`func2(n-1)`不就是函数执行的逻辑分支的最后一句吗？是的，表面上看起来确实是这样，但是lua其实在`func2(n-1)`后面会隐式地插入代码`return`，所以`func2(n-1)`不是最后的操作。也就是说func3()也不能算尾调用:

```lua
-- 此函数简单地遍历n~1
local function func3(n)
    if n == 1 then
        return
    else
        func3(n-1)
        return
    end
end
print(func3(1000000))
```

实际上func2()和func3()是等价的。那我显示地返回最后调用的函数不就行了吗？是的，但是只能是调用函数，不能有其他任何操作。最常见的就是将调用函数用作计算表达的一部分，然后返回这个表达式的值，这也不能算尾调用，正如func()的执行结果所示。再如：

```lua
-- 此函数简单地遍历n~1
local function func4(n)
    if n == 1 then
        return 1
    else
        return (func4(n-1))
    end
end
print(func4(1000000))
```

func4()也会导致stack overflow，因为()也是一个操作，不仅仅是为func4(n-1)的返回值加上一对()。我们对func4()进行修改：

```lua
-- 此函数简单地遍历n~1
local function func5(n)
    if n == 1 then
        return 1
    else
        return func5(n-1)
    end
end
print(func5(1000000))
```

func5()就不会导致stack overflow,因为lua能够执行尾调用。

尾调用属于编译器优化的内容，很多其他语言都会做类似的优化。当然如果语言能支持尾调用，固然很好，在熟悉其特性的前提下用好特性，可以加快编程的效率，但是当语言不支持尾调用或者使用尾调用导致代码的杂乱冗余时，我们可以用循环迭代来替代递归，不失为万全之策。

一个lua的尾调用特性都有这么多需要注意的地方，更遑论计算机领域中那浩瀚的知识海洋了。花费大量时间去弄明白这些知识真的值得吗？工欲善其事,必先利其器，对工具愈加熟悉，开发时愈加得心应手；格物致知，探究知识的原理本就是一件快乐的事;不过，学习也是需要适度的，庄子曰：“以有涯随无涯，殆已！”