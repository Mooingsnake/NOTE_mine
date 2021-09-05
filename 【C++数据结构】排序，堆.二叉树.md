## 目录
- [排序](#sort)
- [堆](#heap)
- [二叉树](#binarytree)
- [B树](#Btree)
- [并查集](#union-find-set)
- [图](#graph)
<span id ="sort"></span>
## O（n²）排序
### 选择排序
![image](https://user-images.githubusercontent.com/47411365/125230656-d3a6c700-e30b-11eb-9cee-8e0447564338.png)

```
while(遍历n){
  while(白色区域中最小的那个){
  
  }
}
```
#### 稳定性
用数组实现的选择排序是不稳定的，用链表实现的选择排序是稳定的。
不过，一般提到排序算法时，大家往往会默认是数组实现，所以选择排序是不稳定的。

### 插入排序
![image](https://user-images.githubusercontent.com/47411365/125230688-e7522d80-e30b-11eb-911a-5aa1fbf88ed2.png)
```
while(遍历n){
  while(下个白方块移到有序位置){
  }
}
```
attention:虽然每次交换位移，但是其实赋值更快，而且有序情况下非常快。
attention:在极端有序的情况下速度是0（n）
#### 稳定性
由于只需要找到不大于当前数的位置而并不需要交换，因此，直接插入排序是稳定的排序方法。

## nlogn排序
### 归并排序
![image](https://user-images.githubusercontent.com/47411365/125230841-3730f480-e30c-11eb-960b-0e755644cc45.png)
通常使用递归二分的方法来归并:O(logn)，然后用O(n)的速度排序
![image](https://user-images.githubusercontent.com/47411365/125230948-68a9c000-e30c-11eb-98c1-0388793ef509.png)
每次用两个min当作指针，把所有小组遍历完成就是排序结束
attention:需要0（n）大小的空间，每次排序需要使用。
```
while(二分遍历) // 一般这里是递归 ，也可以迭代
  while(遍历n) // 这里一边遍历一边挪两个min指针（有额外空间）
```
#### 稳定性
因为我们在遇到相等的数据的时候必然是按顺序“抄写”到辅助数组上的，所以，归并排序同样是稳定算法。
#### 优化()
1.归并两个group的时候先比较是不是已经有序
2.递归到很小数的时候采用插入排序
#### 简单解释下如何迭代写
```
while(步长从1-2-4-8-----n){
  while(对i-（i+步长）){}迭代
}
```

### 快速排序
warning：接近有序的时候退化成0（n²）
1.基本思路是让第一个元素移到中间，使左边的都小于它，右边的都大于它
![image](https://user-images.githubusercontent.com/47411365/125406350-52266600-e3eb-11eb-881e-28dd345be826.png)
2.具体解释是标记j和i，让其他被遍历的数放在j或者i旁边
如果新数大于j，那么不用动位置，如果新数小于j，那么把j+1和当前新数坐标置换
![image](https://user-images.githubusercontent.com/47411365/125406532-84d05e80-e3eb-11eb-8212-08f8e622f2b1.png)
3.最后把v和j位置交换(注意开闭区间)
```
func(把当前v前/后分区) // 基本就是遍历，二分（近似），插入
{
  while(遍历这个分区){
    func(前后分区){
    
    }
  }
}
```
#### 优化
1.在一个较小数量级的时候可以换成插入排序
2.在近乎有序的情况下v的标记可以每次随机一个数，而不是用第一个元素
补充随机方式
```
template <typename T>
int __partition(T arr[], int l, int r) {
	std::swap(arr[l], arr[rand() % (r-l+1)+l]);
........
}

template<typename T>
void quickSort(T arr[],int n) {
	srand(time(NULL));
	__quickSort(arr, 0,n - 1);
}
```
3.在有非常多重复键值对的时候还是会退化到O（N²）
这是因为大量重复键值容易让分区不平衡二分
![image](https://user-images.githubusercontent.com/47411365/125436431-9ac34c62-8112-4728-b886-d1c70ef75c12.png)
避免了大量=V的元素集中在橙色/紫色

<span id="heap"></span>
## 堆
1.堆是一颗具有特定性质的二叉树，堆的基本要求就是堆中所有结点的值必须 __大于或等于（或小于或等于）其孩子结点的值__，这也称为堆的性质；

2.堆还有另一个性质，就是当 h > 0 时，所有叶子结点都处于第 h 或 h - 1 层（其中 h 为树的高度，完全二叉树），也就是说， __堆应该是一颗完全二叉树__；

3.一般堆用数组存，但是arr[0]为空，从arr[1]开始

__有完全二叉树的性质后可以用一个数组存储堆__

1.每个结点的左孩子为下标i的2倍：left child(i) = i * 2；每个结点的右孩子为下标i的2倍加1：right child(i) = i * 2 + 1

2.每个结点的父亲结点为下标的二分之一：parent(i) = i / 2，注意这里是整数除，2和3除以2都为1，大家可以验证一下；
### 向堆中添加元素和Shift Up
![image](https://user-images.githubusercontent.com/47411365/125750818-7ab68dba-8504-47b8-9d61-e60e1eb8c748.png)
像泡泡一样向上替换
### 向堆中删去最大值（删东西）Shift Down（logn）
![image](https://user-images.githubusercontent.com/47411365/125751321-2cc1c2ea-c886-43dd-937a-d57df4aac21a.png)
向下替换
### 二叉堆
基本上市面上的考试的堆都是二叉堆
### 最大堆
父节点大于等于子节点，左右节点无所谓
### 最小堆
父节点小于等于子节点，左右节点无所谓
### O（nlogn）堆排序
把arr[]使用堆排序，需要用堆的方法插入所有元素，然后用堆的方法一个个吐出来

1.把一个元素吐出来用的是二分，所以是logn

2.把所有元素按顺序吐出来，是n

3.放进去再吐出来，一共2nlogn次

较少使用，一般使用归并或者快排
### heapify 堆化，建堆(O(n))
把一个arr梳理成堆，即遍历所有数进行shiftDown。
需要注意,要新复制一个数组，heap[a] = arr[a+1]

### 优先队列
基于最大堆实现的最大优先队列
实际上优先队列的优先级就是最大堆里面的节点值
当插入好几个一样优先级的节点时自动往后排。

<span id="binarytree"></span>
## 二叉树
### 二叉搜索树
1.可以用链表表示，需要leftNode和rightNode，p（父节点）作为成员
2.左子树所有节点<= 中节点 <= 右子树所有节点
3.中序遍历可以求出一个有序的排列情况
```
function inorderSort(x){
	if(x != null){
		inorderSort();
		print(x);
		inorderSort();
	}
}
```
还可以实现的（均为O（logn））：
1.max()
2.min()
3.search(x)
4.insert()，delete(需要翻转子树)
5.successor(x)，predecessor(x)（后继和前驱）
### 红黑树
1.二叉搜索树不能保证最坏情况的时间效率，红黑树是平衡的二叉树2.平衡二叉树追求绝对平衡，红黑树更宽松更广泛更易用

组成：
- 包含颜色，左，右，父，值
- 根节点和叶子节点黑色
- 如果节点是红，则两个子节点是黑

性质：
节点往下，每一个路径黑色节点个数相同

黑高：
某节点之下（不包含本节点）的一条路径的黑色节点数量。

一个通用哨兵T：
1.有一个T.nil表示红黑树，所有的叶子节点的空子节点
2.有一个T.root表示红黑树的根节点

#### 旋转
![image](https://user-images.githubusercontent.com/47411365/126056049-f5b5243f-3c66-48b3-a811-74fbd0e7669e.png)
```
LEFT-ROTATE(T,x)
  y = x.right
  x.right = y.left
  if(y.left != T.nil)
    y.left.p = x;
  y.p = x.p
  if(x.p == T.nil)
    T.root = y;
  else if(x == x.p.left)
    x.p.right = y;
  else x.p.right = y;
  y.left = x;
  x.p = y;
```
 #### 插入（O（logN））
 插入其实很容易，本身就是一个二叉树所以只需要O(lgn)就能找到合适的位置，规定每次插入都是红色点，难点在修正函数
 ```
 // neel是nullptr一样的玩意
void insert_fixup(root, z, node*& neel) {
	while (z.p.color == RED) {

		if (z.p == z.p.p.left) {
			node* y = z.p.p.right;
			if (y.color == RED) {
				z.p.color = 'b';
				y.color = 'b';
				z.p.p.color = RED;
				z = z.p.p;
			}
			else if (z == z.p.right) {
				z = z.p;
				left_rotate(root, z, neel);

				z.p.color = 'b';
				z.p.p.color = RED;
				right_rotate(root, z.p.p, neel);
			}
			else {
				z.p.color = 'b';
				z.p.p.color = RED;
				right_rotate(root, z.p.p, neel);
			}
		}
		else {                                   //i.e. (z.p == z.p.p.right)
			node* y = z.p.p.left;
			if (y.color == RED) {
				z.p.color = 'b';
				y.color = 'b';
				z.p.p.color = RED;
				z = z.p.p;
			}
			else if (z == z.p.left) {
				z = z.p;
				right_rotate(root, z, neel);

				z.p.color = 'b';
				z.p.p.color = RED;
				left_rotate(root, z.p.p, neel);
			}
			else {
				z.p.color = 'b';
				z.p.p.color = RED;
				left_rotate(root, z.p.p, neel);
			}
		}
	}
	root.color = 'b';
}
 ```
 
 以下图片来自http://alrightchiu.github.io/SecondRound/red-black-tree-insertxin-zeng-zi-liao-yu-fixupxiu-zheng.html
 ![image](https://user-images.githubusercontent.com/47411365/126066544-6e6ad983-8477-4e6c-bd33-362292550914.png)

case1：
如果新增点（current，在算法导论里是z）是红，父亲是红，叔叔是红

那么指针（current / z）移到爷爷，爷爷变红，父&叔叔变黑
不旋转

![image](https://user-images.githubusercontent.com/47411365/126066840-e76a7fae-a4c7-4bfa-b18a-a93438fb39ca.png)

case2：
如果当前点（current/z）红，父亲是红，且z是右节点

那么指针移到它爸，并左转
![image](https://user-images.githubusercontent.com/47411365/126066858-5426533d-3ac1-4517-90e4-4b7933f8da34.png)

case3：
如果当前点（current/z）红，父亲是红，且z是左节点

那么指针移到爷爷，并右转

#### 删除
怒了，在你妈的见
https://www.youtube.com/watch?v=eO3GzpCCUSg
死活看不会，一堆条件，烦死了，人家视频讲前驱后继，算法导论讲子树最小值啊啊啊气死我了

<span id="Btree"></span>
## B树
为磁盘或者其他直接存取的辅助设备设计的平衡搜索树，类似红黑树

不同的是节点可以有多个孩子（数个-数千个）

### 定义：
#### 关于结点
1.一个结点x里面有n个关键字个数，表示为：x.n

2.n个关键字分别有key关键字有序存放，必须保证x.key1 <= x.key2 <= x.key.....<= x.keyn ,表示为：x.key（n）

3.一个结点有一个布尔值表示是否为叶子结点，表示为：x.leaf

4.每个内部节点有n+1个指向它们孩子的指针，表示为：x.c1,x.c2,,,,x.cn+1
#### 关于关键字
1.关键字会被分割范围，K1 <= x.key1 <= K2。。。。x.keyn

2.每个结点的关键字个数x.n都有上界和下界t(必须大于等于2)，被称为最小度数

a. 除了根节点外都有至少 t 个孩子，t-1个关键字

b. 每个内部节点至多有 2t 个孩子，2t - 1 个关键字，当有2t-1个关键字的时候，结点时满的

### 基操
搜索，创建，插入，分裂结点，__删除__
#### 搜索
```
b_search(root, k){
  while(i <= x && k > x.keyi){ // 因为x.n < 2t 所以循环时间O（t）
  if
  else if
  else{
  	DISK-READ(x,ci)
	return b_search(x.ci, k); // 被递归的部分O（h） = O（logt N）
      }
  }
}
```
#### 创建
首先我们需要一个函数ALLOCATE-NODE()来为这个结点分配一个磁盘页的存储空间
```
B-TREE-CREATE(T){
    x = ALLOCATE-NODE();
    x.leaf = TRUE;
    x.n = 0;
    DISK-WRITE(x); // 这个程序是在主存运行的，只改主存，要在磁盘上也覆盖一次，对应还有DISK-READ(x)，把磁盘数据读到主存
    T.root = x
}
```
#### 插入
插入其实不是很难，但是如果插入了一个满的结点就时非法的，我们只能裂开，从中间对半裂开，大概长这样：

【这是左半边】 中间一个点 【这是右半边】

中间的点会上升，加入到上面的结点

一般而言，我们在向下遍历的时候就一边遍历一边裂开了，不会在插入的时候发生上面也满员的尴尬

#### 删除
要保证x里面关键字的个数保持在t及以上

有一套很复杂的情况，不过原则上就是，结点里的key，左右的空就是给孩子插的，所以一个n一个n+1

这种时候我们每次删除要保证：1.孩子有空可插 2.左右两个结点都是t-1的时候就合并 3.保证每个结点关键字个数不少于t-1，如果少了要向爸爸要，爸爸向别的孩子要（稳定的三角传递关系）
![image](https://user-images.githubusercontent.com/47411365/126203417-f1c1d63b-4c3e-4f1b-847a-31a1d49dfa5f.png)
![image](https://user-images.githubusercontent.com/47411365/126203433-b7846ad3-57e3-412d-9826-fc7f8b3dc217.png)


<span id="union-find-set"></span>
## 并查集
你一定对sql很熟，对，就是那个union（并） 和select（查）

MAKE-SET(x):建立一个集合

UNION(x,y):删掉原来的xy，组成新的合并的集合

FIND-SET(x):返回一个指针，是那个包含了x的集合的那个

并查集可以用在求最大连通子图，求最小生成树的 Kruskal 算法和求最近公共祖先（Least Common Ancestors, LCA）等，下面是一个比较好的网址

https://www.cnblogs.com/cyjb/p/UnionFindSets.html

### 操作这些基本函数
假设要计算一个无向图的连通分量（确实会有岛屿状的图，连通的子图是需要被计算的）

就要给每个顶点新建一个集合，然后对于每条表进行合并
```
CONNECTED-COMPONENTS(G)
	for (G.v){
	   MAKE-SET(v);
	}
	for (G.e){
	   if(FIND-SET(e1)!=FIND-SET(e2))
	      UNION(e1,e2);
	}

```
### 链表表示并查集
```
class 并查集{
	Node S;
}
// 这个包含头结点指针和尾结点指针
```
当union的时候可能会出现把大集合合并到小集合里，可以考虑维护一个大小元素，称为加权
### 不相交集合森林
就是用树存储，结点方向向上直到根节点（代表当前集合名字）

合并的时候就把一棵树的根结点连到另外一棵树的根结点去

合并有两种方法：1.比较结点数，少的接到多的上面 2. 路径压缩，变成一层
<span id="graph"></span>
## 图
连通图：所有顶点都能到任意另外一个顶点

树是一幅无环连通图；

数据结构：邻接表结构，即每个顶点多有相邻顶点都保存在该顶点对应的元素

连通分量：一个子图，是最大连通图，简称连通分量

最小生成树：带权重，且所有线权重之和是最小

### DFS 深度优先遍历
![image](https://user-images.githubusercontent.com/47411365/127746578-dd69154f-93b5-41ad-8d3d-6ab197a9ec70.png)

准备两个表：一个是parent，实际不用在dfs而是另外一个打印路径的函数需要使用(dfs负责记录)， 一个是visited，每次前进的时候遍历一边下个能走的结点确保不走重路

因为是这样,递归程序会用栈的方式记录分叉点，dfs是void的函数，做完这些事就可以走了，不需要return
```
	for(遍历临近点){
	    if( !visited ){
	       dfs(xxx);
	    }
	}
```
### BFS 广度优先遍历
![image](https://user-images.githubusercontent.com/47411365/127757845-e8c7a36a-3079-4f8a-8d33-b397f8786180.png)

准备两个表（visited，parent） + 一个队列，把队列当前元素的（！visited）邻接点插入，然后弹出点，先入的先出，是队列

因为每次弹出前都要先遍历周围一圈，所以是广度优先

### KRUSKAL 算法
维护一个set（结点，-1），统一把互相联通的部分用一个结点表示，结点本身用负数表示，每个边都事先排序过

![image](https://user-images.githubusercontent.com/47411365/127764851-79616c99-e655-49ca-b3c0-51dfd51ceb89.png)

每次加上不同的树（如果是相同的树就不加了）


### prim 算法

每探索一个点把所有路径列出，然后选择最小的连上去。不用回退，可以使用优先队列存储那些未包含的边




