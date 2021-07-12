### C++教程是b站的那个cherno

无论什么类型的指针，__它都只是个一个片元地址__，这个地址是个整数，32bit 还是64bit和编译器有关和操作系统不直接有关

一个简单例子是64位操作系统使用32位编译器那么它的指针变量就是4字节32bit，不管是int* 还是double*， 都是32bit整数地址，同理使用64位编译器，那么指针变量就是8字节64bit

[指针指向内存，32bit指针能操作的内存最多有多大在这里也可以体现]

void* 说明我们不关心这个指针是什么类型的

void* ptr = 0;

ptr = NULL

ptr = nullptr 三者完全一样，0是一个无效的地址

用vs查看指针把！

![image](https://user-images.githubusercontent.com/47411365/125259405-da483500-e331-11eb-8b95-e08ffb7fe89e.png)

  点击memroy1查看
```
int var = 8;   占4字节Memory： 08 00 00 00 cc cc cc cc(视频教程是小端序啊)

double *ptr = (double*) &var;    memory:08 00 00 00 cc cc cc cc  ，ptr变了么，没变啊，还是指向08的内存



* ptr = 10；当我解引用并且访问它的时候，就是写入内存，这时候标记类型可以辅助修改，写入的时候根据类型决定是几个字节
```
__意思是说，指针永远都是一个地址，就那么一个数，它的值和类型无关，类型真正有用体现在读取和输入的时候，要按照不同type的规则访问和操作内存里的内容。__

比如char*  在访问内存的时候就只修改1字节，

char[] 在访问的时候就会一直往后遍历到/0为止

再比如每次使用指针当作迭代器的时候，a + 5是因为参考了int占用的大小自动 a + 5 * sizeof(int)
```
    int a[5] = { 1,2,3,4,5 };
    //从 a 数组中找到第一个大于 3 的元素
    int *p = upper_bound(a, a + 5, 3);
```

————————————————————————————————

### 指针函数

这是一个比较原始的函数写法，C中就开始这么写了，更贴近C++的表达是用lambda实现的(匿名函数)。

简单来说是将函数赋值给一个变量（assign a function to a variable）

甚至可以把函数A当作参数传进函数B（pass function into other function as parameters）

基本语法：
```
auto function = HelloWorld;
auto function = &HelloWorld;
function();
// 两个表达式是等价的，函数名作为一个值使用时，就会自动转成指针

__函数指针类型怎么写？
bool lengthCompare(const string &, const string &);

// 标准写法：把指针替换为函数名，括号不能少，少了bool *变成返回值
bool (*pf)(const string &, const string &);
pf("A", "B");

// typedef：
typedef bool(*lengthCompareP)(const string &, const string &);
lengthCompareP pf = lengthCompareP;
pf("A", "B");


__为什么这么写？
需要同时记录返回值和参数列表
```
如何使用：

![image](https://user-images.githubusercontent.com/47411365/125260230-b5a08d00-e332-11eb-80c4-14e953a43a0f.png)

我们传入一个指针函数，然后遍历调用这个函数，这是一个简单的例子

当然我们 更推荐使用lambda（C++11的新特性） 读者ps：我是没学会也没觉得好用orz
