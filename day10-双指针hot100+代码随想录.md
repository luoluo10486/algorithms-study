# Day 10 — 双指针 Hot100 + 代码随想录

## 题目：移除元素（Remove Element）

**LeetCode 27 | 代码随想录 | 难度：🟢 简单**

### 题目描述

给你一个数组 `nums` 和一个值 `val`，你需要**原地**移除所有数值等于 `val` 的元素。元素的顺序可能发生改变。然后返回 `nums` 中与 `val` 不同的元素的数量。

不要使用额外的数组空间，你必须仅使用 **O(1) 额外空间**并**原地**修改输入数组。

### 示例

```
输入：nums = [3,2,2,3], val = 3
输出：2, nums = [2,2,_,_]

输入：nums = [0,1,2,2,3,0,4,2], val = 2
输出：5, nums = [0,1,4,0,3,_,_,_]
```

---

## 解法：快慢指针

### 思路

双指针（快慢指针）是最经典的做法——**快指针遍历整个数组，慢指针指向下一个要放置合法元素的位置**。

```
快指针 fast：负责探路，找不等于 val 的元素
慢指针 slow：负责占位，指向下一个要覆盖的位置
```

当 `nums[fast] != val` 时，把它复制到 `nums[slow]` 位置，`slow++`。这样遍历结束后，`[0, slow)` 区间内的所有元素都不是 `val`。

### 思考方式图解

```
nums = [3, 2, 2, 3], val = 3

初始：fast=0, slow=0
  fast=0: nums[0]=3 == val → 跳过，slow 不动
  fast=1: nums[1]=2 != val → nums[0]=2, slow=1
  fast=2: nums[2]=2 != val → nums[1]=2, slow=2
  fast=3: nums[3]=3 == val → 跳过

结果：[2, 2, ...], slow=2 ✅
```

```
nums = [0,1,2,2,3,0,4,2], val = 2

fast=0: 0 != 2 → nums[0]=0, slow=1
fast=1: 1 != 2 → nums[1]=1, slow=2
fast=2: 2 == 2 → 跳过
fast=3: 2 == 2 → 跳过
fast=4: 3 != 2 → nums[2]=3, slow=3
fast=5: 0 != 2 → nums[3]=0, slow=4
fast=6: 4 != 2 → nums[4]=4, slow=5
fast=7: 2 == 2 → 跳过

结果：[0,1,3,0,4, ...], slow=5 ✅
```

### 代码实现

```java
class Solution {
    public int removeElement(int[] nums, int val) {
        int slow = 0;
        for (int fast = 0; fast < nums.length; fast++) {
            if (nums[fast] != val) {
                nums[slow] = nums[fast];
                slow++;
            }
        }
        return slow;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 一次遍历 |
| 🧠 空间复杂度 | **O(1)** — 原地修改 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 快慢指针 |
| 算法类型 | 双指针、数组 |
| 核心技巧 | **快指针探路找合法元素，慢指针占位标记新数组长度** |
| 适用场景 | 原地删除、原地去重（同款模板） |
| 关联题目 | [26. 删除有序数组中的重复项](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/)—同一套快慢指针模板 |
| 易错点 | slow 指向的是**下一个要填的位置**，所以最终 slow 就是新数组的长度；不需要真的删除元素，覆盖即可 |

### 一句话记住

> **「快针探路慢针等，不等于 val 就覆盖。」**

---

*练习日期：2026-07-18*
