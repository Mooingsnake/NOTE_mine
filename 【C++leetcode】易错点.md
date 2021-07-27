

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

### 类型强制转换
int 转 string

    string str = std::tostring(num);
    
char 转string
### 内置排序或者内置规则

内置一个快排
```
class Solution {
public:
    string minNumber(vector<int>& nums) {
        vector<string> strs;
        for(int i = 0; i < nums.size(); i++)
            strs.push_back(to_string(nums[i]));
        quickSort(strs, 0, strs.size() - 1);
        string res;
        for(string s : strs)
            res.append(s);
        return res;
    }
private:
    void quickSort(vector<string>& strs, int l, int r) {
        if(l >= r) return;
        int i = l, j = r;
        while(i < j) {
            while(strs[j] + strs[l] >= strs[l] + strs[j] && i < j) j--;
            while(strs[i] + strs[l] <= strs[l] + strs[i] && i < j) i++;
            swap(strs[i], strs[j]);
        }
        swap(strs[i], strs[l]);
        quickSort(strs, l, i - 1);
        quickSort(strs, i + 1, r);
    }
};

作者：jyd
链接：https://leetcode-cn.com/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/solution/mian-shi-ti-45-ba-shu-zu-pai-cheng-zui-xiao-de-s-4/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
内置一个排序规则,这里很明显用的是lambda

一个点是sort这个函数里面用的排序方法是string > string，而不是compare，所以是true or false
```
class Solution {
public:
    string minNumber(vector<int>& nums) {
        vector<string> strs;
        string res;
        for(int i = 0; i < nums.size(); i++)
            strs.push_back(to_string(nums[i]));
        sort(strs.begin(), strs.end(), [](string& x, string& y){ return x + y < y + x; });
        for(int i = 0; i < strs.size(); i++)
            res.append(strs[i]);
        return res;
    }
};

作者：jyd
链接：https://leetcode-cn.com/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/solution/mian-shi-ti-45-ba-shu-zu-pai-cheng-zui-xiao-de-s-4/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
### 循环的效率问题
我不知道为什么，我写for(int i = 0; i < nums.size(); i++)的时候效率是4ms

写for(auto x: nums)的时候效率是0ms

### 两个链表合并
我并没有发现当出while(l1&l2)循环之后就只剩下一条线，是可以直接拎过来用的。

并不需要双指针

如果实在觉得要记得一个头结点又要遍历一个工具人结点困难，就用哨兵做头结点前一个，工具人结点 = 哨兵，所有的结点都是.next；

### 递归的时候的内存问题
```
 TreeNode* mirrorTree(TreeNode* root) {
        if(root == nullptr)return nullptr;
        TreeNode* tmp = root->left;
        root->left = mirrorTree(root->right);
         root->right = mirrorTree(tmp); <----------注意这里，如果tmp是root->left会报一个heap-use-after-free的错误，而这种错误只出现在free(arr)后调用arr[1]上
         return root;
    }
```
至今仍未知道它在递归的时候是怎么被释放的。

### for_each的leetcode骚操作
用于C++17：
```
for (const auto& [key, value] : myMap) {
    std::cout << key << " has value " << value << std::endl;
}
```
用于C++11
```
for (const auto& kv : myMap) {
    std::cout << kv.first << " has value " << kv.second << std::endl;
}
```
### 得到子序列最小操作次数
要用map和数组，用下标的情况显示

首先我们要求的是经过改变后的arr能找出target的匹配的子序列

    arr = {1, 4, 5, 3, 0, 6, 2}
    target = {0, 1, 2, 3, 4, 5, 6}
    
然后找arr里面有序的部分
### 子问题：求最大递增子序列
一：dp(n²)

实际上对于动态规划，最后一步f(n) = f(n-1) + 1是容易得出的

主要关键是每一次的f(n)它一定要包括nums[n]

然后在所有dp[]之中选择最大值

另外,下面的写法很不错
```*max_element(dp.begin(), dp.end());```
二：二分法和贪心(nlogn)

在上面的动态规划里面，之所以出现了n²，是因为一个循环遍历nums，在循环里面每次都要回去找最大的那个dp[x]然后再加上本身+1

二分法则是需要把dp重新变成n下最大递增子序列（没有必须包括nums[n]的限制），这样dp就是有序的，有序才能二分

![image](https://user-images.githubusercontent.com/47411365/127093482-d4eb650a-3579-46f3-b192-5d8c721df863.png)

