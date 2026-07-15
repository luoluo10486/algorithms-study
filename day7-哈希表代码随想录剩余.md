# Day 7 — 哈希表代码随想录（剩余）

## 题目：四数相加 II（4Sum II）

**LeetCode 454 | 代码随想录 | 难度：🟡 中等**

### 题目描述

给你四个整数数组 `nums1`、`nums2`、`nums3` 和 `nums4`，数组长度都是 `n`，请你计算有多少个元组 `(i, j, k, l)` 满足：

> `nums1[i] + nums2[j] + nums3[k] + nums4[l] == 0`

### 示例

```
输入：nums1 = [1,2], nums2 = [-2,-1], nums3 = [-1,2], nums4 = [0,2]
输出：2
解释：
两个元组如下：
1. (0, 0, 0, 1) → nums1[0] + nums2[0] + nums3[0] + nums4[1] = 1 + (-2) + (-1) + 2 = 0
2. (1, 1, 0, 0) → nums1[1] + nums2[1] + nums3[0] + nums4[0] = 2 + (-1) + (-1) + 0 = 0

输入：nums1 = [0], nums2 = [0], nums3 = [0], nums4 = [0]
输出：1
```

---

## 解法：分组 + HashMap

### 思路

暴力四层循环是 O(n⁴)，显然不可取。

核心思想：**把四数之和拆成两半，两两分组**。

> **AB 一组，CD 一组**：
> - 先统计 A+B 的所有可能和，以及每个和出现的次数 → 存入 HashMap
> - 再遍历 C+D，对于每个和，在 map 中查找 `0 - (c + d)`，累加次数

这样复杂度从 O(n⁴) 降到 **O(n²)**。

### 思考方式图解

```
nums1 = [1,2], nums2 = [-2,-1], nums3 = [-1,2], nums4 = [0,2]

Step 1: 统计 A+B 的所有和
  nums1\nums2  -2   -1
       1      -1    0       → map: {-1:1, 0:1}
       2       0    1       → map: {-1:1, 0:2, 1:1}

Step 2: 遍历 C+D，在 map 中找 -(c+d)
  nums3\nums4   0    2
      -1       -1   -3      → -(c+d): 1→res+=map.get(1)=1,  3→res+=0
       2       -2   -4      → -(c+d): 2→res+=0,  4→res+=0

结果：res = 2 ✅
```

### 代码实现

```java
class Solution {
    public int fourSumCount(int[] nums1, int[] nums2, int[] nums3, int[] nums4) {
        int res = 0;
        Map<Integer, Integer> map = new HashMap<>();

        // Step 1: 统计 A+B 的所有和及出现次数
        for (int i : nums1) {
            for (int j : nums2) {
                int sum = i + j;
                map.put(sum, map.getOrDefault(sum, 0) + 1);
            }
        }

        // Step 2: 遍历 C+D，找 -(c+d) 在 map 中的次数
        for (int i : nums3) {
            for (int j : nums4) {
                res += map.getOrDefault(0 - i - j, 0);
            }
        }

        return res;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n²)** — 两个双层循环 |
| 🧠 空间复杂度 | **O(n²)** — HashMap 最多存储 n² 种和 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 分组 + HashMap |
| 算法类型 | 哈希表、分治 |
| 核心技巧 | **AB 一组、CD 一组，空间换时间——四数降两数** |
| 适用场景 | 多数组求和为定值，分组降低复杂度 |
| 对比 | 四数之和（一个数组内）用排序+双指针，四数相加 II（四个数组）用 HashMap |
| 易错点 | 用 `getOrDefault` 代替手动判空；`0 - i - j` 注意运算顺序 |

### 一句话记住

> **「两两分组算和，HashMap 次数累加。」**

---

*练习日期：2026-07-15*
