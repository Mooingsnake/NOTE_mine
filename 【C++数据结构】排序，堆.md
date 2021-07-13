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

## 堆
