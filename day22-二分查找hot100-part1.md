# Day 22 — 二分查找 Hot100

## 题目：搜索插入位置（Search Insert Position）

**LeetCode 35 | 二分查找 Hot100 | 难度：🟢 简单**

### 题目描述

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

请必须使用时间复杂度为 **O(log n)** 的算法。

### 示例

```
输入：nums = [1,3,5,6], target = 5
输出：2

输入：nums = [1,3,5,6], target = 2
输出：1

输入：nums = [1,3,5,6], target = 7
输出：4
```

---

## 解法：二分查找（找第一个 ≥ target 的位置）

### 思路

查找插入位置，本质上是**找第一个大于等于 target 的元素下标**（左边界二分）。

```
二分框架：
  while (left <= right):
    mid = left + (right - left) / 2
    如果 nums[mid] < target → target 在右半区 → left = mid + 1
    否则 → target 在左半区（含 mid）→ right = mid - 1
  循环结束时 left 就是答案
```

**为什么最后返回 left？** 循环结束后 `left > right`，此时 `left` 指向第一个 ≥ target 的位置（如果全部小于 target，left 会走到 `nums.length`，正好是应该插入的位置）。

### 思考方式图解

```
nums = [1, 3, 5, 6], target = 2

left=0, right=3
  mid=1: nums[1]=3 ≥ 2 → right=0
  mid=0: nums[0]=1 < 2 → left=1
  left=1 > right=0 → 结束

返回 left=1 ✅ （2 应插入到下标 1）

nums = [1, 3, 5, 6], target = 7

left=0, right=3
  mid=1: nums[1]=3 < 7 → left=2
  mid=2: nums[2]=5 < 7 → left=3
  mid=3: nums[3]=6 < 7 → left=4
  left=4 > right=3 → 结束

返回 left=4 ✅ （7 应插入到末尾）
```

### 代码实现

```java
class Solution {
    public int searchInsert(int[] nums, int target) {
        int left = 0;
        int right = nums.length - 1;

        while (left <= right) {
            int mid = left + (right - left) / 2;  // 防溢出的写法
            if (nums[mid] < target) {
                left = mid + 1;     // target 在右半区
            } else {
                right = mid - 1;    // target 在左半区（含 mid）
            }
        }

        return left;  // 第一个 ≥ target 的位置
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(log n)** — 每次排除一半 |
| 🧠 空间复杂度 | **O(1)** — 仅用了几个变量 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 二分查找（左边界） |
| 算法类型 | 二分查找 |
| 核心技巧 | **找第一个 ≥ target 的位置 = 插入位置；`nums[mid] < target` 时 left=mid+1，否则 right=mid-1** |
| 为什么返回 left | 循环结束后 left 指向第一个 ≥ target 的元素；全部小于时 left == nums.length |
| `mid = left + (right - left) / 2` | 防溢出写法，等价于 `(left + right) / 2` |
| 关联题目 | [704. 二分查找](https://leetcode.cn/problems/binary-search/)——标准二分模板 |

### 一句话记住

> **「找第一个不小于 target 的位置，左闭右闭二分——结束返回 left。」**

---

*练习日期：2026-07-31*
