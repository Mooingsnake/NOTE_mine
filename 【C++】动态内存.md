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
