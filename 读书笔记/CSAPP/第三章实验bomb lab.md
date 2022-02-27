主要参考资料：https://zhuanlan.zhihu.com/p/138448999


gdb如何使用：

http://www.gdbtutorial.com/tutorial/commands

![image](https://user-images.githubusercontent.com/47411365/155878460-9e4761f7-6025-456c-9b1a-9442357a1e94.png)


一些比较重要的寄存器：
```%rax  用来返回（return）```

```%rdi  %rsi  %rdx %rcx 函数前1，2，3，4个参数专用寄存器 ```

```%rsp 栈指针 stack pointer```

```%rbp %rbx 被调用者保存```

一些比较重要的查看指令：

layout asm 调出汇编代码窗口

layout reg 调出寄存器窗口

<img width="501" alt="image" src="https://user-images.githubusercontent.com/47411365/155882624-92d7b4f0-b9f7-4e07-9bad-26be4805cf3e.png">

一些比较重要的gdb指令
break main （break到main函数） break \*0x400ee9 (break到这个地址)

r （run到下一个断点）

c （continue到下一个断电）

s （step  或者单步执行（c语言））

si （step instruction 按照汇编指令单步执行）

n （next 单步执行但是不进函数内）

kill （结束本次运行）

print  (\*char) 0x1546876  查看这个地址开始的字符串
