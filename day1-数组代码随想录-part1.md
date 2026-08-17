# Day 1 — 数组（代码随想录）

## 题目：二分查找（Binary Search）

**LeetCode 704 | 代码随想录 | 难度：🟢 简单**

### 题目描述

给定一个 `n` 个元素**有序的（升序）整型数组** `nums` 和一个目标值 `target`，写一个函数搜索 `nums` 中的 `target`。

- 如果目标值存在，返回其下标；
- 否则返回 `-1`。

你可以假设 `nums` 中的所有元素是**不重复**的，并且 `nums` 是**升序**排列的。

### 示例

```
输入：nums = [-1,0,3,5,9,12], target = 9
输出：4
解释：9 出现在 nums 中并且下标为 4

输入：nums = [-1,0,3,5,9,12], target = 2
输出：-1
解释：2 不存在于 nums 中，返回 -1
```

---

## 解法：二分查找（闭区间写法 + 首尾剪枝）

### 思路

二分查找的本质是「**在有序区间里不断砍掉一半**」。因为数组已经升序排列，所以：

- `nums[mid] == target` → 命中，直接返回 `mid`
- `nums[mid] < target`  → 目标只可能在右半边 → `left = mid + 1`
- `nums[mid] > target`  → 目标只可能在左半边 → `right = mid - 1`

**本代码的两个细节：**

1. **循环不变量 = 闭区间 `[left, right]`**：`while (left <= right)`，每轮 `mid` 算在 `[left, right]` 内部，`left/right` 的更新都「越过」了 `mid`（±1），保证不会死循环、不会漏元素。
2. **防溢出取中点**：`mid = left + ((right - left) >> 1)`，等价于 `(left + right) / 2`，但先算差值再右移，避免 `left + right` 在超大数组时整型溢出。
3. **首尾剪枝**（你加的优化）：因为数组有序，如果 `target` 连 `[nums[0], nums[n-1]]` 都落在外面，根本不可能存在，直接 `return -1`，省掉整段无谓循环。

> **右移 `>> 1` vs `/ 2`**：对非负整数结果一致，`>> 1` 位运算略快，且天然规避 `left+right` 溢出，是代码随想录推荐的写法。

### 代码实现

```java
class Solution {
    public int search(int[] nums, int target) {
        // 避免当 target 小于nums[0] 或大于 nums[nums.length - 1] 时多次循环运算
        if (target < nums[0] || target > nums[nums.length - 1]) {
            return -1;
        }
        int left = 0, right = nums.length - 1;
        while (left <= right) {                          // 闭区间 [left, right]
            int mid = left + ((right - left) >> 1);      // 防溢出取中点
            if (nums[mid] == target) {
                return mid;
            }
            else if (nums[mid] < target) {
                left = mid + 1;                          // 目标在右半边
            }
            else { // nums[mid] > target
                right = mid - 1;                         // 目标在左半边
            }
        }
        // 未找到目标值
        return -1;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(log n)** — 每轮砍掉一半，n 为数组长度（首尾剪枝为最坏情况省下的常数代价） |
| 🧠 空间复杂度 | **O(1)** — 只用了几个指针变量 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 二分查找（Binary Search） |
| 算法类型 | 二分、数组 |
| 核心技巧 | **有序数组 + 闭区间 `[left, right]` 循环，每次砍掉一半** |
| 防溢出 | `mid = left + ((right - left) >> 1)`，避免 `left+right` 溢出 |
| 易错点 | 循环条件 `left <= right` 与 `left/right` 的 ±1 必须配套，否则会漏元素或死循环 |
| 剪枝 | 有序数组可先用首尾范围判断 `target` 是否越界，提前返回 |

### 一句话记住

> **「有序就二分，闭区间 left<=right，mid 用差值右移防溢出。」**

---

## 题目：移除元素（Remove Element）

**LeetCode 27 | 代码随想录 | 难度：🟢 简单**

### 题目描述

给你一个数组 `nums` 和一个值 `val`，你需要 **原地** 移除所有数值等于 `val` 的元素，并返回移除后数组的**新长度**。

- 元素的顺序可以**改变**。
- 你不需要考虑数组中超出新长度后面的元素。
- 要求**原地**修改，空间复杂度 O(1)。

### 示例

```
输入：nums = [3,2,2,3], val = 3
输出：2, nums = [2,2,_,_]
解释：函数返回新长度 2，nums 前两个元素被修改为 2, 2（顺序不限）

输入：nums = [0,1,2,2,3,0,4,2], val = 2
输出：5, nums = [0,1,4,0,3,_,_,_]
解释：函数返回新长度 5，前 5 个元素为 [0,1,4,0,3]（顺序不限）
```

---

## 解法：快慢指针（双指针）

### 思路

这道题的关键是「**原地**」——不能用额外数组，只能在原数组上覆盖。

用两个指针分工：

- `fastIndex`（快指针）：一路向前扫描，考察每一个元素
- `slowIndex`（慢指针）：指向「下一个**有效元素**该放的位置」

规则：只有当 `nums[fastIndex] != val` 时，才把这个元素**搬到** `slowIndex` 处，并 `slow++` 前移。等于 `val` 的元素直接被快指针跳过、不搬，相当于被后续有效元素覆盖掉。

```
nums = [0,1,2,2,3,0,4,2], val = 2

fast 扫到 2 → 跳过（不搬）
fast 扫到 0,1,3,0,4 → 依次搬到 slow 指向的位置
最后 slow 停在 5 → 新长度 = 5 ✅
```

> **为什么最后 `return slowIndex` 就是长度？** 因为 `slowIndex` 表示的是「已经确认的有效元素个数」，有效元素恰好从下标 0 连续排到 `slowIndex-1`，所以长度就是 `slowIndex`。

### 代码实现

```java
class Solution {
    public int removeElement(int[] nums, int val) {
        // 快慢指针
        int slowIndex = 0;
        for (int fastIndex = 0; fastIndex < nums.length; fastIndex++) {
            if (nums[fastIndex] != val) {
                nums[slowIndex] = nums[fastIndex];   // 把有效元素搬到慢指针位置
                slowIndex++;
            }
        }
        return slowIndex;                              // 有效元素的个数 = 新长度
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 快指针只遍历一次 |
| 🧠 空间复杂度 | **O(1)** — 原地修改，只用两个指针 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 快慢指针（双指针） |
| 算法类型 | 数组、双指针 |
| 核心技巧 | **快指针探路、慢指针收拢有效元素，不等于 val 才搬** |
| 适用场景 | 原地删除/过滤数组元素，且不需要保持相对顺序的"去值" |
| 易错点 | 返回的是 `slowIndex`（有效个数），不是 `nums.length`；覆盖不会破坏未考察的元素 |

### 一句话记住

> **「快指针找有效，慢指针收拢它；不等于 val 才搬，slow 就是新长度。」**

---

## 题目：有序数组的平方（Squares of a Sorted Array）

**LeetCode 977 | 代码随想录 | 难度：🟢 简单**

### 题目描述

给你一个按 **非递减顺序** 排序的整数数组 `nums`，返回 **每个数字的平方** 组成的新数组，要求也按 **非递减顺序** 排序。

### 示例

```
输入：nums = [-4,-1,0,3,10]
输出：[0,1,9,16,100]
解释：平方后 = [16,1,0,9,100]，排序后 = [0,1,9,16,100]

输入：nums = [-7,-3,2,3,11]
输出：[4,9,9,49,121]
```

---

## 解法：双指针（头尾相向）

### 思路

数组已经升序，但**平方后最大的数一定在两端**（要么最大的正数，要么最小的负数），中间小。

所以思路很自然——**从两端往中间夹**，每次取「当前两端中平方更大的一方」放到结果数组的**末尾**（从大到小填），直到两端相遇。

```
nums = [-4,-1,0,3,10]
left=-4, right=10 → 10²=100 最大 → 放末尾：result[4]=100, right--
left=-4, right=3  → (-4)²=16 > 3²=9 → result[3]=16, left++
left=-1, right=3  → 3²=9 > (-1)²=1 → result[2]=9,  right--
left=-1, right=0  → (-1)²=1 > 0²=0  → result[1]=1,  left++
left=0, right=0   → 0²=0 → result[0]=0, 结束
→ [0,1,9,16,100] ✅
```

> **为什么不用先平方再排序？** 先平方再排序是 `O(n log n)`；双指针利用「已排序」性质做到 `O(n)`，且空间只用结果数组（题目要求的输出空间）。

### 代码实现

```java
class Solution {
    public int[] sortedSquares(int[] nums) {
        int right = nums.length - 1;
        int left = 0;
        int[] result = new int[nums.length];
        int index = result.length - 1;          // 从结果数组末尾开始填（从大到小）
        while (left <= right) {
            if (nums[left] * nums[left] > nums[right] * nums[right]) {
                // 负数平方可能更大，左边的平方胜出
                result[index--] = nums[left] * nums[left];
                ++left;
            } else {
                // 右边（正数）平方胜出
                result[index--] = nums[right] * nums[right];
                --right;
            }
        }
        return result;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 左右指针各走一遍，n 为数组长度 |
| 🧠 空间复杂度 | **O(n)** — 结果数组（题目要求的输出空间） |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 双指针（头尾相向） |
| 算法类型 | 数组、双指针 |
| 核心技巧 | **平方最大值在两端，从两端夹、往结果尾部从大到小填** |
| 适用场景 | 已排序数组的「平方/绝对值」等单调变换后仍需有序的场景 |
| 易错点 | 用 `index--` 从尾部填，循环条件 `left <= right` 包含两端相等那次 |

### 一句话记住

> **「有序数组平方，大数在两端；头尾夹逼，尾部从大到小填。」**

---

*练习日期：2026-08-17*
