## 编译到链接的八股文
下文来源：leetcode https://leetcode-cn.com/leetbook/read/cpp-interview-highlights/e4ns5g/
、
__编译预处理__：处理以 # 开头的指令（#define ，#include， #pragma once）；

__编译优化__：将源码 .cpp 文件翻译成 .s 汇编代码；

__汇编__：将汇编代码 .s 翻译成机器指令 .o 文件；

__链接__：汇编程序生成的目标文件，即 .o 文件. 链接并不会立即执行，因为可能会出现：.cpp 文件中的函数引用了另一个 .cpp 文件中定义的符号或者调用了某个库文件中的函数。那链接的目的就是将这些文件对应的目标文件连接成一个整体，从而生成可执行的程序 .exe 文件。

两种链接优缺点：

静态链接：浪费空间，每个可执行程序都会有目标文件的一个副本，这样如果目标文件进行了更新操作，就需要重新进行编译链接生成可执行程序（更新困难）；优点就是执行的时候运行速度快，因为可执行程序具备了程序运行的所有内容。

动态链接：节省内存、更新方便，但是动态链接是在程序运行时，每次执行都需要链接，相比静态链接会有一定的性能损失。

下面的评论（ljmiao）：

静态链接是将程序调用的库一起打包到可执行文件中，这样执行时就不需要调用别的库了，速度快，但是链接的时候可能同一个库链接了好几次，导致空间浪费，而且如果该库更新了的话，整个程序需要重新编译；

动态链接是在程序执行时才载入引用的库，因此方便更新。
![image](https://user-images.githubusercontent.com/47411365/141247854-81a1b12d-72aa-42ac-a24d-cb1195a664bb.png)

![image](https://user-images.githubusercontent.com/47411365/141247890-605fe1d8-6436-456a-993a-37c4f4abb5b2.png)

## 编译和链接的区别
来源：https://www.bilibili.com/video/BV1VJ411M7WR?p=7

lnk2019：当前文件里声明了，我就是链接不到它老家（定义）在哪， 1.检查库有没有少加 2.检查定义的函数返回值，参数列表，函数名是否匹配

c2084：我只负责编译当前文件，但你连续声明了两次同样名字的函数

c3861： identifier not found 我编译文件的时候没看到你声明了。

link has already defined：你在多个文件重复定义了同样名字的函数

```
void f2(){}
int main() {
   f();    // C3861
   f2();   // OK
}
```
```
extern char B[100];   // B isn't available to the linker  在外部没有char B的定义
int main() {
   B[0] = ' ';   // LNK2019
}
```

b站的那个错误我没有重现出来，反正是看视频的，不过也不考，害。



## 宏定义
### 一个简单的函数宏定义
```
#define LOG(X) std::cout << X << std::endl  
```
X将会被替换成实际代码中的变量
### visual studio 下的骚操作
![image](https://user-images.githubusercontent.com/47411365/122428596-a002a680-cfc4-11eb-82f2-e315c77cc97f.png)

首先这样设置，当然PRDEBUG 后面没有=1(另外中间不要加空格)
```
#include <iostream>
#include <string>

#ifdef PR_DEBUG   //这里需要去project-设置-c++-debug那里设置，需要在preprocesser那里注册一个 PR_DEBUG 的 tag，表示在debug状态下替换成这样
#define LOG(X) std::cout << x << std::endl
#else //如果不在debug状态下，替换空，就是删除
#define LOG(X)             
#endif

int main()
{
  LOG("HELLO");
  std::cin.get();
}
```
这是一个版本
优化版本是，上图设置里面改成=1
```
#if PR_DEBUG  == 1
#define LOG(X) std::cout << x << std::endl
#else //如果不在debug状态下，替换空，就是删除
#define LOG(X)             
#endif
```

另外一个优化版本是
```
#if PR_DEBUG  == 1
#define LOG(X) std::cout << x << std::endl
#elif define(PR_RELEASE) //如果不在debug状态下，替换空，就是删除
#define LOG(X)             
#endif
```

如果要在宏定义里面换行
 ```
 #define MAIN int main() \
 {\
  std::cin.get();\
 }
 ```
 
 ## 关于宏定义和link
 ### 第一个问题，A.h 和B.h中都写了LOG（x）函数，main.cpp include了它们，会报错吗？
   __不会__
 这个宏定义本质没有定义LOG（）函数，编译的时候自动替换成了std::cout << X << std::endl  
 
 ### 问题2，#pragma once 和#ifdef ,#define, #endif能干什么
 防止你忘记，看一下写法
 ```
 //a.h文件
 #ifndef __a_h__
#define __a_h__
void funcA(void);   // 声明
void funcA(void)    // 定义
{}
#endif
```
__能防止同名文件重复引入，不能防止文件中的同名函数重复引入__ 

### 问题3,不容易出错的写法是什么样的(面向函数写法)
以下是来自其他网站的摘抄
https://jiadebin.github.io/2017/04/03/%E5%A4%B4%E6%96%87%E4%BB%B6%E4%B8%AD%E5%AE%9A%E4%B9%89%E5%87%BD%E6%95%B0%E5%BC%95%E5%8F%91%E7%9A%84multiple-definition/
编译的时候，C++是采用独立编译，就是每个cpp单独编译成对应的.o文件，最后linker再将多个.o文件链接成可执行程序。所以从编译的时候，从各个cpp文件看，编译没有任何问题。但是能发现一个问题，b.o中声明和定义了一次funcA()，c.o中也声明和定义funcA()，这就是编译器报重复定义的原因。
这个作者举的例子里面，C.cpp和B.cpp都引用了A.h ,结果C.cpp和B.cpp因为main.cpp(甚至include的是BC的头文件)，也被链接起来了，所以重复定义了

__项目里如果只有单个.cpp文件，.h里面的函数可以被定义__
__如果不止一个.cpp文件，多个.cpp文件(中的函数)最后被采用到一个.cpp文件中，形成了cpp级的链接，.h里面函数只声明，在.cpp文件下定义__

### 问题4，不容易出错的写法是什么样的（面向class写法）

```
//ray.h文件
#ifndef RAY_H
#define RAY_H

#include "vec3.h"

class ray {
    public:
        ray() {}
        ray(const point3& origin, const vec3& direction)
            : orig(origin), dir(direction)
        {}

        point3 origin() const  { return orig; }
        vec3 direction() const { return dir; }

        point3 at(double t) const {
            return orig + t*dir;
        }

    public:
        point3 orig;
        vec3 dir;
};

#endif
```
__一个.h文件一个class，.h当作工具集，最后只用一个.cpp文件运行__
可以去看panda3d的那个程序，或者ray tracing的那个程序，都是单.cpp结构

### 问题5，可以只写一个.cpp文件吗
https://stackoverflow.com/questions/56015870/what-is-the-purpose-of-using-multiple-cpp-files-in-one-solution
结论：多人项目可能不太容易，如果要使用一种vec —> ray ->main  结构，还好解决，还是全.h
如果要写什么.cpp测试文件，要注意结构。




