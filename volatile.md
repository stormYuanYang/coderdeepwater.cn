今天阅读[**Skynet**](https://github.com/cloudwu/skynet)源代码时，遇到了关键词**volatile**，上次遇到这个关键词还是在java的jdk源码中。是的，C语言和java中都有关键词volatile，其含义和作用大体相同。volatile，本意是指易挥发的、不稳定的和易爆炸的。是不是让你想到了中学的化学实验，没错，这个单词在化学领域中用得很频繁。在高级编程语言(比如C和java)中，关键词volatile用于修饰变量，表示变量是易改变之意。接下来，简单介绍volatile在java和C中的具体作用。

总所周知，java是一门对多线程支持较好的高级编程语言，其拥有自己的**GC**(Garbage Collector即垃圾回收)策略、**内存模型**。java为保证基本的线程安全，为其内存模型建立了一系列的数据存取的规则(大概一年以前我阅读过一本名为**《深入理解Java虚拟机》**的书籍，此书讲了很多实现java的规则、策略。其实好多规则的概念我已记不太清楚啦，这也不是本文重点-_-)。

在java的内存模型中，有一个主内存(不是内存条-_-，是一个运行的java程序内部的结构)，各个线程有自己的本地内存(当然是做数据缓存用啦)。读数据时，一个线程一般直接从自己的本地内存读取，显然这样更快。当你使用volatile修饰一个变量时，线程在读取此变量时，每次都必须重新从主内存获取，即每次都获取最新的数据。volatile还有一个作用，阻止java编译器的某些优化。譬如，我曾经遇到过的一种情形：

```java
static int MAX_PLAYER_NUM = 5000;
```

MAX_PLAYER_NUM是服务器中的一句配置代码。当修改MAX_PLAYER_NUM的值并对此代码所在的java文件进行热更替换时，发现替换之后MAX_PLAYER_NUM并没有改变。原因就是编译器对代码进行优化时，发现MAX_PLAYER_NUM并没有被其他任何代码修改，所以编译器就会假设MAX_PLAYER_NUM一直不会变化，就直接将MAX_PLAYER_NUM硬编码到代码中。所以说，热更了MAX_PLAYER_NUM的值是不起作用的。

C语言是一门古老的语言。C语言的内存模型就没有java的那么复杂了，而且也没有GC。当volatile修饰变量时，变量和编译器之间就有了如下对话：

+ volatile变量小V：“喂，编译器吗？我是被volatile修饰的变量小V啦。注意哦，我不只是能被当前执行的代码修改，也有可能被计算机中的其他系统修改哦。切记，切记。”
+ 编译器：“亲爱的小V，你说的情况我已收到，我会正确处理你说的情况。我对你所在的代码文件做优化时，会谨慎考虑的。请放心。”

让我们举一个例子(test.c)：

```c
static int a = 0;
void func() {
    while (a == 1) {
    }
}
```

使用gcc对上述代码采用优化等级3进行编译：

```shell
gcc -S -O3 test.c
```

查看编译出的汇编代码(test.s)：

```assembly
	.file	"test.c"
	.text
	.p2align 4,,15
	.globl	func
	.type	func, @function
func:
.LFB0:
	.cfi_startproc
	rep ret
	.cfi_endproc
.LFE0:
	.size	func, .-func
	.ident	"GCC: (Debian 7.3.0-19) 7.3.0"
	.section	.note.GNU-stack,"",@progbits
```

我们看func()对应的汇编代码，直接就返回了。当编译器发现a没有被任何其他代码改变的可能，其就会假设a是不会发生变化，所以进一步对func()进行优化，a和循环语句就被丢弃了。当a的值被其他系统改变后，func()的逻辑亦不会发生变化，这很可能不是你想要的。我们对a加加上volatile:

```c
static volatile int a = 0;
void func() {
    while (a == 1) {
    }
}
```

同样的，使用gcc对上述代码采用优化等级3进行编译：

```shell
gcc -S -O3 test.c
```

查看编译出的汇编代码(test.s)：

```assembly
	.file	"test.c"
	.text
	.p2align 4,,15
	.globl	func
	.type	func, @function
func:
.LFB0:
	.cfi_startproc
	.p2align 4,,10
	.p2align 3
.L2:
	movl	a(%rip), %eax
	cmpl	$1, %eax
	je	.L2
	rep ret
	.cfi_endproc
.LFE0:
	.size	func, .-func
	.local	a
	.comm	a,4,4
	.ident	"GCC: (Debian 7.3.0-19) 7.3.0"
	.section	.note.GNU-stack,"",@progbits
```

当我们为a加上volatile后，就会阻止编译器对func()进行的上文提到的优化。编译出的func()中就可看到编译出的循环语句**.L2**了。

从效率上来说，在java和C中不使用volatile存取变量时的效率肯定是要比起使用volatile存取变量高的。虽然，volatile自有其适用情景，但是现在许多C/C++ groups不再建议使用volatile，因为volatile并不能保证数据存取操作的原子性。据我所知，C++已有更好的解决方案-_-。