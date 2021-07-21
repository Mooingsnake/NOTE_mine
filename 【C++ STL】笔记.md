POWER UP C++ WITH THE STANDARD TEMPLATE LIBRARY PART ONE

网址：https://www.topcoder.com/thrive/articles/Power%20up%20C++%20with%20the%20Standard%20Template%20Library%20Part%20One

##### 本文尚未学习的知识点
```
1.宏定义如何解决迭代器复用
2. r += (it)( * it);
3. something->   vs  (*something)
```

## 1容器

对于C语言而言容器只有数组

我大约理解的时候把容器看作是一个需要程序员负责的内存空间，比方说vector<int> arr(99,0);然后就成功给这个容器定义了一个99个int大小的空间，每个元素都是0
 
 单纯的定义vector<int> arr 也会让它在内存空间存在，但是需要通过emplace_back()或者push_back()动态分配空间，这两个函数效果一样，不过emplace_back()能把int参数插进vector<vector<int>>里面去，很灵活，但是会给阅读造成困难。
 从源码上（以及非常极端需要效率的时候）讲emplace_back()更快，因为它直接在内存新建，push_back()需要复制
 原文：https://abseil.io/tips/112

## 注意事项

嵌套的vector应该这样定义：
 ``` vector < vector < int > >//中间有一个空格```

而不是：
 ``` vector < vector < int >>```因为编译器对于>>有别的定义
 STL是需要#include的，下面出现的所有容器都是STL

## VECTOR

唯一对C有兼容的容器，也就是一个数组

定义方式和遍历方式：
```
  vector < int > v(10);//定义的时候约定了vector的大小
  for (int i = 0; i < 10; i++) {
  v[i] = (i + 1) * (i + 1);
  }
```
``` vector < int > v[10];  ```这里是v[10]的数组,数组元素是v[10]
###### Attention
``` int elements_count = v.size();```这是vector的长度  时间：O（N）,return的值是unsigned属性的（0-2^n）

```bool is_nonempty_ok = !v.empty();``` 判断是否为空

```  v.push_back(i);``` 在vector的末尾增加，并且size()+1

内存分配不会出问题么？

不会，每次push_back的时候不止加一个元素

```
vector < int > v(20);
v.resize(25);
```

另外，其实push_back永远都是额外增加
```
 vector < int > v(20);
 for (int i = 0; i < 20; i++) {
   v[i] = i + 1;
 }
 v.resize(25);
 for (int i = 20; i < 25; i++) {
   v.push_back(i * 2); // Writes to elements with indices [25…30), not [20…25) ! <
 }
```
##### 二位数组
```vector < vector < int > > Matrix;

int N, M;
// …
vector < vector < int > > Matrix(N, vector < int > (M, -1));
```
这里我们新建了一个N * M的二维数组，vector < int > (M, -1)表示大小为M，统一赋值-1

##### Attention
每次调用都会copy传参，要想拒绝传参就不要用下面的代码
``` 
void some_function(vector < int > v) { // Never do it unless you’re sure what you do!
  // …
}
```
应该这样，使用引用
```
 void some_function(const vector < int > & v) { // OK
   // …
 }
```
当然如果const就无法修改v，可以考虑把const去掉


## PARIS

说实话这里没怎么看懂
```
 pair < string, pair < int, int > > P;
 string s = P.first; // extract string
 int x = P.second.first; // extract first int
 int y = P.second.second; // extract second int
 ```
 
 ## ITERATORS
 首先看C语言怎么表示
 ```
 void reverse_array(int * A, int N) {
  int * first = A, * last = A + N - 1;
  while (first < last) {
    swap( * first, * last);
    first++;
    last–;
  }
}
 ```
我们主要看while循环里发生了什么
1. < 比较大小
2. get * first * last 
3. first++
4. last--

iterator替代了上文的指针，并且适用在任何类型下，
##### 迭代器的主要作用就是让代码有复用性，很多类型的容器的遍历都可以用iterator函数来解决
iterator分两种，普通的和随机访问的：“normal iterators” and “random access iterators”
普通的只能一个个累加遍历，比如链表，严格的one by one ，这样时间消耗就一定是0（n）

##### 回到STL
一般而言STL里都有两个迭代器： “begin” and “end.” 
begin:第一个
end:指向的不是最后一个元素，而是**第一个非法元素，或者最后一个元素之后那个**
c.size() = c.end() - c.begin()
c.empty() = (c.ebign() == c.end());

对STL兼容的反转函数如下,体现了模板的优点：
```
template < typename T > void reverse_array_stl_compliant(T * begin, T * end) {
  // We should at first decrement ‘end’
  // But only for non-empty range
  if (begin != end) {
    end–;
    if (begin != end) {
      while (true) {
        swap( * begin, * end);
        begin++;
        if (begin == end) {
          break;
        }
        end–;
        if (begin == end) {
          break;
        }
      }
    }
  }
}
```
另外浅拷贝的赋值语句可以这么写
```
vector < int > v;
// …
vector < int > v2(v);
vector < int > v3(v.begin(), v.end()); // v3 equals to v2
// …
vector < int > v4(v.begin(), v.begin() + (v.size() / 2));//相当v的前半部分
```
另外，把[]数组赋值到vector,如下所示
```
int data[5] = {
  1,
  5,
  2,
  4,
  3
};
vector < int > X(data, data + 5);
```
由此可见data，iterator都是类似指针

 ##### rbegin() rend()
【补充：反向迭代器primer p363】
```
递增递减的含义会倒过来，例如++it的意思是移动到前一个元素，--it则会移动到下一个元素
容器基本上都支持反向迭代器
sort(vec.begin(),vec.end());//正向排序
sort(vec.rbegin(),vec.rend());//逆向排序
```
#####如何定义和使用iterator
```
//遍历vector
for (vector < int > ::iterator it = v.begin(); it != v.end(); it++) {
  * it++; // Increment the value iterator is pointing to
}
```
**！=比<效率更高，empty()比size()!=0效率更高**，只要想一下汇编的时候判断大小和比较等于哪个更容易就知道了、
##### 关于最大最小函数
```
int data[5] = {
  1,
  5,
  2,
  4,
  3
};
vector < int > X(data, data + 5);
int v1 = * max_element(X.begin(), X.end()); // Returns value of max element in vector
int i1 = min_element(X.begin(), X.end())– X.begin(); // Returns index of min element in vector
int v2 = * max_element(data, data + 5); // Returns value of max element in array
int i3 = min_element(data, data + 5)– data; // Returns index of min element in array

```
 * max_element(X.begin(), X.end())找到最小值    【c语言里面，* p是解引用，把指针地址解为值，& a是取地址，把值的存储地址找到】
 
min_element(X.begin(), X.end())– X.begin() 找到最小值的下标，返回的虽然是int，但是是iterator的地址
##### iterator的排序（没咋看懂这个宏定义）
```
#define all© c.begin(), c.end()
sort(X.begin(), X.end()); // Sort array in ascending order
sort(all(X)); // Sort array in ascending order, use our #define
sort(X.rbegin(), X.rend()); // Sort array in descending order using with reverse iterators
```
rbegin其实是end的位置  [ ， )

## COMPILING STL PROGRAMS
STL的报错信息很重要
【没懂宏这里怎么设置的】
遍历一个常量vector 不能使用iterator，需要用const_iterator   
iterator 可读可写，const_iterator可读不写，和常量指针差不多,那么要如何编写遍历？
```
void f(const vector < int > & v) {
  int r = 0;
  // Traverse the vector using const_iterator
  for (vector < int > ::const_iterator it = v.begin(); it != v.end(); it++) {
    r += (it)( * it);
  }
  return r;
}
```

## DATA MANIPULATION IN VECTOR 数据操纵
```
vector < int > v;
// …
v.insert(1, 42); // 把42插在第一个后面，也就是index 0 ，原先[1,last]都往后移
```
如果要多次插入，最好只用一次insert函数
```
vector < int > v;
vector < int > v2;
// …

// Shift all elements from second to last to the appropriate number of elements.
// 把整个V2放在V1第一个数后面
v.insert(1, all(v2));
```
## STRING
区别于vector<char> 主要在函数和内存分配上有差异
 ```
string s = “hello”;
string
s1 = s.substr(0, 3), // “hel”
  s2 = s.substr(1, 3), // “ell”
  s3 = s.substr(0, s.length() - 1), “hell”
s4 = s.substr(1); // “ello”
 ```
 ## SET
 set可以随意增加，删去元素或者检测是否存在（O(log N)时间内）
 insert已经有的不会报错，什么都不干
 是乱序的，所以push_back()是不会出现的，也不会有index的
 除非用iterator做遍历
 ```
 // Calculate the sum of elements in set
set < int > S;
// …
int r = 0;
for (set < int > ::const_iterator it = S.begin(); it != S.end(); it++) {
  r += * it;
}
 ```
 其实这种遍历是比较麻烦的，比如遇到了二维三维嵌套的时候
 它那里说的是用遍历宏，然后就会变成这个样子
 ```
 set < pair < string, pair < int, vector < int > > > > SS;
int total = 0;
tr(SS, it) {
  total += it - > second.first;
}
 ```
 **对不起完全没有看懂**
 
当然拉，我们使用find()查看一个set里面是否存在了这个目标,但是不要用全局的find()，那个是O（N）
 使用set::find()可以将消耗缩到O（log N），当然也return的是一个iterator，要么是找到的那个元素，要么是s.end()
 ```
 set < int > s;
// …
if (s.find(42) != s.end()) {
  // 42 presents in set
} else {
  // 42 not presents in set
}
 ```
 
 
 
 
