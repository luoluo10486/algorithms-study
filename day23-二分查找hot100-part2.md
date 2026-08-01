# Day 23 — 二分查找 Hot100（Part 2）

## 题目：搜索二维矩阵（Search a 2D Matrix）

**LeetCode 74 | 二分查找 Hot100 | 难度：🟡 中等**

### 题目描述

给你一个满足下述两条属性的 `m x n` 整数矩阵：

- 每行中的整数从左到右按**非递减**顺序排列
- 每行的第一个整数大于前一行的最后一个整数

给你一个整数 `target`，如果 `target` 在矩阵中，返回 `true`；否则，返回 `false`。

### 示例

```
输入：matrix = [[1,3,5,7],
               [10,11,16,20],
               [23,30,34,60]], target = 3
输出：true

输入：matrix = [[1,3,5,7],
               [10,11,16,20],
               [23,30,34,60]], target = 13
输出：false
```

---

## 解法：二维展平 + 开区间二分

### 思路

矩阵满足「每行递增 + 下一行第一个数比上一行最后一个数大」——意味着**整个矩阵按行展平后是一个严格递增的一维数组**。

所以可以把二维矩阵当作长度为 `m*n` 的一维数组做二分查找，下标映射：

```
一维下标 mid → 二维坐标：
  行 = mid / n
  列 = mid % n
```

**开区间二分模板**（灵茶山艾府风格）：

```
left = -1, right = m*n  （开区间 (-1, m*n)，两侧都是"哨兵"）

while (left + 1 < right):
  mid = left + (right - left) / 2
  x = matrix[mid/n][mid%n]
  如果 x == target → true
  如果 x < target → left = mid（说明 target 在右半区）
  否则 → right = mid（target 在左半区）

循环结束时没有找到 → false
```

### 思考方式图解

```
matrix = [[1,3,5,7],
          [10,11,16,20],
          [23,30,34,60]]

m=3, n=4, m*n=12, target=3

展平后（逻辑上）：[1,3,5,7, 10,11,16,20, 23,30,34,60]
                  下标：0 1 2 3  4  5  6  7   8  9 10 11

left=-1, right=12
  mid=5:   x=matrix[5/4][5%4]=matrix[1][1]=11  > 3 → right=5
  mid=2:   x=matrix[2/4][2%4]=matrix[0][2]=5   > 3 → right=2
  mid=0:   x=matrix[0][0]=1                    < 3 → left=0
  mid=1:   x=matrix[0][1]=3                    == 3 → true ✅
```

### 代码实现

```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        int m = matrix.length;
        int n = matrix[0].length;

        int left = -1;       // 开区间左哨兵
        int right = m * n;   // 开区间右哨兵

        while (left + 1 < right) {
            int mid = left + (right - left) / 2;
            // 一维下标 → 二维坐标
            int x = matrix[mid / n][mid % n];

            if (x == target) {
                return true;
            } else if (x < target) {
                left = mid;
            } else {
                right = mid;
            }
        }

        return false;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(log(mn))** — 展平后二分 |
| 🧠 空间复杂度 | **O(1)** — 仅用了几个变量 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 二维展平 + 开区间二分 |
| 算法类型 | 二分查找、矩阵 |
| 核心技巧 | **`mid / n` 得到行、`mid % n` 得到列——把二维矩阵映射成一维数组二分** |
| 开区间写法 | `left=-1, right=m*n`，循环条件 `left+1 < right`，终止时 `left+1 == right` |
| 与 240 的区别 | 240 的行列都升序但行首不一定 > 行尾，不能展平；74 是**严格展平可二分** |
| 关联题目 | [240. 搜索二维矩阵 II](day19-矩阵hot100.md)——Z 字形查找（不能展平） |

### 一句话记住

> **「矩阵展平一维化，mid 除 n 得行、模 n 得列——开区间二分一次搞定。」**

---

## 题目：在排序数组中查找元素的第一个和最后一个位置（Find First and Last Position of Element in Sorted Array）

**LeetCode 34 | 二分查找 Hot100 | 难度：🟡 中等**

### 题目描述

给你一个按照非递减顺序排列的整数数组 `nums`，和一个目标值 `target`。请你找出给定目标值在数组中的**开始位置**和**结束位置**。

如果数组中不存在目标值 `target`，返回 `[-1, -1]`。

你必须设计并实现时间复杂度为 **O(log n)** 的算法解决此问题。

### 示例

```
输入：nums = [5,7,7,8,8,10], target = 8
输出：[3,4]

输入：nums = [5,7,7,8,8,10], target = 6
输出：[-1,-1]

输入：nums = [], target = 0
输出：[-1,-1]
```

---

## 解法：两次二分（lowerBound 求左边界）

### 思路

求「第一个 ≥ target 的下标」就是**左边界二分**（Day 22 的模板）。

```
思路：
  左边界 start = lowerBound(nums, target)          ← 第一个 ≥ target 的位置
  如果 start 越界 或 nums[start] != target → 不存在 → [-1, -1]
  右边界 end = lowerBound(nums, target + 1) - 1     ← 第一个 ≥ target+1 的位置再前移一位
  返回 [start, end]
```

**为什么用 `lowerBound(target+1) - 1` 求右边界？**

`lowerBound(target+1)` 是第一个 ≥ target+1 的下标，也就是第一个**大于 target** 的下标。它前移一位，正好是最后一个等于 target 的位置。

### 思考方式图解

```
nums = [5,7,7,8,8,10], target = 8

start = lowerBound(8) = 3      （第一个 ≥ 8 的位置）
  nums[3] = 8 == target → 存在 ✅

end = lowerBound(9) - 1 = 5 - 1 = 4   （第一个 ≥ 9 的位置是 5，前移一位是 4）

返回 [3, 4] ✅
```

```
nums = [5,7,7,8,8,10], target = 6

start = lowerBound(6) = 1
  nums[1] = 7 != 6 → 不存在 → [-1, -1] ✅
```

### 代码实现

```java
class Solution {
    public int[] searchRange(int[] nums, int target) {
        int start = lowerBound(nums, target);      // 第一个 ≥ target 的位置
        if (start == nums.length || nums[start] != target) {  // 不存在
            return new int[]{-1, -1};
        }
        int end = lowerBound(nums, target + 1) - 1;  // 第一个 > target 的位置前移一位
        return new int[]{start, end};
    }

    // 左边界二分：返回第一个 ≥ target 的下标（不存在时返回 nums.length）
    private int lowerBound(int[] nums, int target) {
        int left = 0;
        int right = nums.length - 1;
        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] >= target) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
        return left;
    }
}
```

### ⚠️ 易错点：数组越界风险

```java
// ❌ 错误写法：先访问 nums[start] 再判断越界——start 可能 == nums.length，直接越界
if (nums[start] != target || start == nums.length) {  // 数组越界！
    return new int[]{-1, -1};
}

// ✅ 正确写法：先判断越界（短路求值），再访问数组
if (start == nums.length || nums[start] != target) {  // 短路：start==length 时不再访问
    return new int[]{-1, -1};
}
```

**为什么 `nums[start] != target || start == nums.length` 会越界？**

当 `target` 大于数组中所有元素时，`lowerBound` 返回 `nums.length`。此时：
- `nums[start]` = `nums[nums.length]` → **IndexOutOfBoundsException** ❌
- `||` 运算符会**先算左边**，左边已经越界，右边根本没机会执行

**正确写法利用短路求值**：`start == nums.length || ...` —— 左边为 true 时直接短路，右边的 `nums[start]` 不会执行，避免越界。

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(log n)** — 两次二分 |
| 🧠 空间复杂度 | **O(1)** — 仅用了几个变量 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 两次二分（lowerBound） |
| 算法类型 | 二分查找 |
| 核心技巧 | **左边界 = lowerBound(target)；右边界 = lowerBound(target+1) - 1** |
| 判断不存在 | `start == nums.length \|\| nums[start] != target`——注意先判越界（短路）再访问 |
| lowerBound | 左边界二分模板，返回第一个 ≥ target 的下标，找不到返回 nums.length |
| 关联题目 | [35. 搜索插入位置](day22-二分查找hot100-part1.md)——左边界二分模板 |

### 一句话记住

> **「两次 lowerBound 定区间——先判越界再取值。」**

---

## 题目：寻找旋转排序数组中的最小值（Find Minimum in Rotated Sorted Array）

**LeetCode 153 | 二分查找 Hot100 | 难度：🟡 中等**

### 题目描述

已知一个长度为 `n` 的数组，预先按照升序排列，经由 `1` 到 `n` 次旋转后，得到输入数组。

例如，原数组 `nums = [0,1,2,4,5,6,7]` 若变化为 `[4,5,6,7,0,1,2]`。

给你一个元素值互不相同的数组 `nums`，它原来是一个升序排列的数组，并按上述情形进行了多次旋转。请你找出并返回数组中的**最小元素**。

你必须设计一个时间复杂度为 **O(log n)** 的算法解决此问题。

### 示例

```
输入：nums = [3,4,5,1,2]
输出：1

输入：nums = [4,5,6,7,0,1,2]
输出：0

输入：nums = [11,13,15,17]
输出：11（没有旋转）
```

---

## 解法：开区间二分（与右端点比较）

### 思路

旋转数组的特点是**两段升序拼在一起**（前段大、后段小）。最小值就在分界点。

核心比较对象：**`nums[mid]` 与 `nums[right]`（右端点）**。

```
如果 nums[mid] < nums[right]：
  → mid 在最小值的右侧（含 mid），最小值在 [left+1, mid] 区间 → right = mid
否则：
  → mid 在最小值的左侧，最小值在 [mid+1, right] → left = mid
```

**为什么拿 mid 和 right 比，而不是和 left 比？**

和 `right` 比较：数组是**右半段递增**的（`nums[right]` 是右段的最后一个元素），可以明确判断 mid 在哪一段。

```
以 [4,5,6,7,0,1,2] 为例：
  nums[mid] < nums[right] → mid 和 right 在同一段递增区间 → 最小值在左边（含 mid）
  nums[mid] > nums[right] → mid 在前段（大段），最小值在右边
```

### 思考方式图解

```
nums = [4,5,6,7,0,1,2], n=7

left=-1, right=6
  mid=2:   nums[2]=6 > nums[6]=2 → left=2
  mid=4:   nums[4]=0 < nums[6]=2 → right=4
  mid=3:   nums[3]=7 > nums[4]=0 → left=3
  left+1==right → 循环结束

返回 nums[right] = nums[4] = 0 ✅
```

### 代码实现

```java
class Solution {
    public int findMin(int[] nums) {
        int n = nums.length;
        int left = -1;      // 开区间左哨兵
        int right = n - 1;  // 右边界初始化为 n-1（nums[n-1] 一定在右段）

        while (left + 1 < right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] < nums[right]) {
                right = mid;      // mid 在最小值右侧，收缩右边界
            } else {
                left = mid;       // mid 在最小值左侧，收缩左边界
            }
        }

        return nums[right];
    }
}
```

---

### 💡 闭区间写法

```java
class Solution {
    public int findMin(int[] nums) {
        int left = 0;
        int right = nums.length - 1;

        while (left < right) {          // 闭区间，循环到 left == right 时收敛
            int mid = left + (right - left) / 2;
            if (nums[mid] < nums[right]) {
                right = mid;            // mid 在最小值右侧（含 mid），右边界收缩到 mid
            } else {
                left = mid + 1;         // mid 在最小值左侧，左边界跳过 mid
            }
        }

        return nums[left];              // left == right == 最小值
    }
}
```

**两种写法对比：**

| | 开区间（left=-1） | 闭区间（left=0） |
|---|----------------|----------------|
| 区间定义 | `(left, right)` 开区间 | `[left, right]` 闭区间 |
| 循环条件 | `left + 1 < right` | `left < right` |
| 收缩规则 | 两侧都是 `right=mid / left=mid` | `right=mid`，但 `left=mid+1`（防死循环） |
| 终止状态 | `left + 1 == right`，答案 `nums[right]` | `left == right`，答案 `nums[left]` |

> 注意闭区间写法中 `nums[mid] >= nums[right]` 时是 `left = mid + 1` 而不是 `left = mid`——因为 `mid` 已经被排除在最小值候选之外；而 `nums[mid] < nums[right]` 时 `mid` 可能是最小值本身，所以 `right = mid`。

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(log n)** — 每次排除一半 |
| 🧠 空间复杂度 | **O(1)** — 仅用了几个变量 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 开区间二分（与右端点比较） |
| 算法类型 | 二分查找、数组 |
| 核心技巧 | **`nums[mid] < nums[right]` → right=mid；否则 → left=mid（开区间）/ left=mid+1（闭区间）** |
| 两种写法 | 开区间（left=-1，终止时返回 nums[right]） / 闭区间（left=0，终止时返回 nums[left]） |
| 为什么比 right 不比 left | right 永远落在递增的右段，比较结果明确；和 left 比会出现边界模糊 |
| 没有旋转时 | `[11,13,15,17]`，每次 mid 都 < right → right 不断收缩到 0，返回 nums[0] ✅ |
| 关联题目 | [33. 搜索旋转排序数组](https://leetcode.cn/problems/search-in-rotated-sorted-array/)——进阶：在旋转数组中找目标值 |

### 一句话记住

> **「和右端点比大小，小则右收、大则左进——开区间二分找最小。」**

---

*练习日期：2026-08-01*
