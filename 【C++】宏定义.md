一个简单的函数宏定义
```
#define LOG(X) std::cout << X << std::endl  
```
X将会被替换成实际代码中的变量

![image](https://user-images.githubusercontent.com/47411365/122428596-a002a680-cfc4-11eb-82f2-e315c77cc97f.png)

首先这样设置，当然PRDEBUG 后面没有=1(另外中间不要加空格)
```
#include <iostream>
#include <string>

#ifdef PR_DEBUG   //这里需要去project-设置-c++-debug那里设置，需要在preprocesser那里注册一个 PR_DEBUG 的 tag，表示在debug状态下替换成这样
#define LOG(X) std::cout << x << std::endl
#else //如果不在debug状态下，替换空，就是删除
#define LOG(X)             
#endif

int main()
{
  LOG("HELLO");
  std::cin.get();
}
```
这是一个版本
优化版本是，上图设置里面改成=1
```
#if PR_DEBUG  == 1
#define LOG(X) std::cout << x << std::endl
#else //如果不在debug状态下，替换空，就是删除
#define LOG(X)             
#endif
```

另外一个优化版本是
```
#if PR_DEBUG  == 1
#define LOG(X) std::cout << x << std::endl
#elif define(PR_RELEASE) //如果不在debug状态下，替换空，就是删除
#define LOG(X)             
#endif
```

如果要在宏定义里面换行
 ```
 #define MAIN int main() \
 {\
  std::cin.get();\
 }
 ```
