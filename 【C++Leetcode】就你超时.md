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

// 要有一种意识，迭代的意识，while(low<=high)不直接表明具体遍数，每次mid 和high 和 low的转化是迭代的体现

作者：LeetCode-Solution
链接：https://leetcode-cn.com/problems/binary-search/solution/er-fen-cha-zhao-by-leetcode-solution-f0xw/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
