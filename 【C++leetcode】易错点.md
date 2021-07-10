## 边界问题
### vector.size() = 0
nums1 或者 nums2 为空的时候，用 nums1.size()-1 或者 nums2.size() - 1 是危险的。因为 vector.size() 返回的是无符号整数。
无符号的 0 - 1 不会得到 -1（因为无符号）。整体来说，应该避免 size() 的结果做减法。一定要做，换成：(int)nums1.size()-1。做强制类型转换
