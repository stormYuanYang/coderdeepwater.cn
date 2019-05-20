lua和C一样提供**for**循环，但是lua的**for**不同于C的for，有些地方甚至可以称之为陷阱：昨天在用lua编写一个测试demo时就遇到了。

现在来详细说说昨天遇到的奇怪现象。示例如下：

```c
local t = {1,0,2,0,3,0,4}
for i = 1, #t do
    while i <= #t and t[i] == 0 do
        i = i + 1
    end
    if i > #t then
    	break
    end
    print(t[i])
end
```

打印结果预期的是：

```lua
1
2
3
4
```

但实际上是：

```lua
1
2
2
3
3
4
4
```

也就是说，for循环中的while循环跳过0失败了。为什么会这样呢？循环变量i的值不是被while循环修改了吗？这个问题也让我昨天晚上困惑了一段时间，接着去查阅了lua53的参考手册并阅读lua虚拟机实现源码中关于for的部分，才恍然大悟。

lua的for和C中的for是不同的，实际上你不能修改lua的for循环中的循环变量的值，正如上述代码，修改的实际是i的一个副本，而i的值只会被for修改，也就是说`for v = e1, e2, e3 do block end`等价代码(来自lua53参考手册)：

```lua
do
    local var, limit, step = tonumber(e1), tonumber(e2), tonumber(3)
    if not (var and limit and step) then
        error()
    end
    var = var - step
    while true do
        var = var + step
        if (step >= 0 and var > limit) or (step < 0 and var < limit) then
            break
        end
        local v = var
        block
    end
end    
```

此等价代码和lua虚拟机中实现的C代码大致类似。在for循环中去修改循环变量**var**的值，实际上是修改的**var**的副本**v**的值，这就是为什么在**for**循环中`while i <= #t and t[i] == 0 then i = i + 1 end`不能真正的修改循环变量**i**的值。

记住lua for循环的特点，不要踩中这个陷阱。