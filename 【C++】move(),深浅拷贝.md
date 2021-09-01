### 深浅拷贝

意思是浅拷贝的时候其实共用同一片内存，深拷贝的时候重新开辟一片内存。

大多数时候拷贝用的都是a=b;  默认的复制构造函数是什么都没有的，也不会开辟新内存，所以需要自己写，所有标准库容器都实现了深拷贝的复制构造函数，所以vector<int> a = b;是重新开辟空间遍历赋值的
  
浅拷贝会遇到的问题(实测微软编译器没有，垃圾教科书)：

```
 class ArrayPoint{
     ..... // 没有复制构造函数
  }; 
  int main(){
     ArrayPoint p1(count);
     ArrayPoint p2 = p1;
     return；
  }
```
![image](https://user-images.githubusercontent.com/47411365/127833140-3fdf074e-beef-41b4-a57d-55435dae02ce.png)

深拷贝会出现的问题：
花内存花时间。

### 复制构造函数
```
class ArrayPoint {
    
public:
    vector<int> arr;
    ArrayPoint(int x){   // 构造函数
         vector<int> b(x, 0);
         arr = b;
    }
    ArrayPoint(const ArrayPoint& a) { // 复制构造函数
        vector<int> carr;
        for (auto x : arr) {
            carr.push_back(x);
        }
   }

};
```
### =default
```
  ArrayPoint() = default;
```
有时候需要默认的构造函数，同时也需要自己定义的构造函数可以这样
  
### move()
可以查看这位的文章： https://zhuanlan.zhihu.com/p/138210501
  
好吧，太难懂了，解释下名词，可以看这里的文章：https://www.geeksforgeeks.org/move-constructors-in-c-with-examples/
  
就是说其实有复制构造( copy constructors)还有移动构造（ move constructors）  复制的使用左值引用且的确得复制整个数据，转移使用右值引用且只是单纯新做了一个指针指向原来的内存

#### 左值引用和右值引用
首先是引用的定义回顾：
  
从语义上解释首先应当明确一点：引用是别名，是 __对象__ 的别名，&xxx不能修改
```
// √
int k = 9;
int& ref = k;

// ×
int k = 9;
int& ref = 0;   // error:C++ expression must be a modifiable lvalue  含义是必须有一个可以修改的左值(lvalue)，&ref 被定死不能修改
```
__传统的 C++ 没有区分『移动』和『拷贝』的概念，造成了大量的数据拷贝，浪费时间和空间。__

__要点:左值持久，右值短暂。__
  
  ```
  int i = 0;
  int &&k = i; // 错误，i是左值，右值引用只能绑定右值
  int &&p = i+1; // 正确，右边是一个右值，会被销毁。
  ```
  
右值引用可以接管将要被销毁的对象。

#### 现在介绍一下move()函数
  为了对左值使用右值引用
```
  int &&rref = std::move(i);
```
ATTENTION:此时我们可以销毁i，可以对i赋新值，但是不能使用i。（懵）


从底层解释，正如文章最前的链接写的那样：__左值在内存里，作为R1等代表寄存器的地址，值可以被改变，右值是立即数(或者自动变成立即数)，不能被改变__

进一步思考，虽然int& k = 3; 和 int &k = 3;都可以存在，但是后者更被C++ primer推荐使用，这是为什么？

从下面的例子里也可能看出一些规律
```
int k = 1024, &ref = k;   这是正确的
```

传统的C把&当作一个取地址符号，是个纯粹的运算符，现代C++兼容了过去的写法，但是更提倡把&ref当成一个值来看待。

然后还搞出了纯右值（prvalue，pure right value）和将亡值（xvalue，expiring value）
  
### 简单的右值引用的方式
Tips：遍历的时候可以用右值引用遍历
```
for(auto&& f: flights){
   xxxxxx
}
```

一般stl的容器，都会有一个copy constructor和一个move constructor，比如vector： 
  
![image](https://user-images.githubusercontent.com/47411365/130948974-0528aed2-962f-4687-8224-272205865f37.png)

如果使用了move的话，其实不算拷贝，而是真的移动了，原来的名字下面的空间会清零，如下图所示：

 ![image](https://user-images.githubusercontent.com/47411365/130949124-ecf96262-423d-4d61-9e11-5e4696bce140.png)

很奇怪的是，好像只有stl的右值引用函数会剥夺原来的空间，如下图所示：
  
![image](https://user-images.githubusercontent.com/47411365/131488737-af81d216-948c-495e-be90-4883492075c6.png)

  vector也是，但是如果是int，就不会有这种情况，如下图所示：
 
![image](https://user-images.githubusercontent.com/47411365/131491291-7c578ee2-6fd6-4c57-b7da-7dd2f12dfa6c.png)

 
 这个网址可以解答上面的情况：基本类型类似int是表示拷贝，但是在遇到指针或者stl的时候是移动语义（Move Semantics），另外虽然被移动了为空，但是其实仍然合法（还是可以调用.size()），不过也仅仅只是可以调用，size和capacity都被置0，可见只剩一个指针和一个类型信息：https://zhuanlan.zhihu.com/p/107445960
  
