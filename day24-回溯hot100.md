# Day 24 — 回溯 Hot100

## 题目：全排列（Permutations）

**LeetCode 46 | 回溯 Hot100 | 难度：🟡 中等**

### 题目描述

给定一个不含重复数字的数组 `nums`，返回其所有可能的全排列。你可以**按任意顺序**返回答案。

### 示例

```
输入：nums = [1,2,3]
输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]

输入：nums = [0,1]
输出：[[0,1],[1,0]]

输入：nums = [1]
输出：[[1]]
```

---

## 解法：回溯（固定长度 path + 索引控制）

### 思路

回溯的核心是**做选择 → 递归 → 撤销选择**。全排列要求每个数字只能用一次，所以需要一个 `onPath` 数组标记哪些数字已使用。

你这份实现的特殊之处：**用 `path.set(i, nums[j])` 按索引填入，而不是 path.add / path.remove**。

```
dfs(i)：正在决定排列的第 i 个位置放什么

  如果 i == n → path 填满了 → 加入答案

  遍历每个数字 nums[j]：
    如果 nums[j] 未被使用（!onPath[j]）：
      path.set(i, nums[j])   ← 做选择：放在位置 i
      onPath[j] = true        ← 标记已用
      dfs(i+1)                ← 递归决定下一个位置
      onPath[j] = false       ← 撤销选择（回溯）
```

**为什么用 `path.set` 而不是 `add/remove`？**

`path` 是固定长度的 List（初始化为 `n` 个 null），通过 `set(i, val)` 填值、下一轮递归直接覆盖位置 i+1。这样就不需要 `remove` 操作，撤销选择只需 `onPath[j] = false`，更简洁。

### 思考方式图解

```
nums = [1,2,3]

dfs(0): 决定位置 0
  j=0(1): path[0]=1, onPath[0]=T → dfs(1)
    j=0: onPath[0]=T 跳过
    j=1(2): path[1]=2, onPath[1]=T → dfs(2)
      j=0,1 跳过
      j=2(3): path[2]=3, onPath[2]=T → dfs(3)
        i==3 → 加入 [1,2,3] ✅
      onPath[2]=F
    onPath[1]=F
    j=2(3): path[1]=3, onPath[2]=T → dfs(2)
      j=2 跳过
      j=2... 只有 j=1(2): path[2]=2 → 加入 [1,3,2] ✅
    onPath[2]=F
  onPath[0]=F
  j=1(2): path[0]=2, onPath[1]=T → dfs(1)
    ... 产生 [2,1,3] [2,3,1]
  j=2(3): path[0]=3, onPath[2]=T → dfs(1)
    ... 产生 [3,1,2] [3,2,1]

共 6 种排列 ✅
```

### 代码实现

```java
class Solution {
    public List<List<Integer>> permute(int[] nums) {
        int n = nums.length;
        List<Integer> path = Arrays.asList(new Integer[n]);  // 固定长度的路径
        boolean[] onPath = new boolean[n];                   // 标记已使用的数字
        List<List<Integer>> ans = new ArrayList<>();

        dfs(0, nums, path, onPath, ans);
        return ans;
    }

    // i 表示正在决定排列的第 i 个位置
    private void dfs(int i, int[] nums, List<Integer> path, boolean[] onPath, List<List<Integer>> ans) {
        if (i == nums.length) {              // 所有位置都填好了
            ans.add(new ArrayList<>(path));  // 拷贝一份加入答案
            return;
        }

        for (int j = 0; j < nums.length; j++) {
            if (!onPath[j]) {                // 数字 nums[j] 还未被使用
                path.set(i, nums[j]);        // 做选择：放到位置 i
                onPath[j] = true;            // 标记已用
                dfs(i + 1, nums, path, onPath, ans);  // 递归决定位置 i+1
                onPath[j] = false;           // 撤销选择（回溯）
            }
        }
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n × n!)** — n! 种排列，每种拷贝 O(n) |
| 🧠 空间复杂度 | **O(n)** — 递归栈深度 n + path/onPath 数组 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 回溯（固定长度 path） |
| 算法类型 | 回溯 |
| 核心技巧 | **`path.set(i, val)` 按索引填值 + `onPath[]` 标记已用——做选择/撤销选择只需改标记** |
| 为什么不用 add/remove | path 长度固定，set 直接覆盖，撤销只需 `onPath[j]=false`，省去 remove 操作 |
| 剪枝 | `!onPath[j]` 跳过已使用的数字（全排列不允许重复使用） |
| 关联题目 | [47. 全排列 II](https://leetcode.cn/problems/permutations-ii/)——有重复数字，需要额外去重 |

### 一句话记住

> **「位置定序，onPath 防重——set 填值、标记回溯。」**

---

*练习日期：2026-08-02*
