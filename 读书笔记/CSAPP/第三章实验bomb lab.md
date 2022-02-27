主要参考资料：https://zhuanlan.zhihu.com/p/138448999


gdb如何使用：

http://www.gdbtutorial.com/tutorial/commands

![image](https://user-images.githubusercontent.com/47411365/155878460-9e4761f7-6025-456c-9b1a-9442357a1e94.png)


一些比较重要的寄存器：
```%rax  用来返回（return）```

```%rdi  %rsi  %rdx %rcx 函数前1，2，3，4个参数专用寄存器 ```

```%rsp 栈指针 stack pointer```

```%rbp %rbx 被调用者保存```

一些比较重要的查看：

layout asm 调出汇编代码窗口

layout reg 调出寄存器窗口

<img width="501" alt="image" src="https://user-images.githubusercontent.com/47411365/155882624-92d7b4f0-b9f7-4e07-9bad-26be4805cf3e.png">
