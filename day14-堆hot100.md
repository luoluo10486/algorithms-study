# Day 14 — 堆 Hot100

## 题目：数组中的第 K 个最大元素（Kth Largest Element in an Array）

**LeetCode 215 | 堆 Hot100 | 难度：🟡 中等**

### 题目描述

给定整数数组 `nums` 和整数 `k`，请返回数组中第 **k** 个最大的元素。

请注意，你需要找的是数组排序后的第 `k` 个最大的元素，而不是第 `k` 个不同的元素。

你必须设计并实现时间复杂度为 **O(n)** 的算法解决此问题。

### 示例

```
输入：nums = [3,2,1,5,6,4], k = 2
输出：5

输入：nums = [3,2,3,1,2,4,5,5,6], k = 4
输出：4
```

---

## 解法一：快速选择（QuickSelect）—— 最优解

### 思路

快速选择是**快速排序的变体**——利用 partition 分区，根据枢轴的位置决定在左区间还是右区间继续查找，不需要全排序。

```
核心思想：

1. 在 [left, right] 范围内随机选一个枢轴 pivot
2. partition 后，pivot 落在下标 pivotIdx
3. 如果 pivotIdx == targetIndex（n-k），直接返回
4. 如果 pivotIdx > targetIndex → 在左区间 [left, pivotIdx-1] 继续找
5. 如果 pivotIdx < targetIndex → 在右区间 [pivotIdx+1, right] 继续找
```

**为什么随机选枢轴？** 避免最坏情况 O(n²)——如果每次都选到最小/最大值，partition 退化为每次只排除一个元素。随机化后，期望时间复杂度为 **O(n)**。

**为什么 targetIndex = n - k？**

```
第 k 个最大元素 = 升序排列后的第 n-k 个元素（下标从 0 开始）
如 [3,2,1,5,6,4], k=2:
  排序后 [1,2,3,4,5,6]
  第 2 大 = 5 = 下标 n-k = 6-2 = 4 → nums[4] = 5 ✅
```

### 思考方式图解

```
nums = [3,2,1,5,6,4], k=2, targetIndex = 6-2 = 4

初始：left=0, right=5

第 1 轮 partition（随机选枢轴，假设选中 4）：
  [3, 2, 1, 4, 5, 6]  ← partition 后 4 落在下标 3
  pivotIdx=3 < targetIndex=4 → 去右区间 [4, 5] 找

left=4, right=5
第 2 轮 partition（随机选枢轴，选中 5）：
  [3, 2, 1, 4, 5, 6]  ← 5 在位置 4
  pivotIdx=4 == targetIndex=4 → 返回 5 ✅
```

### 代码实现

```java
class Solution {
    private static final Random rand = new Random();

    public int findKthLargest(int[] nums, int k) {
        int n = nums.length;
        int targetIndex = n - k;   // 第 k 大 = 升序第 n-k 个
        int left = 0;
        int right = n - 1;

        while (true) {
            int pivotIdx = partition(nums, left, right);
            if (pivotIdx == targetIndex) return nums[pivotIdx];
            if (pivotIdx > targetIndex) {
                right = pivotIdx - 1;   // 去左区间
            } else {
                left = pivotIdx + 1;    // 去右区间
            }
        }
    }

    // 双指针分区：返回枢轴的最终下标
    private int partition(int[] nums, int left, int right) {
        // 随机选一个枢轴，交换到最左边
        int i = left + rand.nextInt(right - left + 1);
        int pivot = nums[i];
        swap(nums, i, left);

        i = left + 1;   // 左指针
        int j = right;  // 右指针

        while (true) {
            while (i <= j && nums[i] < pivot) i++;  // 跳过小于 pivot 的
            while (i <= j && nums[j] > pivot) j--;  // 跳过大于 pivot 的
            if (i >= j) break;
            swap(nums, i, j);  // 交换左右违规元素
            i++;
            j--;
        }

        swap(nums, left, j);  // 把枢轴放到正确位置
        return j;
    }

    private void swap(int[] nums, int a, int b) {
        int tmp = nums[a];
        nums[a] = nums[b];
        nums[b] = tmp;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **期望 O(n)**，最坏 O(n²) — 随机化后最坏概率极低 |
| 🧠 空间复杂度 | **O(1)** — 原地交换，不计递归栈（这里是迭代实现） |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 快速选择（QuickSelect） |
| 算法类型 | 分治、快速排序变体 |
| 核心技巧 | **利用 partition 分区，根据枢轴位置判断在左还是右继续查找，避免全排序** |
| targetIndex = n-k | 第 k 大 = 升序排列后的第 n-k 个元素 |
| 随机枢轴 | 随机化避免最坏 O(n²)，保证期望 O(n) |
| 另一种解法 | 小顶堆 O(n log k) —— 维护大小为 k 的小顶堆，堆顶就是第 k 大 |

### 一句话记住

> **「快速选择找第 k 大，分区看枢轴位置——随机化保证 O(n)。」**

---

## 题目：前 K 个高频元素（Top K Frequent Elements）

**LeetCode 347 | 堆 Hot100 | 难度：🟡 中等**

### 题目描述

给你一个整数数组 `nums` 和一个整数 `k`，请你返回其中出现频率前 `k` 高的元素。你可以按**任意顺序**返回答案。

### 示例

```
输入：nums = [1,1,1,2,2,3], k = 2
输出：[1,2]

输入：nums = [1], k = 1
输出：[1]
```

---

## 解法：桶排序（HashMap 计数 + 频次桶）

### 思路

常规思路是用**小顶堆**（PriorityQueue）维护频率前 k 大的元素，O(n log k)。但当频次范围可控时，**桶排序**可以做到真正的 **O(n)**。

```
1. 统计频次：用 HashMap 统计每个元素的出现次数
2. 构建桶：以频次为下标，把元素放进对应的桶里（下标 = 出现次数）
3. 取出结果：从高频桶往低频桶遍历，取前 k 个元素
```

**为什么用桶？** 频次范围是 `[1, maxCnt]`，频次本身就是天然的有序下标——从 `maxCnt` 往下遍历，就是按频率从高到低取元素，不需要额外排序。

### 思考方式图解

```
nums = [1,1,1,2,2,3], k = 2

Step 1: 统计频次
  {1:3, 2:2, 3:1}
  maxCnt = 3

Step 2: 构建桶（下标 = 频次）
  buckets[0] = []      ← 不用
  buckets[1] = [3]     ← 出现 1 次
  buckets[2] = [2]     ← 出现 2 次
  buckets[3] = [1]     ← 出现 3 次

Step 3: 从高到低取 k=2 个
  i=3: 取 1 → ans = [1]
  i=2: 取 2 → ans = [1, 2]

结果：[1, 2] ✅
```

### 代码实现

```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        // Step 1: 统计每个元素的频次
        Map<Integer, Integer> cnt = new HashMap<>();
        for (int x : nums) {
            cnt.put(x, cnt.getOrDefault(x, 0) + 1);
        }

        // Step 2: 构建频次桶（下标 = 频次）
        int maxCnt = Collections.max(cnt.values());
        List<Integer>[] buckets = new ArrayList[maxCnt + 1];
        for (int i = 0; i < maxCnt + 1; i++) {
            buckets[i] = new ArrayList<>();
        }
        for (Map.Entry<Integer, Integer> e : cnt.entrySet()) {
            buckets[e.getValue()].add(e.getKey());
        }

        // Step 3: 从高频到低频取 k 个
        int[] ans = new int[k];
        int j = 0;
        for (int i = maxCnt; j < k; i--) {
            for (int x : buckets[i]) {
                ans[j++] = x;
            }
        }
        return ans;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 统计 O(n) + 入桶 O(n) + 取结果 O(n) |
| 🧠 空间复杂度 | **O(n)** — HashMap + 桶数组 |

> 相比小顶堆的 O(n log k)，桶排序在频次范围 `[1, maxCnt]` 可控时可以达到真正的 O(n)。

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 桶排序（频次桶） |
| 算法类型 | 哈希表、桶排序 |
| 核心技巧 | **HashMap 统计频次后，以频次为下标构建桶，直接按频次从高到低取前 k 个** |
| 与堆排序对比 | 小顶堆 O(n log k)，桶排序 **O(n)**（频次范围不大时更优） |
| 适用条件 | 频次范围 `[1, maxCnt]` 可控，能接受 O(maxCnt) 空间 |
| 关联题目 | [215. 第 K 个最大元素](day14-堆hot100.md)——QuickSelect 找第 k 大 |
| 易错点 | `Collections.max(cnt.values())` 拿到最大频次，桶数组长度是 `maxCnt + 1` 不是 `maxCnt`；`buckets[0]` 不会用到 |

### 一句话记住

> **「HashMap 计数，频次作桶下标——从高到低取 k 个。」**

---

*练习日期：2026-07-23*
