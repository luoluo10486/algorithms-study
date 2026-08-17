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

## 题目：搜索旋转排序数组（Search in Rotated Sorted Array）

**LeetCode 33 | 二分查找 Hot100 | 难度：🟡 中等**

### 题目描述

整数数组 `nums` 按升序排列，数组中的值**互不相同**。

在传递给函数之前，`nums` 在预先未知的某个下标 `k` 上进行了**旋转**（下标从 0 开始计数）。例如 `[0,1,2,4,5,6,7]` 在下标 3 处经旋转后可能变为 `[4,5,6,7,0,1,2]`。

给你旋转后的数组 `nums` 和一个整数 `target`，如果 `nums` 中存在这个目标值，则返回它的下标，否则返回 `-1`。

你必须设计一个时间复杂度为 **O(log n)** 的算法解决此问题。

### 示例

```
输入：nums = [4,5,6,7,0,1,2], target = 0
输出：4

输入：nums = [4,5,6,7,0,1,2], target = 3
输出：-1

输入：nums = [1], target = 0
输出：-1
```

---

## 解法：找最小值 + 分段二分（复用模板）

### 思路

旋转数组被最小值分成**两段递增区间**，target 必然在其中一段。所以可以**先找到最小值的位置 `i`，再判断 target 在哪一段，对该段做二分**。

```
步骤：
1. 用 153 的模板找到最小值下标 i
2. 判断 target 与 nums[n-1]（数组末尾）的大小关系：
   - target > nums[n-1] → target 在前半段（大段）[0, i)，二分区间 (-1, i)
   - target ≤ nums[n-1] → target 在后半段（小段）[i, n-1]，二分区间 (i-1, n)
3. 在该区间内用 lowerBound 二分查找
```

**为什么用 `nums[n-1]` 判断？** 旋转后数组是「大段 + 小段」，`nums[n-1]` 是小段的最后一个元素。如果 target 比它大，说明 target 只可能在前半段；否则在后半段。

### 思考方式图解

```
nums = [4,5,6,7,0,1,2], target = 0

Step 1: 找最小值
  findMin → i = 4（nums[4]=0）

Step 2: 判断在哪一段
  target(0) ≤ nums[6](2) → 在后半段 → 二分区间 (i-1, n) = (3, 7)

Step 3: lowerBound(nums, 3, 7, 0)
  mid=5: nums[5]=1 > 0 → right=5
  mid=4: nums[4]=0 == 0 → right=4
  left+1==right → nums[4]==0 → 返回 4 ✅
```

### 代码实现

```java
class Solution {
    public int search(int[] nums, int target) {
        int i = findMin(nums);     // 最小值下标（= 旋转点）
        int n = nums.length;

        // target 在哪一段？
        if (target > nums[n - 1]) {
            return lowerBound(nums, -1, i, target);      // 前半段 (大段)
        }
        return lowerBound(nums, i - 1, n, target);       // 后半段 (小段)
    }

    // 开区间二分：在 (left, right) 中找 target，找不到返回 -1
    private int lowerBound(int[] nums, int left, int right, int target) {
        while (left + 1 < right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] < target) {
                left = mid;
            } else {
                right = mid;
            }
        }
        return nums[right] == target ? right : -1;
    }

    // 153 模板：找最小值下标
    private int findMin(int[] nums) {
        int n = nums.length;
        int left = -1;
        int right = n - 1;
        while (left + 1 < right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] < nums[n - 1]) {   // 和数组末尾比较
                right = mid;
            } else {
                left = mid;
            }
        }
        return right;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(log n)** — 找最小值 O(log n) + 二分 O(log n) |
| 🧠 空间复杂度 | **O(1)** — 仅用了几个变量 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 找最小值 + 分段二分 |
| 算法类型 | 二分查找、数组 |
| 核心技巧 | **先 findMin 定位旋转点，用 `target > nums[n-1]` 判断在前半段还是后半段，再对那段二分** |
| 模板复用 | 直接复用了 153（findMin）和开区间 lowerBound 两个模板 |
| 边界处理 | 前半段开区间 `(-1, i)`，后半段 `(i-1, n)`——注意区间定义不能重叠 |
| 关联题目 | [153. 寻找旋转排序数组中的最小值](day23-二分查找hot100-part2.md)——findMin 的来源 |

### 一句话记住

> **「先找旋转点，比末尾定段，再对那段二分。」**

---

## 题目：寻找两个正序数组的中位数（Median of Two Sorted Arrays）

**LeetCode 4 | 二分查找 Hot100 | 难度：🔴 困难**

### 题目描述

给定两个长度分别为 `m` 和 `n` 的**升序（正序）**数组 `nums1` 和 `nums2`。

请你找出并返回这两个**正序数组的****中位数**，并且算法的时间复杂度应为 **O(log(m + n))**。

### 示例

```
输入：nums1 = [1,3], nums2 = [2]
输出：2.00000
解释：合并后为 [1,2,3]，中位数 2

输入：nums1 = [1,2], nums2 = [3,4]
输出：2.50000
解释：合并后为 [1,2,3,4]，中位数 (2+3)/2 = 2.5
```

---

## 解法一：合并 + 排序（直观法，你的代码）

### 思路

最直白的做法：把两个数组合并成一个，排序，然后按总长度取中位数。

```
合并 → 排序 → 取中点
  奇数长度：中位数 = merged[mid]
  偶数长度：中位数 = (merged[mid] + merged[mid+1]) / 2.0
```

### 代码实现

```java
class Solution {
    public double findMedianSortedArrays(int[] a, int[] b) {
        int m = a.length;
        int n = b.length;
        int[] merged = new int[m + n];
        System.arraycopy(a, 0, merged, 0, m);
        System.arraycopy(b, 0, merged, m, n);
        Arrays.sort(merged);

        int s = m + n;
        int k = (s - 1) / 2;
        return s % 2 > 0 ? merged[k] : (merged[k] + merged[k + 1]) / 2.0;
    }
}
```

> **正确性**：你的代码完全正确。中点下标 `k = (s-1)/2` 对奇数 `s` 正好取中间、对偶数 `s` 取左中，`merged[k]`/`merged[k+1]` 即两个中间元素；`/2.0` 强制浮点除法，结果正确。
> **代价**：`Arrays.sort` 是 O((m+n)log(m+n))，**没有用到"两个数组已经有序"这个关键性质**，且新建了 O(m+n) 数组。面试中通常要求 O(log(m+n))，所以需要解法二。

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O((m+n)log(m+n))** — 排序主导 |
| 🧠 空间复杂度 | **O(m+n)** — 合并数组 |

---

## 解法二：二分查找划分（真正的 O(log(min(m,n))) 解法）

### 思路

中位数本质上是在「把两个数组拼起来后，前一半（较小的一半）的分割线」。核心思想：**在较短的数组上二分一个分割点 `i`，让两个数组被切成「左半 / 右半」，且左半恰好包含较小的一半元素**。

```
设 a 取前 i 个、b 取前 j 个进入"左半"，则：
  j = (m + n + 1) / 2 - i        ← 保证左半元素数 = ceil((m+n)/2)，比右半多 0 或 1 个

合法分割的判定（左半最大值 ≤ 右半最小值）：
  a[i-1] <= b[j]   且   b[j-1] <= a[i]

不满足就调整 i：
  a[i-1] > b[j]  → i 太大（a 左边进多了）→ 左移 i
  否则           → i 可行或偏小 → 右移 i 试探更优

取到划分后：
  奇数 (m+n) → 中位数 = max(a[i-1], b[j-1])            （左半最大值）
  偶数 (m+n) → 中位数 = (max(左半) + min(右半)) / 2.0
```

**为什么在较短数组上二分？** 复杂度由二分区间长度决定，取 `min(m,n)` 保证最坏 O(log(min(m,n))) ≤ O(log(m+n))。

### 思考方式图解

```
a = [1, 3],  b = [2]       m=2, n=1, totalLeft = (2+1+1)/2 = 2
（a 较短，在 a 上二分 i）

i=1: j = 2-1 = 1
  a[i-1]=a[0]=1, b[j]=b[1] 越界→MAX
  b[j-1]=b[0]=2, a[i]=a[1]=3
  判断 a[i-1](1) <= b[j](MAX) ✅ 且 b[j-1](2) <= a[i](3) ✅ → 合法
  左半 = {1, 2}，右半 = {3} → 奇数 → 中位数 = max(1,2) = 2 ✅
```

### 代码实现

```java
class Solution {
    public double findMedianSortedArrays(int[] a, int[] b) {
        // 保证在较短数组上二分，复杂度 O(log(min(m,n)))
        if (a.length > b.length) {
            int[] t = a; a = b; b = t;
        }
        int m = a.length, n = b.length;
        int totalLeft = (m + n + 1) / 2;   // 左半应有元素个数（奇数时左半多一个）

        int left = 0, right = m;           // 在 a 的「前 i 个」上二分
        while (left < right) {
            int i = left + (right - left) / 2;   // a 取前 i 个
            int j = totalLeft - i;               // b 取前 j 个（保证左右数量平衡）
            if (a[i - 1] > b[j]) {
                right = i - 1;                    // i 太大，a 左边进多了 → 左移
            } else {
                left = i + 1;                     // i 可行或偏小 → 右移试探
            }
        }
        int i = left, j = totalLeft - i;

        // 边界：i==0 表示 a 全在右半；i==m 表示 a 全在左半（b 同理）
        int aLeft  = i == 0 ? Integer.MIN_VALUE : a[i - 1];
        int aRight = i == m ? Integer.MAX_VALUE : a[i];
        int bLeft  = j == 0 ? Integer.MIN_VALUE : b[j - 1];
        int bRight = j == n ? Integer.MAX_VALUE : b[j];

        if ((m + n) % 2 == 1) {
            return Math.max(aLeft, bLeft);                 // 奇数：左半最大值
        }
        return (Math.max(aLeft, bLeft) + Math.min(aRight, bRight)) / 2.0;
    }
}
```

> **关键点**：用 `Integer.MIN_VALUE / MAX_VALUE` 处理"某数组全部落入左半 / 右半"的边界，避免 `a[i-1]`/`a[i]` 越界。循环的 `while (left < right)` + `left = i+1 / right = i-1` 是闭区间二分的经典收口写法，终止时 `left == right == i`。

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(log(min(m, n)))** — 在较短数组上二分 |
| 🧠 空间复杂度 | **O(1)** — 仅用几个变量 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 二分查找划分（Partition / 划分数组找中位数） |
| 算法类型 | 二分查找、双数组、困难题 |
| 朴素解法 | 合并 + 排序 O((m+n)log(m+n))、O(m+n) 空间（你的代码，正确但非最优） |
| 最优解法 | **在较短数组二分分割点 i，令 j = (m+n+1)/2 - i，使左半 = 较小一半；判定 a[i-1]≤b[j] 且 b[j-1]≤a[i]** |
| 中位数取法 | 奇数 → `max(左半)`；偶数 → `(max(左半)+min(右半))/2.0` |
| 边界处理 | 用 `±∞` 哨兵处理某数组全在单侧 |
| 复杂度 | O(log(min(m,n))) 时间、O(1) 空间，满足题目要求 |
| 关联题目 | [33/153. 旋转数组](day23-二分查找hot100-part2.md)、[34. 区间查找](day23-二分查找hot100-part2.md)——同属二分家族 |

### 一句话记住

> **「两有序数组求中位 = 在短数组上二分分割点，让左半装下较小一半，a[i-1]≤b[j] 且 b[j-1]≤a[i] 即合法。」**

---

*练习日期：2026-08-01*
