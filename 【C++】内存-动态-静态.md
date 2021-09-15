## 前言-关于sizeof和union和内存对齐

下面是基本类型的对应sizeof：

![image](https://user-images.githubusercontent.com/47411365/133243622-10d6e628-951a-4853-b5bf-7fcb61c5f8fe.png)

我也不知道为什么别人的表格里面long和unsigned long是32位 4， 64位 8，反正我的实测visual studio是这样.

下面是一些例子（sum：55以及之后的部分是我别的例子剩下的，只看前几行纯数字就行）

![image](https://user-images.githubusercontent.com/47411365/133358836-9c1451b9-3548-4557-9fd6-efc7ede83587.png)


1.string在std里面是一个类，计算方法不一样 2.char[9]就是9  3.尽管我认为char a[8]的a是一个指针，但是这里能算出总长  4.所有的指针都是8位占位（我开的x64运行环境） 5."ssss"是char g[]自动增加以为休止位


__一个解释union的网页__ ：https://www.cnblogs.com/jeakeven/p/5113508.html

摘要：
```
struct student
{
     char mark;   // 1
     long num;    // 4
     float score;   // 4
};

...
  sizeof(student);   // 4+4+4 = 12
...
```
```
union test
{
     char mark;
     long num;
     float score;  
};

sizeof(test);  //  4 
```
sizeof(union test)的值为4。因为共用体将一个char类型的mark、一个long类型的num变量和一个float类型的score变量存放在同一个地址开始的内存单元中

### sizeof

__一个把sizeof的东西基本都讲清楚的网页__ ：https://www.cnblogs.com/chio/archive/2007/06/11/778934.html



我看不懂它讲的多级指针部分，但是有一点：strlen("abcd") == 4 ；sizeod("abcd") == 5；因为一个是字符串长度，一个是字符串容量

sizeof("1234")是5，sizeof(string("1234"))是28     // 在我的机器上，在别人的机器上可能是16

![image](https://user-images.githubusercontent.com/47411365/133237120-953122e6-d398-4f64-8726-8d813c99446a.png)

需要明确的一点是，遇到"xxx"的时候，类型其实是左边这个，遇到string("xxxx")的时候作为值可以打印为xxxx，但是sizeof就是28

![image](https://user-images.githubusercontent.com/47411365/133238156-1df3b8b7-e5b0-49b8-ad43-010da614b5b9.png)

__why？__

### sizeof 与struct
struct有默认的对齐方式：

![image](https://user-images.githubusercontent.com/47411365/133356227-97d606d3-f9ac-420e-9d5e-86d538e79604.png)

1.cpu对一个边界的取值会寻找其中占内存最大的类型，比如最大是int，那么边界就是4； 最大时double，那么边界就是8.

2.如果char-》double-》char排列，那么就会出现1（8）-》8（8）-》1（8）最终为24

3.如果char-》char-》double，那么就是两个char可以挤在一个8边界的空间里，所以是16

4.例子里面char和int挤在一起了，所以还是24 和 16

![image](https://user-images.githubusercontent.com/47411365/133356733-c72dae0b-4c62-423c-89c2-aedee6496678.png)![image](https://user-images.githubusercontent.com/47411365/133356750-dd934b17-c7fa-4339-bdcd-482d57f71675.png)






## 简述

内存分三种：

1.栈内存，比如函数{}括号内的部分，在递归的时候能看的很清楚是通过栈的方式运行。将会在栈弹出的时候自行销毁

```
foo(){
  vector<int> arr(1000,0);
}
```

2.静态内存：一直都在，程序结束的时候才销毁

```
size_t count_calls(){
    static size_t str = 0;
    return ++ctr;
}
int main(){
    for(int i = 0; i < 10; i++){
        std::cout << count_calls() << endl; 
    }
} // 打印 1-10
```

3.动态内存：需要手工解放，比如new 和delete，在每个程序中作为一个内存池存在，被称为自由空间(free store),或者堆(heap)

一般会产生两个错误：

1.内存泄漏：忘记释放内存

2.产生引用非法内存的指针：在尚有指针引用的情况下就释放了内存

## 全局静态变量，局部静态变量，局部变量，全局变量
一个中文的csdn解释：https://blog.csdn.net/yangfengby0909/article/details/6501147

一个英文的教程网站：https://www.learncpp.com/cpp-tutorial/static-local-variables/

__存储区域__: 全局变量和所有静态是放在内存的静态存储区域， 局部变量存放在内存的栈区

__作用域__：全局变量在整个工程文件有效，静态全局只在定义的文件内有效（二者都是程序开始的时候初始化，在程序结束的时候消失）；局部变量只在函数内有效，局部静态也只在定义的函数内有效，但是程序只分配一次内存（且在第一次执行到定义的地方才初始化），函数返回也不会消失。

__局部静态__ 最基本的使用情况：
```
#include <iostream>

void incrementAndPrint()
{
    int value{ 1 }; // automatic duration by default
    ++value;
    std::cout << value << '\n';
} // value is destroyed here

int main()
{
    incrementAndPrint();   // 2
    incrementAndPrint();   // 2
    incrementAndPrint();    // 2

    return 0;
}
```
```
#include <iostream>

void incrementAndPrint()
{
    static int s_value{ 1 }; // static duration via static keyword.  This initializer is only executed once.
    ++s_value;
    std::cout << s_value << '\n';
} // s_value is not destroyed here, but becomes inaccessible because it goes out of scope

int main()
{
    incrementAndPrint();  // 2
    incrementAndPrint();  // 3
    incrementAndPrint();  // 4

    return 0;
}
```
局部静态使用起来像全局变量一样，但是因为只在函数内生效，所以变得更安全了！

Best practice：记得初始化局部静态变量！

一个关于静态局部常量的tip：如果这个局部常量经常使用且经常被初始化（从数据库取数据，开销很大），那么就可以使用静态的局部常量，因为它现在像全局变量一样了！

Best practice：__永远不要使用__ 局部静态变量，除非它不需要被重置。

解释：你在main函数里仅仅调用了几次同样的函数，这些函数每次都和上次结果不同，这已经很困惑了。 然后假设你又去做别的事情，做完以后重新调用这个函数，却发现数值还在增长。

###  静态成员函数和静态成员变量



## 智能指针
### 普通写法
写法如下，一个可以多个指针指向同一个对象，一个只能一对一
```
#include<memory>
std::shared_ptr<string> p1;    //初始为nullptr
std::shared_ptr<std::list<int>> p2;
std::unique_ptr<string> p1;
```

![image](https://user-images.githubusercontent.com/47411365/127818817-dfb07552-4f43-4abf-8b84-b8efeba378db.png)

一般shared_ptr还有配套的专用函数：
```
std::shared_ptr<int> p3 = std::make_shared<int>(9); // 9
std::shared_ptr<string> p4 = std::make_shared<string>(10,'9'); // 9999999999
std::shared_ptr<int> p5 = std::make_shared<int>(); // 0
图方便可以用auto代替std::shared_ptr<int>

auto p = std::make_shared<int>(9);
auto q(p); // 此时有了两个引用者

auto r = std::make_shared<int>(45);
r = q; // 没有指针引用45，自动销毁

智能指针内部会检测有多少个指针引用（引用计数， reference count），为0的时候自动销毁，销毁调用析构函数
```
指针如果离开函数体（参考最上文的栈内存），它被销毁了内存也就没有了

### shared_ptr 和new
二者不能隐式直接强转：std::shared_ptr<int> a= new int(3);是错误的

如果要转换只能这样构造函数新建：
```
  std::shared_ptr<int> a(new int(3));
```
### 动态数组
其实就是new一个数组的写法
```
int *arr = new int[getsize()];  
delete []arr;
```
### allocator
  allocator 和 new不同的是，可以先分配一个空间但不和对象绑定，更自由。
