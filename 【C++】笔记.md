POWER UP C++ WITH THE STANDARD TEMPLATE LIBRARY PART ONE

网址：https://www.topcoder.com/thrive/articles/Power%20up%20C++%20with%20the%20Standard%20Template%20Library%20Part%20One

## 1容器

对于C语言而言容器只有数组

## 注意事项

嵌套的vector应该这样定义：
 ``` vector < vector < int > >//中间有一个空格```

而不是：
 ``` vector < vector < int >>```因为编译器对于>>有别的定义

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

对STL兼容的反转函数如下：
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




