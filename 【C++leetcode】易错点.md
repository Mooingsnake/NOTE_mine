

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
#### 这是异或
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

更效率的

```
#### 这是简单进位
```
 int hammingWeight(uint32_t n) {
        int ret = 0;
        for (int i = 0; i < 32; i++) {
            if (n & (1 << i)) {
                ret++;
            }
        }
        return ret;
    }

作者：LeetCode-Solution
链接：https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/solution/er-jin-zhi-zhong-1de-ge-shu-by-leetcode-50bb1/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
更效率的方式：
  int count = 0;
        for(int i = 0; i< 32;i++){
             count += ((n >> i) & 1);
        }
        return count;
    }
```

### 快慢指针
我只要跑的比我妈快就一定会在某一时刻遇见我妈（成环的时候），步长为1和2的时候最小公倍数就是相遇的契机。
适合用在O（1）空间消耗的情况下

什么时候决定结束？
链表结束的时候

实际上可以如果是一个环形的链表或者数组，那么可以看作是图的形式——

我们需要一个额外空间记录visited，但是一旦记录了以后，成环就会变得简单。

### 双指针
双指针可以给两个链表使用，a指针遍历完去b链，反之同样。
适合用在O（1）空间消耗的情况下
---------
双指针还可以用在判断子序列上面，
需要注意的是指针一般指到最后一个元素+1，也就是end()的位置

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

我太难了难成傻逼我直接unravel，这段小小的代码在来姨妈的时候折磨了我整整一天一夜，凸(艹皿艹 )
```
 int n = target.size();
        unordered_map<int, int> pos;
        for (int i = 0; i < n; ++i) {
            pos[target[i]] = i;
        }
        vector<int> d;
        for (int val : arr) {
            if (pos.count(val)) { // 如果有含有target的元素
                int idx = pos[val];  // 记录那个元素在target中的值
                auto it = lower_bound(d.begin(), d.end(), idx); // 在新数组里找更小的，这里函数是二分
                if (it != d.end()) { // 如果找到了
                    *it = idx; // 就记录那个值
                } else {
                    d.push_back(idx); // 如果没找到，就加上去
                }
            }
        }
        return n - d.size();
    }

作者：LeetCode-Solution
链接：https://leetcode-cn.com/problems/minimum-operations-to-make-a-subsequence/solution/de-dao-zi-xu-lie-de-zui-shao-cao-zuo-ci-hefgl/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

### 二叉树寻路
1.既然这个二叉树是Z形状，那就可以通过max-当前+ min 的方式转换出来
2.二叉树正常情况下就是2^i ~2^(i+1) -1 那么用log就能得出有几层
3.父节点就是子节点除以2

### 矩阵中战斗最弱的一行，二分查找，堆排序
因为题目里面是1在前，0在后，所以可以用二分查找寻找最小。

堆排序的查找时间是O（n），很合适。

二分查找需要手写代码，堆排序可以使用现成代码：

### 辗转相除法求公约数
```
int gcd(int a,int b){
    int tmp;
    while(b){
        tmp = b; b = a % b ; a = tmp;
    }
    return a;
}
```

### acm 模式
循环输出用例直到结尾：
```
	while (cin >> n >> a) {
		int i = 0;
		int bn = 0;
		while (i < n) {
			cin >> bn;
			if (a > bn)a += bn;
			else {
				a += bcd(a, bn);
			}
			i++;
		}
		cout << a << endl;
	}
```

### 字符串倒转
看清楚，迭代器可以直接加减，reverse是常用函数，最重要的是min(i+k,n)这样子就可以精准狙击各种第一遍和最后一遍k不足的情况。
```
    string reverseStr(string s, int k) {
        for(int i = 0; i < s.size(); i += k *2){
            int n = s.size() ;
            reverse(s.begin()+i,s.begin() + min(i+k,n));
        }
        return s;
    }
```


### 压缩字符串
https://leetcode-cn.com/problems/string-compression/solution/ya-suo-zi-fu-chuan-by-leetcode-solution-kbuc/

双指针，记录写入的坐标和相同的字符的串的最后一个位置，然后取每个位数的时候记得反向
```
 int compress(vector<char>& chars) {
        int n = chars.size();
        int write = 0, left = 0;
        for (int read = 0; read < n; read++) {
            if (read == n - 1 || chars[read] != chars[read + 1]) {  // 首先确定是否会越界，然后挑中数个一样字符的最后一个
                chars[write++] = chars[read];   // 下一个就写上这一串字符的字符类型
                int num = read - left + 1;  // 然后我们要知道一共有多少个这样的字符，用双指针记录
                if (num > 1) {   	 	
                    int anchor = write;       // 新建一个锚用来遍历管理每一位
                    while (num > 0) {
                        chars[write++] = num % 10 + '0';
                        num /= 10;
                    }
                    reverse(&chars[anchor], &chars[write]);   // 但是需要反序，这里reverse(,)只需要两个指针就可以了
                }
                left = read + 1;   	//不要忘记给left+1,这里left和read是一个维度，write是另外一个关系
            }
        }
        return write;
    }

```

### K 站中转内最便宜的航班，DP,BFS,图

__DP__

Q:这道题出现了一个图状的数据结构，对于dp而言一般都是线性的遍历方式，怎么办？

A:直接遍历它的信息矩阵，然后一遍遍更新即可

Q:如何从图的结构寻找转移方程？

A:![image](https://user-images.githubusercontent.com/47411365/130937146-f79d4493-393a-4836-8de2-f6722be2b207.png)

![image](https://user-images.githubusercontent.com/47411365/130937254-4122fa37-d22b-4ec1-9afa-eb10a1dce965.png)

简单地再解释下，还是一条路径就一定会有A->B->C 那么BC段就是基于AB的转移状态    之后只要寻找寻找结果的最小值


Tips：遍历的时候可以用右值引用遍历
```
for(auto&& f: flights){
   xxxxxx
}
```

__BFS__


## 所有奇数长度数组的和 前缀和 概率论
方法一：前缀和
```
class Solution {
public:
    int sumOddLengthSubarrays(vector<int>& arr) {

        vector<int> presum = {0};
        for(int e: arr) presum.push_back(presum.back() + e);   // 前缀和    O（n）

        int res = 0;
        for(int i = 0; i < arr.size(); i ++)
            for(int sz = 1; i + sz - 1 < arr.size(); sz += 2)
                res += presum[i + sz] - presum[i];            // 用差一个个表示所有O（n2）

        return res;
    }
};

作者：liuyubobobo
链接：https://leetcode-cn.com/problems/sum-of-all-odd-length-subarrays/solution/cong-on3-dao-on-de-jie-fa-by-liuyubobobo/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
#### 方法二：概率论论证每个数字将会被加多少遍，如果在前面选择了偶数个数字，那么在后面，也必须选择偶数个数字，这样加上它自身，才构成奇数长度的数组；

如果在前面选择了奇数个数字，那么在后面，也必须选择奇数个数字，这样加上它自身，才构成奇数长度的数组；

数字前面共有 left 个选择，其中偶数个数字的选择方案有 left_even = (left + 1) / 2 个，奇数个数字的选择方案有 left_odd = left / 2 个；

数字后面共有 right 个选择，其中偶数个数字的选择方案有 right_even = (right + 1) / 2 个，奇数个数字的选择方案有 right_odd = right / 2 个；

