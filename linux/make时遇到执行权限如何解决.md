今天更新了**skynet**的源代码，然后重新编译其源码，在执行make clean和make时遇到了执行权限的问题。一开始很奇怪，之前一直好好的，怎么突然就遇到权限问题了。看到"**Permission denied**"，以为是当前用户无权限访问文件，然后su root切换到root用户后，执行make clean照样提示"Permission denied"。What？还有这样奇怪的事情，root都没有权限不对劲啊🤨！

其实报错信息早就提示我了，完整的报错信息是"make: **execvp**: ./config.status: Permission denied",注意重点——**execvp**,make告诉你是config.status这个文件不是可执行文件，所以当然不能执行啰，就算你切换成root用户也没用。config.status的属性：

-rw-r--r--   1 yangyuan  staff     4KB  8  1 15:43 config.status

是吧，对于拥有config.status的用户来说config.status的mod只有rw（读写）没有x（可执行）。所以改变config.status的mod，为其加上x属性就OK。

chmod +x config.status

在linux或者Mac OS的command line中执行上述命令即可为config.status增加可执行属性：

-rwxr-xr-x   1 yangyuan  staff     4KB  8  1 15:43 config.status

所以说，分析问题时一定要仔细阅读已有的信息，不然就是失之毫厘差之千里😄。