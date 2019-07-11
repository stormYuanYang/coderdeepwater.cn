# switch的孤独

“我还以为你从来都不会选我呢？”，switch孤独地呢喃着。

在C语言中，关于条件判定、分支执行，我们使用的最多的应该就是`if else`语句了吧；是不是都快忘记**switch**，switch同样地可以进行条件判定和分支执行，甚至很多时候，switch让代码更加灵活和高效。

# 为什么switch更高效？

switch会在可能时，采用**跳转表**的方式来实现代码的跳转；这比起写一大堆的`if else`的效率来得高得多。“talk is cheap，show me the code”，让我们直接看例子：

```c
void func(int x) {
    switch (x) {
        case 1001:
            x = 1001;
            break;
        case 1021:
            x = 1021;
            break;
        case 1031:
            x = 1031;
        case 1041:
            x = 1041;
            break;
        case 1051:
            x = 1051;
            break;
    }
}
```

如果func()用`if else`来进行代码跳转，看起来大概是这样子：

```c
void func(int x) {
    if (x == 1001) {
        x = 1001
    } else if (x == 1021) {
        x == 1021
    } else if (x == 1031 || x == 1041) {
        x = 1041;
    } else if (x == 1051) {
        x = 1051;
    } else {
    } // do nothing
}
```

如果x等于1051的话，采用`if else`需要进行4次跳转才能执行`x = 1051;`；而采用switch则只需要2次判断，即可（分支数量不会改变switch的判断次数，而随着分支数量的增加`if else`的执行效率就会严重下降）:

+ switch首先会判断x是否大于1051，如果x大于1051直接退出switch语句；
+ 如果x等于1051，那么switch会直接跳转到`case 1051`处执行代码。同样地x等于1001，switch会直接跳转到`case 1001`处执行代码，依次类推。

如果我们将1001到1051之间的所有分支填满，此时`if else`语句的数量是异常多的，并且当x等于1051时，func()需要进行51次左右的判断；但是switch仍然只需要2次判断。此时，`switch`的执行效率和`if else`的执行效率一比较，高下立判。那么，switch是如何做到如此高效的进行代码跳转的呢？答案是**跳转表**。下面用一段伪代码来表示：

```c
void func(int x) {
    int jump_table[51];
    jump_table[1001-1001] = 代码case 1001的地址
    jump_table[1002-1001] = 跳出switch
    ... 
    jump_table[1021-1001] = 代码case 1021的地址
    jump_table[1022-1001] = 跳出switch
    ...
    jump_table[1031-1001] = 代码case 1031的地址
    jump_table[1032-1001] = 跳出switch
    ...
    jump_table[1041-1001] = 代码case 1041的地址
    jump_table[1042-1001] = 跳出switch
    ... 
    jump_table[1051-1001] = 代码case 1051的地址

    跳转表会尝试对case的值进行操作，将其值对应到
    跳转表的下标；下标对应的就是应该执行的代码地址；
    以此达到快速执行代码跳转的目的

    switch (x) {
        case 1001:
            x = 1001;
            break;
        case 1021:
            x = 1021;
            break;
        case 1031:
            x = 1031;
        case 1041:
            x = 1041;
            break;
        case 1051:
            x = 1051;
            break;
    }
}
```

switch通过对case的值进行计算，将所有case的代码执行地址映射到一个数组中去，这样就可以根据x的值直接进行代码跳转了，非常灵活而高效。那么，GCC真的是通过上述伪代码类似的跳转表来实现switch语句的吗？在linux下对使用switch的func()进行编译（只编译成汇编代码，执行命令gcc -S func.c，注意某些低版本的gcc不会用跳转表实现switch，笔者自测gcc4.4版本以上会使用跳转表实现switch，笔者使用macbook pro中的gcc 4.2版本并未使用跳转表实现switch）:

```assembly
	.file	"func.c"
	.text
	.globl	func
	.type	func, @function
func:
.LFB0:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	movl	%edi, -4(%rbp)
	movl	-4(%rbp), %eax
	subl	$1001, %eax #减去1001
	cmpl	$50, %eax# 大于50直接退出switch
	ja	.L9
	movl	%eax, %eax
	leaq	0(,%rax,4), %rdx
	leaq	.L4(%rip), %rax
	movl	(%rdx,%rax), %eax
	movslq	%eax, %rdx
	leaq	.L4(%rip), %rax
	addq	%rdx, %rax
	jmp	*%rax
	.section	.rodata #只读数据
	.align 4
	.align 4
.L4:#这个就是跳转表了
	.long	.L3-.L4#可以看到这里对应case 1001
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L5-.L4# 这里对应case 1021
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L6-.L4# 这里对应case 1031
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L7-.L4# 这里对应case 1041
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L9-.L4
	.long	.L8-.L4# 这里对应case 1051
	.text
.L3:# case 1001
	movl	$1001, -4(%rbp)
	jmp	.L2
.L5:# case 1021
	movl	$1021, -4(%rbp)
	jmp	.L2
.L6:# case 1031
	movl	$1031, -4(%rbp)# .L6执行完，继续执行.L7
.L7:# case 1041
	movl	$1041, -4(%rbp)
	jmp	.L2
.L8:# case 1051
	movl	$1051, -4(%rbp)
	nop
.L2:
.L9:
	nop
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE0:
	.size	func, .-func
	.ident	"GCC: (Debian 7.3.0-19) 7.3.0"
	.section	.note.GNU-stack,"",@progbits
```

gcc编译后的汇编代码，非常清晰，其中的`.L4`就是跳转表了，`.L4`中出现最多的就是`.L9-.L4`，这就是未出现在switch中的case，默认都是跳出switch语句。

# switch名言：“我不是万能的”

“亲爱的程序员，我不是万能的。”

由上文可以，在跳转分支较多时，switch比起`if else`是有很大优势的，但是switch也不是万能的。请你思考一下：

使用跳转表固然大幅提升了代码跳转的效率，但是使用跳转表本身就是一种消耗。如果我的case范围差距非常大，那我的跳转表岂不是也会非常大？如果我的func()是这样的：

```c
void func(int x) {
    switch (x) {
        case 1001:
            x = 1001;
            break;
        case 1021:
            x = 1021;
            break;
        case 1031:
            x = 1031;
        case 1041:
            x = 1041;
            break;
        case 2000:
            x = 2000;
        case 3000:
        	x = 3000;
            break;
    }
}
```

那我的跳转表岂不是要容纳3000个执行地址？这样岂不是会得不偿失。请你放心，51已经是switch的极限了（针对笔者当前测试的数据而言，这也是为什么上述例子中的case最小值是1001，case最大值是1051），一旦计算后的case范围大于51，switch就会放弃使用**跳转表**，转而使用普通的条件分支判断（就像`if else`那样）。你可以将`case 51`中的51改为52，再使用gcc进行编译，查看汇编代码：跳转表就不见了。

# switch的赠语

“亲爱的程序员，我在大多数情况下都能比`if else`更加高效和灵活哦，请不要忘记我:-)”。