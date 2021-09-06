## 从递归到while
### 二分查找
垃圾版本：递归找(left,right),以为自己剪枝了其实还在超时，mid都不知道放哪里，【mid，mid+1】一直进行不下去

优秀版本：while循环
```
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int low = 0, high = nums.size() - 1;
        while(low <= high){
            int mid = (high - low) / 2 + low;  //我是（high+low）/2  ，感觉差不多
            int num = nums[mid];
            if (num == target) {
                return mid;
            } else if (num > target) {
                high = mid - 1;
            } else {
                low = mid + 1;
            }
        }
        return -1;
    }
};

// ###@@###要有一种意识，迭代的意识，while(low<=high)不直接表明具体遍数，每次mid 和high 和 low的转化是迭代的体现

作者：LeetCode-Solution
链接：https://leetcode-cn.com/problems/binary-search/solution/er-fen-cha-zhao-by-leetcode-solution-f0xw/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
### 翻转单词顺序
垃圾的代码：先寻找方法切成vector<string>,然后用accumulate的函数逆序拼回去，结果还不知道处理首尾那些空格。

优秀的代码- 双指针，非常干净的逻辑，for循环达到该点然后直接处理，没有内层结构。
```
     string reverseWords(string s) {
        int n = s.size(), l, r = n - 1;
        string ret;
        while(r >= 0){  
            while(r >= 0 && s[r] == ' ') --r; // 去掉所有的空格，包括后缀和中间的空格
            if(r < 0) break;
            for(l = r; l >= 0 && s[l] != ' '; --l); // 用一个for让l每次来到空格停下来
            ret += (s.substr(l + 1, r - l) + " ");
            r = l;                            // 让r离开当前单词，我发现“ ” ' ' 都可以
         }
        if(!ret.empty()) ret.pop_back();     // 把最后一个空格去掉,if是为了防止空字符串
        return ret;
    }

作者：nobodyyyyy
链接：https://leetcode-cn.com/problems/fan-zhuan-dan-ci-shun-xu-lcof/solution/jian-zhi-offer-58-i-fan-zhuan-dan-ci-shu-vvjl/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```    
