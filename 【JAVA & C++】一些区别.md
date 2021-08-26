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

https://stackoverflow.com/questions/14116003/difference-between-constexpr-and-const

这里有一个概念是，常量表达式 ！= 常量



