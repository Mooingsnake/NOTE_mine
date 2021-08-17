【算法 动态规划】
### 使用范围
动态规划：一个最优解 or 一个方式解 or 求多少种方式 or 求存在性
递归、bfs：输出所有解

### 步骤
1.一般解动态规划都会用一个数组表示状态 arr[x][y]可以通过最后一步，和相关子问题确定什么样的表达式能代表状态共有个硬币，

最后一枚银币：ak （元），则之前有27-ak
		
我们求的不是ak，我们求的是最少用多少枚硬币拼出27，所以f(27) = 最少用多少枚硬币拼出27
		
		f(27) = min{f(27 - 5)+ 1,f(27 - 2) + 1, f(27 - 7) + 1}
递归解法的话
		```
		int f(x){
		if  (xxx)return res;  //边界
		if(xxx)
			return min {f(x -2),res}
		}
		```
		是这样，递归解法它的状态容易重复计算，我们需要拥有**状态转移**方程进行状态转移，和计算顺序改变

2.转移方程
		
		f(27) = min{f(27 - 5)+ 1,f(27 - 2) + 1, f(27 - 7) + 1}

3.初始条件和边界情况
		
	1.如果x -2< 0怎么办？A ：去掉这种情况，设置为正无穷就可以在min比较的时候忽略它
		
	2.怎么停下来？当f（0） = 0的时候，有些题目中需要设置f(x,0),f(0,y) = 0或者1进一步的，任何用转移方程算不出来的都需要被定义
		
	这个例子里面时间复杂度是3 * 27
4.计算顺序
状态转移方程：
```
bool [,] isPalindrome = new ...

isPalindrome [i, j] = isPalidrome[i + 1, j - 1] && (s[i] == s[j] )
```

其中，isPalindrome [i, j] 就是**状态**，i和j代表的是**边界值**，左边指针和右边指针

之后又有：
```
for (int i = n -1; i >= 0; i++) { //这么写是为了对应 i+1，从循环条件开始防止溢出
		for (int j = i; j < n; j++) {
		
		}
}
```

ps：在python里面有个小技巧是用_代替i，一般用在循环里不用i表示arr[i] ，仅仅只是循环工具人的时候
类似foreach

pps:
如何使用contiune减少代码缩进
```
减少缩进行
for()...
	if ...
		做些处理
		做些处理
		做些处理
				
				
for()...
	if. !条件
		continue;
	做些处理；
	做些处理；
	
让if的嵌套关系变成并列关系

for()..
	if ...A
		做些处理
			if ...B
				做些处理；
				做些处理；
				做些处理；
		
for ()..
		if !A
				continue;
		做些处理
		if !B
				continus;
		做些处理；
		做些处理；
		做些处理；
		
```
## 例题

### 第一题
![image](https://user-images.githubusercontent.com/47411365/129543176-1e260179-48f0-4f68-baff-f444a174a9cb.png)

1.求所有不重复字串 等价于 求以arr[i] 结尾的所有不重复子串

2.瓶颈有2：在index：i-1的时候往前推多少是不重复的子串    ；在index ：i的时候往前推多少遇到相同的value

因为第一个瓶颈其实是可以复用的滚动的值，即所有i都依赖i-1的结果，所以可以把dp数组换成这个变量，节省空间

![image](https://user-images.githubusercontent.com/47411365/129546537-89897ba1-634b-45bc-ba60-d87c38e8f68f.png)

preMaxLen ： 瓶颈1

last[0-25] : 字符上次出现的位置

### 第二题

![image](https://user-images.githubusercontent.com/47411365/129547749-0792ccc5-be43-48d4-9787-d7ef1fd5cc32.png)

给每个字符串写摘要，ascii的顺序有就写没有就不写

![image](https://user-images.githubusercontent.com/47411365/129549339-f6d2fc67-a235-48ed-8ac3-29b54d697d28.png)

### 第三题
![image](https://user-images.githubusercontent.com/47411365/129549510-8d26383e-841c-401a-b392-757bb72e1ab6.png)

首先确定概念： __子序列就是每一位要/不要，也就是2^n，也就是01011010__ 

然后累加子序列内所有数字和，加完以后还要取一个input 数取模，求模完以后的最大值

#### 解答

dp[i][j] 表示arr[0..i]中是否能够凑出j

子序列：要么前i-1个元素能自己凑出j   要么i-1个元素能凑出j-arr[i](需要注意的是这里有可能会越界，j - arr[i] >= 0)
