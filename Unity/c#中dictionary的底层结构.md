# C# Dictionary 底层结构与遍历性能解析

---

## 一、底层数据结构

C# 中的 `Dictionary<TKey, TValue>` 是基于**哈希表（Hash Table）**实现的键值对集合，底层采用**拉链法（Chaining）的变体（Entry 数组 + 哈希桶）**来解决哈希冲突。它的核心数据存储在一块连续的内存数组中，而不是使用传统的链表指针。

### 核心组成部分

* **`int[] buckets`（哈希桶数组）：** 存放对应哈希值在 `entries` 数组中的**起始索引**。初始默认值为 `-1`，表示桶为空。
* **`Entry[] entries`（数据数组）：** 连续存放真正的键值对数据及链表指针。每个 `Entry` 结构体包含 4 个字段：
  * `hashCode`：键的哈希码（高位置位，避免负数）。
  * `next`：冲突链表中下一个节点在 `entries` 数组中的**索引**（用数组下标模拟指针，内存紧凑）。
  * `key`：元素的键。
  * `value`：元素的值。

---

## 二、工作原理与核心机制

### 1. 插入数据 (Add / Write)
* **计算桶索引：** 调用 `Key.GetHashCode()` 算出哈希码，通过取模运算计算索引：`bucketIndex = hashCode % buckets.Length`。
* **写入数据：** 将键值对存入 `entries` 数组的下一个可用位置 `count`（即 `entries[count]`）。
* **挂载冲突链表（头插法）：** 若 `buckets[bucketIndex]` 已有值（发生哈希冲突），把原有的索引赋值给新条目的 `next` 字段（`entries[count].next = buckets[bucketIndex]`），再将 `buckets[bucketIndex]` 更新为当前新条目的索引 `count`。

### 2. 查找数据 (Lookup / Read)
* 计算桶索引 `bucketIndex`。
* 获取 `i = buckets[bucketIndex]`，若 `i >= 0` 则开始遍历链表：
  * 比较 `entries[i].hashCode` 以及 `Key` 的实际值（通过 `EqualityComparer.Equals`）。
  * 匹配成功则返回 `entries[i].value`；若不匹配，沿着 `i = entries[i].next` 继续查找，直到 `i == -1`。

### 3. 动态扩容与 Rehash
* 当 `entries` 数组达到容量上限时触发扩容。
* 新容量会扩展为**大于等于原容量 2 倍的下一个素数**（使用素数作为桶大小可大幅减少哈希碰撞）。
* 创建全新的 `buckets` 和 `entries` 数组，重新计算旧数据对应的桶索引（Rehash）并填入新桶。

---

## 三、时空复杂度与性能特点

* **时间复杂度：** 查找、插入、删除的平均时间复杂度均为 **$O(1)$**；极少出现的严重冲突或扩容阶段为 **$O(n)$**。
* **缓存友好（Cache Friendly）：** 传统拉链法易产生内存碎片并降低 CPU 缓存命中率，而 C# 使用连续的 `Entry[]` 数组存储，内存紧凑，同时避免了频繁 GC。
* **非线程安全：** `Dictionary<TKey, TValue>` 不支持并发写入。多线程场景下请使用 `ConcurrentDictionary<TKey, TValue>`。

---

## 四、`foreach` 遍历与 GC / 装箱问题

在现代 .NET（.NET Core 及 .NET Framework 3.5+）中，对 `Dictionary<TKey, TValue>` 使用 `foreach` 遍历**不会产生装箱（Boxing），也不会触发额外的 GC 垃圾回收**。

### 无装箱/GC 堆分配的原因

1. **迭代器为结构体（Struct Allocator-Free）：** 
   `Dictionary<TKey, TValue>.GetEnumerator()` 返回的是结构体 `Dictionary<TKey, TValue>.Enumerator` 而非接口 `IEnumerator<T>`。`foreach` 编译后直接在栈上创建该结构体，零堆内存分配。
2. **强类型元素也是结构体：** 
   遍历过程中获取到的元素类型是 `KeyValuePair<TKey, TValue>`（结构体）。只要 `TKey` 与 `TValue` 也是值类型，整个遍历流程均在栈上完成。

### 会引发装箱或 GC 的特例场景

* **隐式/显式转换为接口遍历：**
  ```csharp
  IEnumerable<KeyValuePair<TKey, TValue>> map = dict;
  foreach (var kvp in map) { } // 迭代器结构体会被装箱为 IEnumerable 接口对象
