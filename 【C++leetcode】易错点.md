

## 边界问题
### vector.size() = 0
nums1 或者 nums2 为空的时候，用 nums1.size()-1 或者 nums2.size() - 1 是危险的。因为 vector.size() 返回的是无符号整数。
无符号的 0 - 1 不会得到 -1（因为无符号）。整体来说，应该避免 size() 的结果做减法。一定要做，换成：(int)nums1.size()-1。做强制类型转换

### int pre;
不赋值的情况下是一个很随便的数，不是0；
### (l+r)/2
int 类型的l或者r会遇到特别大的数的时候会溢出
l/2+r/2的话应该可以

### 最大值
int maxprice = 1e9;

### 排序
leetcode里面可以使用
```
sort(arr.begin(), arr.end());
arr.back();
```

### 位运算
寻找只出现一次的数，时间消耗是0（N）不允许有额外的空间，所以禁哈希表
此时需要位运算
```
    int singleNumber(vector<int>& nums) {
        int ret = 0;
        for (auto e: nums) ret ^= e;
        return ret;
    }

作者：LeetCode-Solution
链接：https://leetcode-cn.com/problems/single-number/solution/zhi-chu-xian-yi-ci-de-shu-zi-by-leetcode-solution/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

### 快慢指针
我只要跑的比我妈快就一定会在某一时刻遇见我妈（成环的时候），步长为1和2的时候最小公倍数就是相遇的契机。
适合用在O（1）空间消耗的情况下

### 双指针
双指针可以给两个链表使用，a指针遍历完去b链，反之同样。
适合用在O（1）空间消耗的情况下

### 链表空指针
首先是head == nullptr,head -> next == nullptr
然后循环的写法也是以判断为先
```
        while (slow != fast) {
            if (fast == nullptr || fast->next == nullptr) {
                return false;
            }
            slow = slow->next;
            fast = fast->next->next;
        }

作者：LeetCode-Solution
链接：https://leetcode-cn.com/problems/linked-list-cycle/solution/huan-xing-lian-biao-by-leetcode-solution/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

### 最大子数组之和
学不会的动态规划，如何体现连续，和最大？
1.让动态方程只能连续（可以抛弃/不抛弃之前的串，但永远加上当前的arr[i]当作下次循环的pre）
2.用一个max来维护最大
第二种 分治解法（线段树）

### 从股票问题到最大子数组之和
中间只差了一步转价格为价格变化的数组

### 访问越界reference binding to misaligned address
reference binding to misaligned address
比如说将行下标对应的整型数作为列的下标索引对数组进行访问
比如说stack为空但是有stack.pop()指令

### 构造函数里不需要清空栈
栈在声明时是空的

### vector.emplace_back()，push_back()
```
        if(n <=1) return n;
        vector<long long>vec(n+1,0);
        vec[0] = 0;
        vec[1] = 1;
        for(int i = 2; i <= n; i++){
            vec.emplace_back(((long long)vec[i - 1] % (long long)(1e9+7)+ vec[i - 2]%(long long)(1e9+7)) %(long long )((1e9+7)));
        } 
        return (int)vec[2];
    }
```
这样的写法下，vec[1]以后的都是0，这是因为vector新建的时候就是（n+1,0）emplace_back()在后面增加

还有个写法
```
vector<int> vec;
vec[1] = 1;
```
将会导致空指针(leetcode 报错信息：reference binding to null pointer of type 'value_type')，因为这里初始化下vec的大小是0；
__总结__：

关于一个容器大小，初始化的时候分配了空间就是[n]下标流

如果没分配空间，就是emplace_back函数

### 大数注意
(long)((long)+(long))
