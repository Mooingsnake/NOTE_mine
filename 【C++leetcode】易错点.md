- [边界](#bound)
- [位运算](#bitcal)
- [快慢指针](#fast_slow_ptr)
- [链表](#linknode)
- [排序](#sort)
- [动态规划](#dp)
- [BFS](#bfs)
- [递归-二叉树-队列](#recursion)
- [栈](#stack)
- [字符串](#string)
- [一些函数](#functions)
- [数据结构模拟](#数据结构模拟)
- [并查集](#union-find-set)




<span id ="bound"></span>
## 边界问题
### vector.size() = 0
nums1 或者 nums2 为空的时候，用 nums1.size()-1 或者 nums2.size() - 1 是危险的。因为 vector.size() 返回的是无符号整数。
无符号的 0 - 1 不会得到 -1（因为无符号）。整体来说，应该避免 size() 的结果做减法。一定要做，换成：(int)nums1.size()-1。做强制类型转换

### int pre;
不赋值的情况下是一个很随便的数，不是0；
### (l+r)/2
int 类型的l或者r会遇到特别大的数的时候会溢出
l/2+r/2的话应该可以

### 访问越界reference binding to misaligned address
reference binding to misaligned address
比如说将行下标对应的整型数作为列的下标索引对数组进行访问
比如说stack为空但是有stack.pop()指令

### 最大值
int maxprice = 1e9;

### 大数注意
(long)((long)+(long))

<span id ="sort"></span>
## 排序
leetcode里面可以使用
```
sort(arr.begin(), arr.end());
arr.back();
```
### 数组中出现超过一半的数字
1.排序，中点必为那个数字

2.摩尔投票法

![image](https://user-images.githubusercontent.com/47411365/132016131-a2df76eb-dca2-4df4-ae3f-cf3bc49a1e51.png)



### 内置排序或者内置规则

内置一个快排
```
class Solution {
public:
    
    minNumber(vector<int>& nums) {          
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
    void quickSort(vector<string>& strs, int l, int r) {         // 手写快排现场
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
        sort(strs.begin(), strs.end(), [](string& x, string& y){ return x + y < y + x; });    // 内置新的排序方法
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

<span id ="bitcal"></span>
## 位运算
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
#### 位运算计算加法
https://www.jianshu.com/p/3f165f976edd
```
 int add(int a, int b) {
        while(b){
            int carry = (unsigned int)(a & b)<<1;   // unsigned 是防止出错， 与运算代表进位 1&1 = 1，因为是进位所以左移
            int sum = a ^ b;				//  相加的本位，异或 
            a = sum ;//  每次不断更新b作为进位，让a做那个sum值，一直到进无可进的时候b会自动归0
            b = carry;
	}
        return a; 
    }
```
如果只是简单的(a&b)那么当a为负数的时候，左移会报错。


<span id ="fast_slow_ptr"></span>
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

双指针还可以用在判断子序列上面，需要注意的是指针一般指到最后一个元素+1，也就是end()的位置

<span id ="linknode"></span>
## 链表
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
### 两个链表合并
我并没有发现当出while(l1&l2)循环之后就只剩下一条线，是可以直接拎过来用的。

并不需要双指针

如果实在觉得要记得一个头结点又要遍历一个工具人结点困难，就用哨兵做头结点前一个，工具人结点 = 哨兵，所有的结点都是.next；

### 反转链表 (迭代)
1.while()里面是curr，还是curr->next?        2.如何正确记录前中后个节点且不会打架且可以迭代？
```
    ListNode* reverseList(ListNode* head) {
        ListNode* prev = nullptr;
        ListNode* curr = head;
        while (curr) {
            ListNode* next = curr->next;
            curr->next = prev;
            prev = curr;
            curr = next;
        }
        return prev;              // 循环结束node是nullptr，所以是pre，这很重要
    }

作者：LeetCode-Solution
链接：https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof/solution/fan-zhuan-lian-biao-by-leetcode-solution-jvs5/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

![image](https://user-images.githubusercontent.com/47411365/131944136-01907dd3-5962-46f8-9db8-c518e5c69cac.png)

移动，先用tmp记录下一个，然后断掉关系再平移挪位，然后建立关系。

递归解法：
```
    public ListNode reverseList(ListNode head) {
        return recur(head, null);    // 调用递归并返回
    }
    private ListNode recur(ListNode cur, ListNode pre) {
/*
本递归方法返回的永远是最后一个节点，无论是哪一层递归
*/
        if (cur == null) return pre; // 终止条件，这里的pre是最后一个节点
        ListNode res = recur(cur.next, cur);  // 递归后继节点，res是最后一个节点
        cur.next = pre;//注意此pre一直在变化
        return res;                  // 返回反转链表的头节点，返回最后一个节点res
    }
``` 
// 循环结束node是nullptr，所以是pre，这很重要，  还有就是return 回最后一个元素的技巧，pre一直在变化的技巧

### 反转链表Ⅱ (头插法，虚拟指针)
![image](https://user-images.githubusercontent.com/47411365/134858032-71eab17d-645e-4936-a84d-7301931e8cd8.png)

```
    ListNode* reverseBetween(ListNode* head, int left, int right) {
        ListNode* dummy_node = new ListNode(0);  // 因为答案头结点不再是遍历的到底的那个节点了，所以得用来记录
        dummy_node->next = head;
        ListNode* g = dummy_node;   // g guard 守卫节点，x-k-k-k-x 第一个x的位置，永远不动
        ListNode* p = dummy_node->next; // p pointer 指针节点 ， x-k-k-k-x 一直在往后捞，但是永远指着同一个节点

        for(int i = 0; i < left-1; i++){ // 遍历到要反转的地方
            g = g->next;
            p = p->next;
        }

        for(int i =0 ; i < right - left;i++){ // 头插法，像扑克一样选一个最小3放最右边，然后一个个插到左边末尾
            ListNode* removed = p->next;  // 不会写的请多画草稿
            p->next = p->next->next;
            removed->next = g->next;
            g->next = removed;
        }

        return dummy_node->next;
    }
```

### 构造函数里不需要清空栈
栈在声明时是空的

### vector.emplace_back()，push_back()
```
        if(n <=1) return n;
        vector<long long>vec(n+1,0); // 注意这里
        vec[0] = 0;
        vec[1] = 1;
        for(int i = 2; i <= n; i++){
            vec.emplace_back(((long long)vec[i - 1] % (long long)(1e9+7)+ vec[i - 2]%(long long)(1e9+7)) %(long long )((1e9+7)));  // emplace_back和push_back一样，都是在最后加
        } 
        return (int)vec[2];
    }
```
这样的写法下，vec[1]以后的都是0，这是因为vector新建的时候就是（n+1,0）emplace_back()在[n+1]增加，另外emplace_back和push_back在c++11之前可能有右值引用的差别，但是C++11及以后差别真的不大

还有个写法
```
vector<int> vec;
vec[1] = 1;
```
将会导致空指针(leetcode 报错信息：reference binding to null pointer of type 'value_type')，因为这里初始化下vec的大小是0；
__总结__：

关于一个容器大小，初始化的时候分配了空间就是[n]下标流

如果没分配空间，就是emplace_back和push_back函数往后填。




### 循环的效率问题
我不知道为什么，我写for(int i = 0; i < nums.size(); i++)的时候效率是4ms

写for(auto x: nums)的时候效率是0ms

<span id ="recursion"></span>
## 递归-迭代-二叉树-队列
最典型的递归是树的遍历（先中后根遍历）

![image](https://user-images.githubusercontent.com/47411365/132642841-cab76f5e-b905-462f-b28b-b9e4c1a5a92a.png)

### 约瑟夫问题 （迭代 递归
```
// 迭代  避免使用栈空间，空间复杂度O（1），只使用常数个变量
    int lastRemaining(int n, int m) {
        int f = 0;
        for (int i = 2; i != n + 1; ++i) {
            f = (m + f) % i;
        }
        return f;
    }

作者：LeetCode-Solution
链接：https://leetcode-cn.com/problems/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-lcof/solution/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-by-lee/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
```
// 递归
private:
    int f(int n, int m) {
        if (n == 1) {
            return 0;
        }
        int x = f(n - 1, m);
        return (m + x) % n;
    }
public:
    int lastRemaining(int n, int m) {
        return f(n, m);
    }

作者：LeetCode-Solution
链接：https://leetcode-cn.com/problems/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-lcof/solution/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-by-lee/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
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

### dfs解二叉搜索树第k大的点
二叉搜索树用右，根，左顺序看是从小到大有序的，别慌.            递归情况下，会出现k==0 和node==nullptr都应该结束返回的情况，应该使用void，在函数内手动记录
```
class Solution {
private:
    int k,res;
    void bfs(TreeNode* node){
        if(node == nullptr) return ;
        bfs(node->right);
        if(k == 0)return;          从k开始，k=0结束，合情合理，比如如果是第一大的点，那就先归零，然后在下次循环游戏结束。
        if(--k == 0){             // 前缀就是先减后赋值 
            res = node->val;
        }
        bfs(node->left);
    }
public:
    int kthLargest(TreeNode* root, int k) {
        this->k = k;
        bfs(root);
        return res;
    }
};

作者：lin-shen-shi-jian-lu-k
链接：https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/solution/zui-jian-jie-yi-dong-de-dai-ma-tu-wen-bi-spxa/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
### 二叉搜索树找最近公共祖先  最简单的while迭代
再次强调，二叉搜索树的性质是 __左<中<右__  所以能直接判断大致在左子树还是在右子树上
```
若 root.val < p.valroot.val<p.val ，则 pp 在 rootroot 右子树 中；
若 root.val > p.valroot.val>p.val ，则 pp 在 rootroot 左子树 中；
若 root.val = p.valroot.val=p.val ，则 pp 和 rootroot 指向 同一节点 。
```
```
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        while(root!=nullptr){
            if(p->val < root->val && q->val < root->val){
                root = root->left;                            // ##神仙迭代法，多学学啊你
            }else if(p->val > root->val && q->val > root->val){
                root = root->right;
            }
            else break;           // 只能再这里break，不然返回void
        }
        return root;
    }
```
### 二叉树找最近公共祖先  bfs递归
![image](https://user-images.githubusercontent.com/47411365/132119075-fdcf0f21-42ed-4c00-b1c7-8e4ea7db0559.png)

__考虑通过递归对二叉树进行先序遍历，当遇到节点 p 或 q 时返回。从底至顶回溯，当节点 p, q 在节点 root的左右子树上时，节点 root 即为最近公共祖先，则向上返回 root 。__

Q：什么 root 一定能返回到最近的祖先节点？

![image](https://user-images.githubusercontent.com/47411365/132119362-fe050f5c-421c-4a59-92d9-09b3e2379da1.png)

A：为其他支路的节点都是空的，所以保留了第一个祖先节点的数字。

Q：何判断在左右子树上？

A：在left和right都不是空的时候。

```
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(root == nullptr || root == p || root == q) return root;           // A 为空，或者相遇，返回相遇的元素（或者空）
        TreeNode *left = lowestCommonAncestor(root->left, p, q);		// 递归
        TreeNode *right = lowestCommonAncestor(root->right, p, q);		// 递归
        if(left == nullptr) return right;					// 
        if(right == nullptr) return left;
        return root;
    }
};

作者：jyd
链接：https://leetcode-cn.com/problems/er-cha-shu-de-zui-jin-gong-gong-zu-xian-lcof/solution/mian-shi-ti-68-ii-er-cha-shu-de-zui-jin-gong-gon-7/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

![image](https://user-images.githubusercontent.com/47411365/132119792-5d7a0108-efdb-426a-a022-f7bd9df19f2f.png)


### 二叉树按层存储
使用队列  ，吐出来一个就塞进去
```
    vector<vector<int>> levelOrder(TreeNode* root) {
        queue<TreeNode*> q;
        vector<vector<int> > ans;
        if(root==NULL){				// 需要先处理边界情况
            return ans;
        }
        q.push(root);				// 需要先埋种子
        while(!q.empty()){                  //一轮又一轮，记得准备好vector<int>作为每层的容器
            vector<int> temp;
            for(int i=q.size();i>0;i--){  // 这里的循环只会将队列q最初的长度值赋给i，所以后面入队的元素不影响, q.size()天然规定了一轮多少个
                TreeNode* node = q.front();
                q.pop();
                temp.push_back(node->val);
                if(node->left!=NULL) q.push(node->left);   // 要记住始终检查空指针
                if(node->right!=NULL) q.push(node->right);
            }
            ans.push_back(temp);//存下当前层的元素
        }

        return ans;
    }

作者：zai-ye-bu-hui
链接：https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-ii-lcof/solution/bfs-an-ceng-cun-chu-by-zai-ye-bu-hui-i4r3/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
### 二叉树按层存储pro (C++双向队列deque)
```
  vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> res;
        deque<TreeNode*> queue;    // 双边队列
        queue.push_back(root);
        if(root==nullptr){   //  只要有push_back(nullptr)
            return res;
        }
        while(!queue.empty()){         
            TreeNode* tmp;
            vector<int> line;
            for(int i = queue.size(); i > 0; i--){
                if(res.size() % 2 == 1){
                    tmp = queue.front();  // 正着取
                    queue.pop_front();
                    if(tmp->right!=nullptr) queue.push_back(tmp->right); 反着放
                    if(tmp->left!=nullptr) queue.push_back(tmp->left);
                }else{
                    tmp = queue.back();
                    queue.pop_back();
                    if(tmp->left!=nullptr) queue.push_front(tmp->left);
                    if(tmp->right!=nullptr) queue.push_front(tmp->right);
                }
                line.push_back(tmp->val);
            }
            res.push_back(line);
        }
        return res;
    }
    
```

### 平衡二叉树  递归 剪枝
当左右子树已经不平衡的时候，就直接return函数，这个return在平衡的时候记录深度，在不平衡的时候返回-1结束游戏，达到剪枝的效果。

return 的是是或否，如何记录左右子树深度？
```
class Solution {
public:
    int recur(TreeNode* root){
        if(root == nullptr)return 0;
        int left = recur(root->left);    
        if(left == -1)return -1;          // 剪枝
        int right = recur(root->right);
        if(right == -1)return -1;	 // 剪枝
        return abs(left-right)>=2?-1:max(left,right)+1;   // 在不平衡的时候表示-1，在平衡的时候记录深度
    }
    bool isBalanced(TreeNode* root) {
        return recur(root)!= -1;
    }
};
https://leetcode-cn.com/problems/ping-heng-er-cha-shu-lcof/submissions/
```
### 重建二叉树 递归
给出前序和后序，比如preorder = [3,9,20,15,7], inorder = [9,3,15,20,7]   ，可以看到3自动把左右子树分成【9 | 15，20，7 】

Q：对于一个链子，二叉树而言，如何做到在递归的时候还能找到自己的头节点？

A：返回它自己，在返回之前设置左右
```
recur(root:0,0,0);

TreeNode* recur(int root, int left, int right){	
	TreeNode* node = new TreeNode(root);
	root->left = recur(left, left+1,right);
	root->right = recur(right, left,right-1);
	return node;
}
```

Q:这是一个什么样的递归？

A：我们可以通过中根inorder判断任意节点的左右子树，我们可以通过明确左右子树来反过来确认先根preorder的下一个根在哪里

换言之，根是可以被求出来的，那么node->left = recur(); 这个函数，当然可以返回一个根的值。而这个值是根据当前子树的左右边界和map了inorder一起决定的


```
public:
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        this->preorder = preorder;
        for(int i = 0; i < inorder.size(); i++)
            dic[inorder[i]] = i;
        return recur(0, 0, inorder.size() - 1);
    }
private:
    vector<int> preorder;
    unordered_map<int, int> dic;
    TreeNode* recur(int root, int left, int right) { 
        if(left > right) return nullptr;                        // 递归终止
        TreeNode* node = new TreeNode(preorder[root]);          // 建立根节点
        int i = dic[preorder[root]];                            // 划分根节点、左子树、右子树
        node->left = recur(root + 1, left, i - 1);              
        node->right = recur(root + i - left + 1, i + 1, right); 
        return node;                                            // 回溯返回根节点
    }

作者：jyd
链接：https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/solution/mian-shi-ti-07-zhong-jian-er-cha-shu-di-gui-fa-qin/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

### 匹配树的子结构 递归
Q：已知AB两树头节点，如何在A中找到那个匹配B的子树？

A：进行递归遍历找到那个B的头结点

Q：已知我有两个相同头结点，如何遍历得出它们一样？

A：进行递归遍历，一直到B树被递归完（即B树的节点最终为null）

Q：如何处理两个递归？

A：两个函数都调用它们自己的递归
#### recur(A, B) 函数：

#### 终止条件：
当节点 BB 为空：说明树 BB 已匹配完成（越过叶子节点），因此返回 true ；

当节点 AA 为空：说明已经越过树 AA 叶子节点，即匹配失败，返回 false；

当节点 AA 和 BB 的值不同：说明匹配失败，返回 falsefalse ；
#### 返回值：
判断 AA 和 BB 的左子节点是否相等，即 recur(A.left, B.left) ；

判断 AA 和 BB 的右子节点是否相等，即 recur(A.right, B.right) ；

#### isSubStructure(A, B) 函数：

#### 特例处理：
当 树 AA 为空 或 树 BB 为空 时，直接返回 false ；
#### 返回值： 
若树 BB 是树 AA 的子结构，则必满足以下三种情况之一，因此用或 || 连接；

以 节点 AA 为根节点的子树 包含树 BB ，对应 recur(A, B)；

树 BB 是 树 AA 左子树 的子结构，对应 isSubStructure(A.left, B)；

树 BB 是 树 AA 右子树 的子结构，对应 isSubStructure(A.right, B)；

```
    public boolean isSubStructure(TreeNode A, TreeNode B) {
        return (A != null && B != null) && (recur(A, B) || isSubStructure(A.left, B) || isSubStructure(A.right, B));
    }
    boolean recur(TreeNode A, TreeNode B) {
        if(B == null) return true;
        if(A == null || A.val != B.val) return false;
        return recur(A.left, B.left) && recur(A.right, B.right);
    }

作者：jyd
链接：https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/solution/mian-shi-ti-26-shu-de-zi-jie-gou-xian-xu-bian-li-p/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
### 二叉搜索树与双向链表

一点牢骚：我真的气死了，又是递归我又不会做，你妈妈的。是的我的确知道二叉搜索树只要中序遍历就是有序的，中序遍历我也会写，不就是两个递归中间夹一个处理自己最后再返回。
但是我 __不知道怎么处理上一个和下一个怎么衔接，不知道初始化写在哪里
```
        if(pre != nullptr) pre->right = cur;   // 不是第一次的情况下，pre往后连
        else head = cur;   // 是第一次的情况下，
        cur->left = pre;  // cur往前连
        pre = cur;        // 下一个，在第一次的语义下也是初始化赋值。

作者：jyd
链接：https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/solution/mian-shi-ti-36-er-cha-sou-suo-shu-yu-shuang-xian-5/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

### 给一个数组判断是否为二叉搜索树后序遍历
#### 1.递归分治（分别解决左子树和右子树）
终止条件： 当 left≥right ，说明此子树节点数量<1，无需判别正确性，因此直接返回 true；

返回值： 所有子树都需正确才可判定正确，因此使用 与逻辑符 && 连接。

p = j： 判断 此树 是否正确。

recur(i, m - 1)： 判断 此树的左子树 是否正确。

recur(m, j - 1)： 判断 此树的右子树 是否正确。

作者：jyd
链接：https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/solution/mian-shi-ti-33-er-cha-sou-suo-shu-de-hou-xu-bian-6/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

#### 2. 辅助单调栈（坑）

```

```
### 二叉树寻找和为target的路径（dfs）

dfs是只有当到底的时候才进行返回，所以只有在root==nullptr的时候才会return。

bfs是不停的return，到底return
```
public:
    vector<vector<int>> ret;
    vector<int> path;

    void dfs(TreeNode* root, int target) {
        if (root == nullptr) {     // 截止条件
            return;
        }
        path.emplace_back(root->val);
        target -= root->val;
        if (root->left == nullptr && root->right == nullptr && target == 0) { // 会返回的，我们把这个if线合并在root==nullptr里面了
            ret.emplace_back(path);               // 既然条件满足，就放进去
        }
        dfs(root->left, target);
        dfs(root->right, target);
        path.pop_back();      // 因为我们要记录多条路径，path 是一个全局变量，每当有一条线完整走完return以后就会从dfs(左)和dfs（右）运行到这一句，（欸↘我从王府井回来辣
    }

    vector<vector<int>> pathSum(TreeNode* root, int target) {
        dfs(root, target);
        return ret;
    }

作者：LeetCode-Solution
链接：https://leetcode-cn.com/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/solution/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-68dg/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
<span id ="stack"></span>
## 栈
### 栈的压入、弹出序列
题目：已知压入序列，判断弹出序列是否合理

解析：已知压入和弹出序列，那么当时情况唯一。所以理论上整个过程可以模拟。 __如何模拟？__  

你就真的拿一个stack出来，然后压入，当栈顶和pop序列一致就弹出，弹到不一致了就接着压。

大循环是压入，小循环是弹出。 你可以通过反复模拟发现的确是大循环套小循环。
```
    bool validateStackSequences(vector<int>& pushed, vector<int>& popped) {
       stack<int> stk;
       int i =0;
       for(auto x:pushed){
           stk.push(x);
           while(!stk.empty()&&stk.top()== popped[i]){
               stk.pop();
               i++;
           }
       }
       return stk.empty();
    }
```

### 递归的调用顺序和 return之间的问题 【原题】反转链表 (双指针)

![image](https://user-images.githubusercontent.com/47411365/131963232-91b60e41-3da2-46aa-b49a-6062cb00838d.png)

你仔细看好，你每次recur(a, b)是不是正常的，很明显第二个不对了，因为cur->next被改了


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
<span id ="dp"></span>
## 动态规划
### 最大子数组之和
学不会的动态规划，如何体现连续，和最大？
1.让动态方程只能连续（可以抛弃/不抛弃之前的串，但永远加上当前的arr[i]当作下次循环的pre）
2.用一个max来维护最大
第二种 分治解法（线段树）

### 从股票问题到最大子数组之和
中间只差了一步转价格为价格变化的数组

### 得到子序列最小操作次数
要用map和数组，用下标的情况显示

首先我们要求的是经过改变后的arr能找出target的匹配的子序列

    arr = {1, 4, 5, 3, 0, 6, 2}
    target = {0, 1, 2, 3, 4, 5, 6}
    
然后找arr里面有序的部分
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

### 求最长上升子序列
https://leetcode-cn.com/problems/longest-increasing-subsequence/solution/zui-chang-shang-sheng-zi-xu-lie-dong-tai-gui-hua-2/
一：dp(n²)

动态规划讲究的是依赖之前的结果。

子序列必定是：dp[i]记录前i个字符内子序列的状态，当nums[7]时，可以遍历前六个nums[j]如果其中nums[4]<nums[7] && dp[4] = 4，那就可以得dp[7] = dp[4]+1

最长嘛，就是前六个都比较完了以后的最大的那个。

每次计算dp[i]必定会循环（0，i）,所以是O(n²)

__我的错误__：
如果最后不计算dp中的最大值，只采用dp[n-1]，是错误的

用例：[1,3,6,7,9,4,10,5,6]，得出的dp[1,2,3,4,5,3,6,4,5]   ,dp明明是前i个子序列最长，为什么会出现前i-1个比前i个还多的情况？

A：因为计算dp的时候因为比较了nums[i]，所以默认包含第i个

二：动态规划 + 二分查找(nlogn)

在上面的动态规划里面，之所以出现了n²，是因为一个循环遍历nums，在循环里面每次都要回去找最大的那个dp[x]

但是我们可以构造一个已知的最长子序列，然后开始二分，我们不需要知道这个子序列分别对应数组1 2 5 7下标，我们只需要知道，是否可以更新，是否可以添加新的
```

    int lengthOfLIS(vector<int>& nums) {
        int n = nums.size();
        vector<int> dp(n,1);
        vector<int> tail(n);
        int res = 0;
        for(auto num:nums){
            int i = 0, j = res;
            while(i<j){
                int mid = (i + j) / 2;
                if(num > tail[mid]){ // 那就说明我是后1/2部分
                    i = mid + 1;
                }else{
                    j = mid;
                }
            }
            tail[i] = num;   // 因为我每次都会更新tail
            if(res == j) res++; // 检索到最大（res）就是胜利（res++）
        }
        return res;
    }

作者：jyd
链接：https://leetcode-cn.com/problems/longest-increasing-subsequence/solution/zui-chang-shang-sheng-zi-xu-lie-dong-tai-gui-hua-2/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
### 最长递增子序列的个数
在上一题朴素的dp情况下，需要计数出现的次数

```
    int findNumberOfLIS(vector<int>& nums) {
        int n = nums.size();
        vector<int> dp(n,1);
        vector<int> count(n,1); // 先全部赋值为1
        for(int i = 0;i <n;i++){
            for(int j = 0;j < i; j++){
                if(nums[j]<nums[i] ){
                    if( dp[j]+1>dp[i]){  // 头一次出现 比如出现dp[..,..,..,3,4,,..,..,..] count[..,..,..,..,..1,1,..,..,..]
                        dp[i] = dp[j]+1;
                        count[i] = count[j];  
                    }         
                    else if(dp[i] ==dp[j]+1){ // 第二次及以后出现dp[..,..,..,4,4..,..,..,] count[..,..,..,..,1,2,..,..,..,]
                        count[i] += count[j];
                    }
                }    
            }
        }
        int res = 0;
       int max = *max_element(dp.begin(),dp.end()); 防止出现nums[2,2,2,2,2]的情况
       for(int i = 0;i < n;i++){
           if(dp[i] == max){
               res += count[i];
           }
       }
       return res;
    }
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
## BFS 
### 矩阵中的路径 时间：O(3^kMN) k是字符串word长度，mn是矩阵长宽,3^k是指每个方块的选择是去掉上一个方块方向的剩下三个选择   空间：O(mn)
```
public:
    bool exist(vector<vector<char>>& board, string word) {
        rows = board.size();
        cols = board[0].size();
        for(int i = 0; i < rows; i++) {
            for(int j = 0; j < cols; j++) {
                if(dfs(board, word, i, j, 0)) return true;
            }
        }
        return false;
    }
private:
    int rows, cols;
    bool dfs(vector<vector<char>>& board, string word, int i, int j, int k) {
        if(i >= rows || i < 0 || j >= cols || j < 0 || board[i][j] != word[k]) return false;  // 去掉越界情况和不相同情况
        if(k == word.size() - 1) return true;							// 最终串完一整串的时候返回true
        board[i][j] = '\0';									//表示访问过了现在是空的
        bool res = dfs(board, word, i + 1, j, k + 1) || dfs(board, word, i - 1, j, k + 1) || 	// 四个方向找路
                      dfs(board, word, i, j + 1, k + 1) || dfs(board, word, i , j - 1, k + 1);
        board[i][j] = word[k];   // board还会在其他递归分支里用到，所以得还原，比如左-右-右-下，左-下-下-右的时候当然不希望第二个格子是空的
        return res;
    }
```
？？实话我没有看明白为什么这里四个dfs但是他认为是3^k，而且也没有看见他用'\0'当判定条件，噢这里可能是包含在 board[i][j] != word[k]里面了

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
<span id ="string"></span>
	
## 字符串
### 类型强制转换
int 转 string

    string str = std::tostring(num);
    
char 转string
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

### 最长公共子序列（二维dp）
```
    int longestCommonSubsequence(string text1, string text2) {
        int n = text1.size();
        int m = text2.size();
        vector<vector<int>> dp (n+1,vector<int>(m+1,0));

        for(int i = 1;i <= n;i++){
            for(int j = 1;j <= m;j++){
                if(text1[i-1] == text2[j-1]){
                    dp[i][j] = dp[i-1][j -1]+1; // 处理一下这里的减一问题
                }else{
                    dp[i][j] = max(dp[i-1][j], dp[i][j-1]); // 在不相等的时候再去取上和左的数据
                }
            }
        }
        return dp[n][m];
    }
```
### 两个字符串的删除操作（二维dp，和上一题类似）
同上
### 匹配字符串
只有( ) * 且* 表示可以替换为( ) 和空字符串

使用栈

容易错的部分：我只记录了* 有多少个，期待在遍历结束后删掉比较多少相互抵消( * ，但是会有 * 出现在(之前，所以需要记录* 的位置，我们可以用vector<int>记录

https://leetcode-cn.com/problems/valid-parenthesis-string/solution/czhan-qiu-jie-zhu-shi-qing-xi-by-hoboom-f6ih/
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

<span id ="functions"></span>
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

对于标号为i的元素而言，且在连续的情况下左边共有i+1种选择（包含什么也不选的情况），同理右边是size-i+1（数字个数+1）

如果在前面选择了奇数个数字，那么在后面，也必须选择奇数个数字，这样加上它自身，才构成奇数长度的数组；（简单的左边2种选法右边三种选法一共2 * 3 =6）

数字前面共有 left 个选择，其中偶数个数字的选择方案有 left_even = (left + 1) / 2 个，奇数个数字的选择方案有 left_odd = left / 2 个；

数字后面共有 right 个选择，其中偶数个数字的选择方案有 right_even = (right + 1) / 2 个，奇数个数字的选择方案有 right_odd = right / 2 个；

## 按权重随机选择  前缀和 二分查找

太长不看版：
```
1.mt19937头文件是<random> 是伪随机数产生器，用于产生高性能的随机数
2.uniform_int_distribution 头文件在<random>中，是一个随机数分布类，参数为生成随机数的类型，构造函数接受两个值表示区间段
3.accumulate 头文件在<numeric>中，求特定范围内所有元素的和。
4.spartial_sum函数的头文件在<numeric>，对(first, last)内的元素逐个求累计和，放在result容器内
5.back_inserter函数头文件<iterator>，用于在末尾插入元素。
6.lower_bound头文件在<algorithm>，用于找出范围内不大于num的第一个元素
```

#### 首先介绍一下一个随机数写法:mt19937
#### 2^19937 – 1 which means mt19937 produces a sequence of 32-bit integers that only repeats itself after 2^19937 – 1 number have been generated.
意思是生成的数字一直到2^19937以后才开始重复
```
// C++ program for demonstrating
// similaritites
#include <ctime>
#include <iostream>
#include <random>
using namespace std;

int main()
{
// Initializing the sequence
// with a seed value
// similar to srand()
mt19937 mt(time(nullptr));
mt19937 my1(random_device{}());    // 只是不同的seed value，每一个seed value只能产生一个相同的随机数结果，要想每次都不同，就需要用机器时间拉之类的东西设置不同的seed value


// Printing a random number
// similar to rand()
cout << mt() << '\n';             // 是一个随机数  ，在当前程序内多次调用产生不同的mt(),但是程序多次启动的时候，如果seed value一直都是一样的，就会一直保持和第一次运行一样的结果，见下图
cout << "the minimum integer it can generate is " << 
           mt.min() << endl;         // 是0
  cout << "mt19937 can generate random numbers upto " << 
           mt.max() << endl;           // 是 2^32 – 1 = 4294967295
return 0;
}
```

![image](https://user-images.githubusercontent.com/47411365/131485339-72e48be3-af7a-430c-8dac-6bb9afb84b4f.png)

#### 然后介绍一下极值写法
```
#include <limits>
...
std::numeric_limits<T>::min();    // 基本写法就是这样
...
```

![image](https://user-images.githubusercontent.com/47411365/131479624-89560b16-7b84-4d0d-b3b1-9a97995a238e.png)

可以看到，关于浮点数的时候，min是指最小最精确的正数，lowest是指一个很大的负数，就是头发丝和马里亚纳海沟的区别

#### 介绍一下离散均匀分布类：
```
std::uniform_int_distribution<> d; // Distribution over 0 to max for type int, inclusive
std::cout << "Range from 0 to "<< std::numeric_limits<std::uniform_int_distribution<> :: result_type>::max()<<std::endl; // Range from 0 to 2147483647

//单纯给你看看uniform_int_distribution这个类的最大值是多少,即2147483647
    std::random_device rd;  // 生产一个seed value
    std::mt19937 gen(rd()); //  搞一个随机数
    std::uniform_int_distribution<> distrib(1, 6);   // 新建，并表示将会在[1,6]之间随机分布
 
    for (int n=0; n<10; ++n)
        //Use `distrib` to transform the random unsigned int generated by gen into an int in [1, 6]
        std::cout << distrib(gen) << ' ';    // 调用
    std::cout << '\n';
```
#### 介绍一下函数 accumulate（）
```
#include <iostream>
#include <vector>
#include <numeric>
#include <string>
#include <functional>
 
int main()
{
    std::vector<int> v{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
 
    int sum = std::accumulate(v.begin(), v.end(), 0);          // 求和,(初始iterator， 终结iterator（尾部向后一格），初始化的sum值)
 
    int product = std::accumulate(v.begin(), v.end(), 1, std::multiplies<int>());   //求积
 
    auto dash_fold = [](std::string a, int b) {
                         return std::move(a) + '-' + std::to_string(b);
                     };                                                           // lambda 设置一个函数能通过不断调用把int连成stringz  这里move(a)不用也行不会出错
 
    std::string s = std::accumulate(std::next(v.begin()), v.end(),
                                    std::to_string(v[0]), // start with first element    // 连成string  ，因为初始化的sum值是1，所以需要std::next(v.begin())指向2
                                    dash_fold);
 
    // Right fold using reverse iterators
    std::string rs = std::accumulate(std::next(v.rbegin()), v.rend(),
                                     std::to_string(v.back()), // start with last element  // 连成反向string
                                     dash_fold);
 
    std::cout << "sum: " << sum << '\n'
              << "product: " << product << '\n'
              << "dash-separated string: " << s << '\n'
              << "dash-separated string (right-folded): " << rs << '\n';
}
```
#### 介绍一下 partial_sum()
和求和唯一不太一样的地方在于，会累加，1 1+2 1+2+3 这样
```
    std::vector<int> v = {2, 2, 2, 2, 2, 2, 2, 2, 2, 2}; // or std::vector<int>v(10, 2);
 
    std::cout << "The first 10 even numbers are: ";
    std::partial_sum(v.begin(), v.end(), 
                     std::ostream_iterator<int>(std::cout, " "));
    std::cout << '\n';
 
    std::partial_sum(v.begin(), v.end(), v.begin(), std::multiplies<int>());
    std::cout << "The first 10 powers of 2 are: ";
    for (auto n : v) {
        std::cout << n << " ";
    }
    std::cout << '\n';
    
    The first 10 even numbers are: 2 4 6 8 10 12 14 16 18 20 
    The first 10 powers of 2 are: 2 4 8 16 32 64 128 256 512 1024
```

#### 介绍一下back_inserter
一个末尾插入的函数
```
    std::vector<int> v1{ 1, 2, 3, 4, 5, 6 };

    *(std::back_inserter(v1)) = 10;
    v1:   1    2       3       4       5       6       10
    
    拷贝v2到v1末尾
        std::vector<int> v1{ 1, 2, 3, 4, 5, 6};
    std::vector<int> v2{ 10, 20, 30, 40, 50, 60};

    std::copy(v2.begin(), v2.end(), std::back_inserter(v1));
```
#### 介绍一下lower_bound（）
返回第一个不小于i的 __迭代器__，如果没有就返回end()
```
struct PriceInfo { double price; };
 
int main()
{
    const std::vector<int> data = { 1, 2, 4, 5, 5, 6 };
    for (int i = 0; i < 8; ++i) {
        // Search for first element x such that i ≤ x
        auto lower = std::lower_bound(data.begin(), data.end(), i);
 
        std::cout << i << " ≤ ";
        lower != data.end()
            ? std::cout << *lower << " at index " << std::distance(data.begin(), lower)   // 迭代器就像指针一样使用就可以了
            : std::cout << "[not found]";
        std::cout << '\n';
    }
 
    std::vector<PriceInfo> prices = { {100.0}, {101.5}, {102.5}, {102.5}, {107.3} };
    for(double to_find: {102.5, 110.2}) {
      auto prc_info = std::lower_bound(prices.begin(), prices.end(), to_find,   //  参数列表 (r.begin(), r.end(), value, [](){})
          [](const PriceInfo& info, double value){
              return info.price < value;
          });
 
      prc_info != prices.end()
          ? std::cout << prc_info->price << " at index " << prc_info - prices.begin()
          : std::cout << to_find << " not found";
      std::cout << '\n';
    }
}
```

#### 答案
```
class Solution {
private:
    mt19937 gen;
    uniform_int_distribution<int> dis;
    vector<int> pre;

public:
    Solution(vector<int>& w): gen(random_device{}()), dis(1, accumulate(w.begin(), w.end(), 0)) {
        partial_sum(w.begin(), w.end(), back_inserter(pre));
    }
    
    int pickIndex() {
        int x = dis(gen);
        return lower_bound(pre.begin(), pre.end(), x) - pre.begin();
    }
};

作者：LeetCode-Solution
链接：https://leetcode-cn.com/problems/random-pick-with-weight/solution/an-quan-zhong-sui-ji-xuan-ze-by-leetcode-h13t/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
## 数据结构模拟
<span id ="数据结构模拟"></span>
### 辅助栈
#### 如果下面有min函数，那么就需要用std::min（）或者::min()
```
class MinStack {
    stack<int> x_stack;
    stack<int> min_stack;
public:
    MinStack() {
        min_stack.push(INT_MAX);
    }
    
    void push(int x) {
        x_stack.push(x);
        min_stack.push(min(min_stack.top(), x));   // 表示在目前为止最小
    }
    
    void pop() {
        x_stack.pop();
        min_stack.pop();   // 一起弹出，因为min_stk有重复，且天然记录在...[index]之前最小，所以没有问题
    }
    
    int top() {
        return x_stack.top();
    }
    
    int min() {
        return min_stack.top();
    }
};

作者：demigodliu
链接：https://leetcode-cn.com/problems/bao-han-minhan-shu-de-zhan-lcof/solution/fu-zhu-zhan-bao-han-minhan-shu-de-zhan-b-fx7t/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
<span id ="union-find-set"></span>
## 并查集
https://www.nowcoder.com/questionTerminal/11ee0516a988421abf40b315a2b28d08?answerType=1&f=discussion
