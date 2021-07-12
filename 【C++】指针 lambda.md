## 指针和指针函数
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


## lambda
这次是侯捷的

这是一个最简形式：
```
//定义
[]{
  std::cout << "lamabda" << std::endl;
}

//调用
[]{
  std::cout << "lamabda" << std::endl;
}();

//一般使用的情况：
auto l = []{
	std::cout << "lamabda" << std::endl;
};
l();
```
具体结构定义长这样
![image](https://user-images.githubusercontent.com/47411365/125266986-f56a7300-e338-11eb-90db-82ec48cf7b17.png)


[captures]：可以获取的外部的变量 eg:[x, &y]  传值或者传引用

中间的mutable ，throwSpec ，retType都是关键字，可写可不写，写了一个就一定要带前面的()。

#### 具体例子如下：
![image](https://user-images.githubusercontent.com/47411365/125267036-01563500-e339-11eb-98da-4aba15605236.png)


[id] 要在外取一个id的变量(pass by value)，整个方框内相当于右边的一个class，我们命名f()为新函数，那么重载()获得函数，并且现在只是定义，实例化以后为f

![image](https://user-images.githubusercontent.com/47411365/125267055-05825280-e339-11eb-9b51-c0555f083410.png)

第一个输出id:0，是因为；lambda定义前的id是0，没有被id = 42所以这就类似于前一张图里面，id已经变成了f的成员，f独占一片区域，和楼下的id并不相同

lambda的类型：这是一个匿名的函数对象（anoymous function object）

mutable：一个关键词，意思是[..]是可变的，如果没有写mutable，左边的id就不能++

【所以左边和右边的写法一定程度上等效但不完全等效】

我们再接着看：
![image](https://user-images.githubusercontent.com/47411365/125267075-0adf9d00-e339-11eb-8f80-87cad9e6469c.png)


左1 pass by value，类似函数传参被重新复制了一样

左2 pass by reference ，函数传参但是里面和外面是同一个数，外界有变化也会影响它，(int param)就是f(7)中的7，虽然调用了3次其实都是8 8 8

左3 error
![image](https://user-images.githubusercontent.com/47411365/125267099-103ce780-e339-11eb-9c67-8e1d3943e40c.png)


1的内容能代替2的内容

27：12



————————————————————————————

实际上我更喜欢cherno的视频C++
![image](https://user-images.githubusercontent.com/47411365/125267123-15019b80-e339-11eb-9176-7e20dc534139.png)


我们结合了函数指针和lambda

__一种使用方式是，有一天我们能写一个函数，作为api的参数，然后再调用api获得我们自己想要的结果（某种意义上是改动了api）1__

![image](https://user-images.githubusercontent.com/47411365/125267156-1c28a980-e339-11eb-842b-d62db938f6fc.png)



1.[=] ：把所有东西作为值传递进来

2.forEach(xx,yyyy)传递了一个函数，定义函数类型的时候依赖<functional>头文件有一个特殊语法定义

3.类似api函数本土化，我不止要用我自己的函数，我还要用我自己程序里的变量

这里cherno推荐了一个网站cppreference.com，介绍的很形象详细，侯捷的视频里没有capture这个词，但是[capture]很直白的体现了[....]的作用，里面的语言也很简单
![image](https://user-images.githubusercontent.com/47411365/125267187-264aa800-e339-11eb-9f37-db4a291b8ca8.png)

真的好严谨又好懂
mutable:允许函数体修改参数（copy来的那些[capture]），允许调用非常量成员函数
视频结尾cherno给出了一个std库函数的写法
![image](https://user-images.githubusercontent.com/47411365/125267215-2c408900-e339-11eb-9c77-bd2fee907f48.png)
找到第一个大于3的数
  ![image](https://user-images.githubusercontent.com/47411365/125267251-3498c400-e339-11eb-81e4-fb3dcc7e59bb.png)

控制台上将会获得5
一点反思____________________________

即使这样，lambda似乎不能完全替代指针函数，我在尝试用lambda中传递一个<T arr[] >的时候出了问题，查阅cppreference后发现似乎支持了一部分的template但是不包括arr[]。

呃，我不是很清楚parameter pack 是不是包括容器数组这类东西，就算包括我的vs里面也最多支持到C++17

![image](https://user-images.githubusercontent.com/47411365/125267284-3bbfd200-e339-11eb-83a9-823c39157772.png)

这真的太奇怪了，lambda对模板的支持这么差让我觉得它可能不适合拿来写模板函数

https://github.com/lynnboy/CppCoreGuidelines-zh-CN/blob/master/CppCoreGuidelines-zh-CN.md#SS-lambdas
我在cpp core guideline上看也没有说lambda适合用在模板函数上，更准确的应该是本地化的时候比较合适

所以我决定用回指针函数，真香

[丢人bb：其实就是在.h函数里面写了一个测试算法性能的函数，需要把.cpp里面写的算法当作参数传进去，当然算法是模板类型的]，绕了一大圈结果还是指针函数香，太丢人了] 作者：10862638750_bili 
