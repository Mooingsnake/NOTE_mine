## 关于new
我真的难受死了为什么找不到那个关于C++ 里面不new也是实例化对象new了也能实例化对象
以前写的笔记找也找不到
有new 的话需要自己手动删除，没有new的话就是在当前程序块删除{}

```
void foo()
{
  Point p = Point(0,0);
} // p is now destroyed.

for (...)
{
  Point p = Point(0,0);
} // p is destroyed after each loop
```
来自https://stackoverflow.com/questions/679571/when-to-use-new-and-when-not-to-in-c

## 关于const
需要解决的是为什么C++ 有const和constexpr，链接在下面

在 C++11 以后，建议凡是「常量」语义的场景都使用 constexpr，只对「只读」语义使用 const。

constexpr 是 C++11 引入的，一方面是为了引入更多的编译时计算能力，另一方面也是解决 C++98 的 const 的双重语义问题。在 C 里面，const 很明确只有「只读」一个语义，不会混淆。C++ 在此基础上增加了「常量」语义，也由 const 关键字来承担，引出来一些奇怪的问题。C++11 把「常量」语义拆出来，交给新引入的 constexpr 关键字。

作者：知乎用户
链接：https://www.zhihu.com/question/35614219/answer/798370856
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

## struct 和class
安全性，struct不能禁止访问，class可以


