POWER UP C++ WITH THE STANDARD TEMPLATE LIBRARY PART ONE

网址：https://www.topcoder.com/thrive/articles/Power%20up%20C++%20with%20the%20Standard%20Template%20Library%20Part%20One
 - [1容器](#container)
 - [VECTOR](#VECTOR) 
 - [PAIRS](#PAIRS)
 - [ITERATORS](#iterators)
 - [COMPILING STL PROGRAMS编译错误](#comiling)
 - [DATA MANIPULATION IN VECTOR 数据操纵](#manipulation)
 - [string](#string)
 - [SET](#set)
 - [unordered_map](#unordered_map)
<span id="container"></span>
## 1容器

对于C语言而言容器只有数组

我大约理解的时候把容器看作是一个需要程序员负责的内存空间，比方说vector<int> arr(99,0);然后就成功给这个容器定义了一个99个int大小的空间，每个元素都是0
 
 单纯的定义vector<int> arr 也会让它在内存空间存在，但是需要通过emplace_back()或者push_back()动态分配空间，这两个函数效果一样，不过emplace_back()能把int参数插进vector<vector<int>>里面去，很灵活，但是会给阅读造成困难。
 从源码上（以及非常极端需要效率的时候）讲emplace_back()更快，因为它直接在内存新建，push_back()需要复制
 原文：https://abseil.io/tips/112
<span id="VECTOR"></span>
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

不会，每次push_back的时候不止加一个元素，并且在需要的时候将会重新分配内存空间，再把旧的元素放到新的空间。

```
vector < int > v(20);
v.resize(25);  //会自己调整数组的个数
v.reserve(25)； // 单纯修改一个capacity，表示能用的内存被扩大了，但是里面的内容和element没有初始化或者删掉，所以直接访问可能会越界
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
最后，再强调一遍emplace_back和push_back之间的差别https://haoqchen.site/2020/01/17/emplace_back-vs-push_back/
                                                                                    
（上面网页的复读机）C++11以后，最主要的区别是，emplace_back支持in-place construction，也就是说emplace_back(10, “test”)可以只调用一次constructor，而push_back(MyClass(10, “test”))必须多一次构造和析构
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

## value_type 在容器中的使用
 这个关键词应用在容器上，可以指代容器内单个元素的类型，如vector<int>::value_type AnInt;    vector<int>::value_type就指代了一个int
 
 我们可以用在一个模板函数里面 from:https://stackoverflow.com/questions/44571362/what-is-the-use-of-value-type-in-stl-containers
 ```
 template<typename Container>
typename Container::value_type sum(const Container& cont)
{
    typename Container::value_type total = 0;    // typename表示的是Container::value_type指代一个类，因为B::A有可能是一个变量，这个关键词可以去歧义
    for (const auto& e : cont)  //const告诉别人我们不修改这个遍历结果  ，auto&表示是引用，我们不复制，不过没有const的时候可以改值
        total += e;
    return total;
}
 ```
 
<span id="PAIRS"></span>
## PARIS

说实话这里没怎么看懂
```
 pair < string, pair < int, int > > P;
 string s = P.first; // extract string
 int x = P.second.first; // extract first int
 int y = P.second.second; // extract second int
 ```
 <span id="iterators"></span>
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

 ``` 
c.size() = c.end() - c.begin()
c.empty() = (c.ebign() == c.end());
```

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
另外赋值语句可以这么写
```
vector < int > v;
// …
vector < int > v2(v);  //是深拷贝
vector < int > v3(v.begin(), v.end()); // v3 equals to v2
// …
vector < int > v4(v.begin(), v.begin() + (v.size() / 2));//相当v的前半部分
```
实际上，C++ 11 还实现了浅拷贝赋值
  
![image](https://user-images.githubusercontent.com/47411365/130231694-0a1081f9-7a6d-46c2-b856-58de3c89ec09.png)
 
![image](https://user-images.githubusercontent.com/47411365/130232712-d683d7e9-d2d7-4b0b-9056-1a0f4fbb9300.png)

有意思的是如果移动了那么原来的那个
 
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
                           // 实际上是，迭代器就是指针，完全可以用*it的方式解引用获得原值
vector < int > X(data, data + 5);
int v1 = * max_element(X.begin(), X.end()); // Returns value of max element in vector
int i1 = min_element(X.begin(), X.end())– X.begin(); // Returns index of min element in vector
int v2 = * max_element(data, data + 5); // Returns value of max element in array
int i3 = min_element(data, data + 5)– data; // Returns index of min element in array

```
 * max_element(X.begin(), X.end())找到最小值    【c语言里面，* p是解引用，把指针地址解为值，& a是取地址，把值的存储地址找到】
 
min_element(X.begin(), X.end())– X.begin() 找到最小值的下标，返回的虽然是int，但是是iterator的地址
##### iterator的排序
```
#define all© c.begin(), c.end()
sort(X.begin(), X.end()); // Sort array in ascending order
sort(all(X)); // Sort array in ascending order, use our #define
sort(X.rbegin(), X.rend()); // Sort array in descending order using with reverse iterators
```
 
 __rbegin其实是end的位置  [ ， )__

 #### 关于next和prev
 具体用法就是
```
    std::vector<int> v{ 4, 5, 6 };
 
    auto it = v.begin();
    auto nx = std::next(it, 2);
    std::cout << *it << ' ' << *nx << '\n';
 
    it = v.end();
    nx = std::next(it, -2);
    std::cout << ' ' << *nx << '\n';
 
    std::next(it); // 返回下一个迭代器
```
 http://c.biancheng.net/view/7384.html
 Q： 既然it+12345 就能方便的访问数组任意位置，那为什么要有next和prev
 当我们使用迭代器的时候，不希望迭代器改变值
 <span id="comiling"></span>
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
<span id="manipulation"></span>
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
<span id="string"></span>
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
 <span id="set"></span>
 ## SET（红黑树）
 参考网址：https://www.jianshu.com/p/834cc223bb57
 
 红黑树的数据结构（set，map，multiset，multimap）可以随意增加，删去或者查找（O(log N)时间内），这里的删除添加都是节点A连到节点B，不会有内存拷贝
 
 insert已经有的不会报错，什么都不干

 是二叉树，所以push_back()是不会出现的，也不会有index的
 
 可以用iterator做遍历，就像平时题目里一样，前序后序遍历
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
 <span id="unordered_map"></span>
 ## unordered_map（哈希表hash table）
 ```
 添加： mp[key] = value; 时间：O(1)
 删除   mp.erase(key);   时间：O(1)
 访问： mp[key];         时间：O(1)
 ```
 ### map 和unorder_map的区别（红黑树vs哈希表）
 from：https://www.youtube.com/watch?v=XHDNlgGBnm4
 
![image](https://user-images.githubusercontent.com/47411365/136736343-18e84635-b4f2-486b-8af5-ed8a2a06acbb.png)

 左边的compare函数表示了红黑树的order，并且这里默认是个升序[]（int a,int b）{return a < b;}，你可以改成greater（也就是图上的那个单词
                                                                                         
 右边的hash 函数表示哈希表没有排序                                                                                         
 ![image](https://user-images.githubusercontent.com/47411365/136736773-d7e27101-4a99-4804-a22a-7b87bc9cd50f.png)
                
排序？  map √              unordered_map × 
线程安全？  map ×          unordered_map √                                                                                          

                                                                                          
红黑树直达电梯：https://www.jianshu.com/p/e136ec79235c                                                                                          
                                                                                          
### insert() []  emplace() 三者差别
 from:https://stackoverflow.com/questions/17172080/insert-vs-emplace-vs-operator-in-c-map
 ```
   mp[key] = value // 除了添加还可以修改
 
K t; V u;
std::map<K,V> m;           // std::map<K,V>::value_type is std::pair<const K,V>

m.insert( std::pair<const K,V>(t,u) );      // 1
m.insert( std::map<K,V>::value_type(t,u) ); // 2   1和2是一样的，是std::pair<const K,V> 
m.insert( std::make_pair(t,u) );            // 3   是std::pair<K,V> 注意缺少const，不过不影响编译
 m.insert( std::make_pair<const K,V>(t,u) );  // 4   变成和1，2一样的办法
 
 m.emplace(t,u);      // 5 emplace()中的参数被发送给了当前value_type的构造函数，解决了mp[key]=value只能依赖默认构造函数的情况
``` 

