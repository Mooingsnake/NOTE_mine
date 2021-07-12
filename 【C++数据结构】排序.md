## O（n²）排序
### 选择排序
![image](https://user-images.githubusercontent.com/47411365/125230656-d3a6c700-e30b-11eb-9cee-8e0447564338.png)

```
while(遍历n){
  while(白色区域中最小的那个){
  
  }
}
```
### 插入排序
![image](https://user-images.githubusercontent.com/47411365/125230688-e7522d80-e30b-11eb-911a-5aa1fbf88ed2.png)
```
while(遍历n){
  while(下个白方块移到有序位置){
  }
}
```
attention:虽然每次交换位移，但是其实赋值更快，而且有序情况下非常快。

## nlogn排序
### 归并排序
![image](https://user-images.githubusercontent.com/47411365/125230841-3730f480-e30c-11eb-960b-0e755644cc45.png)
通常使用递归二分的方法来归并:O(logn)，然后用O(n)的速度排序
![image](https://user-images.githubusercontent.com/47411365/125230948-68a9c000-e30c-11eb-98c1-0388793ef509.png)
每次用两个min当作指针，把所有小组遍历完成就是排序结束
attention:需要0（n）大小的空间，每次排序需要使用。

###
