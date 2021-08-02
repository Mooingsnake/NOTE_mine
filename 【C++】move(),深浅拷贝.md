
可以查看这位的文章： https://zhuanlan.zhihu.com/p/138210501

从语义上解释首先应当明确一点：引用是别名，是 __对象__ 的别名，&xxx不能修改

```
// √
int k = 9;
int& ref = k;

// ×
int k = 9;
int& ref = 0;   // error:C++ expression must be a modifiable lvalue  含义是必须有一个可以修改的左值(lvalue)，&ref 被定死不能修改
```
从底层解释，正如文章最前的链接写的那样：__左值在内存里，作为R1等代表寄存器的地址，值可以被改变，右值是立即数(或者自动变成立即数)，不能被改变__

进一步思考，虽然int& k = 3; 和 int &k = 3;都可以存在，但是后者更被C++ primer推荐使用，这是为什么？

从下面的例子里也可能看出一些规律
```
int k = 1024, &ref = k;   这是正确的
```

传统的C把&当作一个取地址符号，是个纯粹的运算符，现代C++兼容了过去的写法，但是更提倡把&ref当成一个值来看待。

然后还搞出了纯右值（prvalue，pure right value）和将亡值（xvalue，expiring value）
