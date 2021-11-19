### C++内存分配的几个层次
![image](https://user-images.githubusercontent.com/47411365/124862395-e063ac00-dfe7-11eb-93ff-a17f2d73fac5.png)


### 常用的四个层次的用法
![image](https://user-images.githubusercontent.com/47411365/124865324-05a6e900-dfed-11eb-819d-9081e21be9bc.png)
代码如下：
```
void* p1 = malloc(512); // 512bytes;
free(p1);

complex<int>* p2 = new complex<int>; // one object
delete p2;

void* p3 =  ::operator new(512);
::operator delete(p3); // 里面调用了free(p1)；

#ifdef _MSC_VER
    int* p4 = std::allocator<int>().allocate(3);
    std::allocator<int>().deallocate(p4,3);

#indef __GNUC__  //似乎有时候在写法上不一样，暂且不管，因为微软爸爸用的是上面的编译器
    xxxxx;
    xxxx;
```

### new ，::operator new ， malloc之间的区别
__new做的事情:__ 
  1.先分配一块内存用来放object
  2.调用构造函数 
基本的源码结构：
```
Complex* pc = new Complex(1, 2);
// 以上等价于下面部分
Complex* pc;
try {
  void* mem = operator new ( sizeof(Complex) ); // alloc
  pc = static_cast<Complex*>(mem);              // 指针转类型
  pc -> Complex::Complex(1,2);                  // 
}
catch ( std::bad_alloc ) {
  //如果allocation失败则不执行constructor
}

// 关于new里面的operator new函数
void* operator new (size_t size, xxxxxx){
                                              // try to allocate size bytes
 void* p;
 while((p = malloc(size) == 0){
 // buy more memory or return null pointer
 if(_callnewh(size) == 0) break;              // 调用自己写的辅助释放的函数去解决malloc内存分配不过来的问题
    
 }
}
```
attention：new出来的对象和直接声明的对象，不同在这里
```
1）A  a；         // 在栈(stack)上分配空间；
2）A  * a；       // 只是声明，还没有分配空间；
3）A  * a= new A；// 在堆(heap)上分配空间；
// 栈上空间自动回收，堆空间需要程序员手动回收。
```
__delete做的事情__
```
Complex* pc = newComplex(1, 2);
...
delete pc;
   //编译器转化为
 pc -> ~Complex();                         // 先析构
 operator delete(pc)；                     // 然后释放内存,也就是里面free(xx);
```
