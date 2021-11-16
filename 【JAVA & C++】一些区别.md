## C++11 的新特性 
from:leetcode C++面试突击（付费内容，我也没怎么看懂）

1.```auto``` 类型推导

注意：编译器推导出来的类型和初始值的类型并不完全一样，编译器会适当地改变结果类型使其更符合初始化规则。

2.```decltype``` 类型推导

关键字：decltype 是“declare type”的缩写，译为“声明类型”。和 auto 的功能一样，都用来在编译时期进行自动类型推导。如果希望从表达式中推断出要定义的变量的类型，但是不想用该表达式的值初始化变量，这时就不能再用 auto。decltype 作用是选择并返回操作数的数据类型。

二者区别：
```
auto var = val1 + val2; 
decltype(val1 + val2) var1 = 0; 
```
3.```lambda```表达式
耳熟能详了我不介绍了
```
[capture list] (parameter list) -> return type
{
   function body;     
};
```
4.for循环
```
for (declaration : expression){
    statement
}
```
5.右值引用

6.标准库 move() 函数

7.智能指针

8.delete 函数和 default 函数
```
#include <iostream>
using namespace std;

class A
{
public:
	A() = default; // 表示使用默认的构造函数
	~A() = default;	// 表示使用默认的析构函数
	A(const A &) = delete; // 表示类的对象禁止拷贝构造
	A &operator=(const A &) = delete; // 表示类的对象禁止拷贝赋值
};
int main()
{
	A ex1;
	A ex2 = ex1; // error: use of deleted function 'A::A(const A&)'
	A ex3;
	ex3 = ex1; // error: use of deleted function 'A& A::operator=(const A&)'
	return 0;
}

```
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


