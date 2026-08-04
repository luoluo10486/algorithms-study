# Day 25 — 回溯 Hot100（Part 2）

## 题目：电话号码的字母组合（Letter Combinations of a Phone Number）

**LeetCode 17 | 回溯 Hot100 | 难度：🟡 中等**

### 题目描述

给定一个仅包含数字 `2-9` 的字符串，返回所有它能表示的字母组合。答案可以按任意顺序返回。

数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。

```
2: abc    3: def    4: ghi
5: jkl    6: mno    7: pqrs
8: tuv    9: wxyz
```

### 示例

```
输入：digits = "23"
输出：["ad","ae","af","bd","be","bf","cd","ce","cf"]

输入：digits = ""
输出：[]

输入：digits = "2"
输出：["a","b","c"]
```

---

## 解法：回溯（字符数组 path 直接覆盖）

### 思路

每个数字对应一组字母，逐位选择：**第 i 位从 digits[i] 对应的字母集合中选一个**。

```
dfs(i)：正在决定第 i 位数字对应哪个字母

  如果 i == n → path 填满 → 加入答案

  遍历 digits[i] 对应的所有字母 letters：
    path[i] = c          ← 直接覆盖（字符数组天然支持）
    dfs(i+1)             ← 递归决定下一位
```

**为什么用 `char[] path`？** 每个位置只填一次字符，用 `path[i] = c` 直接覆盖，天然支持回溯，不需要 `add/remove`，也不需要撤销操作。

### 思考方式图解

```
digits = "23"

数字 2 → a,b,c      数字 3 → d,e,f

dfs(0): 决定第 0 位（2 的字母）
  path[0]='a' → dfs(1)
    决定第 1 位（3 的字母）
    path[1]='d' → 加入 "ad"
    path[1]='e' → 加入 "ae"
    path[1]='f' → 加入 "af"
  path[0]='b' → dfs(1) → "bd","be","bf"
  path[0]='c' → dfs(1) → "cd","ce","cf"

结果：["ad","ae","af","bd","be","bf","cd","ce","cf"] ✅
```

### 代码实现

```java
class Solution {
    private static final String[] MAPPING =
        new String[]{"", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"};

    public List<String> letterCombinations(String digits) {
        int n = digits.length();
        if (n == 0) {
            return List.of();
        }

        List<String> ans = new ArrayList<>();
        char[] path = new char[n];  // 注意：path 长度一开始就是 n
        dfs(0, ans, path, digits.toCharArray());
        return ans;
    }

    // i 表示正在决定第 i 位数字
    private void dfs(int i, List<String> ans, char[] path, char[] digits) {
        if (i == digits.length) {
            ans.add(new String(path));  // 字符数组直接转字符串
            return;
        }

        String letters = MAPPING[digits[i] - '0'];  // 当前数字对应的字母集合
        for (char c : letters.toCharArray()) {
            path[i] = c;              // 直接覆盖，无需撤销
            dfs(i + 1, ans, path, digits);
        }
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n × 4ⁿ)** — 最坏每个数字对应 4 个字母，4ⁿ 种组合 |
| 🧠 空间复杂度 | **O(n)** — path 数组 + 递归栈 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 回溯（字符数组直接覆盖） |
| 算法类型 | 回溯、字符串 |
| 核心技巧 | **`path[i] = c` 直接覆盖 + 无需撤销——字符数组天然支持索引填值** |
| MAPPING 数组 | `digits[i] - '0'` 把字符转成下标直接查表 |
| 空串处理 | `n == 0` 时返回空列表（不是 `[""]`） |
| 关联题目 | [46. 全排列](day24-回溯hot100-part1.md)——同样的路径覆盖思想 |

### 一句话记住

> **「数字查表取字母，path[i] 直接覆盖——无需撤销。」**

---

## 题目：组合总和（Combination Sum）

**LeetCode 39 | 回溯 Hot100 | 难度：🟡 中等**

### 题目描述

给你一个**无重复元素**的整数数组 `candidates` 和一个目标整数 `target`，找出 `candidates` 中可以使数字和为目标数 `target` 的所有**不同组合**，并以列表形式返回。你可以按任意顺序返回这些组合。

`candidates` 中的**同一个数字可以无限制重复被选取**。如果至少一个数字的被选数量不同，则两种组合是不同的。

### 示例

```
输入：candidates = [2,3,6,7], target = 7
输出：[[2,2,3],[7]]

输入：candidates = [2,3,5], target = 8
输出：[[2,2,2,2],[2,3,3],[3,5]]

输入：candidates = [2], target = 1
输出：[]
```

---

## 解法：回溯（选/不选 + 可重复选取）

### 思路

子集问题的变体，但有两个关键区别：

1. **数字可以无限重复选取** → 选了 `candidates[i]` 后，`i` 不推进（下一层还可以选同一个）
2. **有目标和限制** → 用 `left`（剩余目标）控制剪枝

```
dfs(i, left)：正在决定 candidates[i] 是否选取，left 是剩余目标

  如果 left == 0 → path 总和正好是 target → 加入答案
  如果 left < 0 或 i == candidates.length → 剪枝返回

  不选 candidates[i]：dfs(i+1, left)
  选 candidates[i]：  path.add(candidates[i]) → dfs(i, left - candidates[i]) → path.removeLast()
                      ↑ 注意 i 不推进！因为数字可无限重复选取
```

**为什么选了之后 `i` 不推进？** 因为同一个数字可以无限重复选取，选了 `candidates[i]` 后，下一层仍然可以选择它。只有当"不选"时 `i` 才推进，保证组合不重复（不会出现 [2,3] 和 [3,2] 两种）。

### 思考方式图解

```
candidates = [2,3,6,7], target = 7

递归树（简化）：
           dfs(0, 7)
      不选2       选2 → dfs(0, 5)
     dfs(1,7)    /     \
            不选2     选2 → dfs(0, 3)
            dfs(1,5)  /    \
                  不选2   选2 → dfs(0,1)
                  dfs(1,3)  left<0 剪枝
                  /    \
                选3 → left=0 ✅ [2,2,3]

完整解：
  [2,2,3]（2×2 + 3 = 7）
  [7]（7 = 7）
```

### 代码实现

```java
class Solution {
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        List<List<Integer>> ans = new ArrayList<>();
        List<Integer> path = new ArrayList<>();

        dfs(0, target, candidates, ans, path);
        return ans;
    }

    // i 表示正在决定 candidates[i]，left 是剩余目标
    private void dfs(int i, int left, int[] candidates,
                     List<List<Integer>> ans, List<Integer> path) {
        if (left == 0) {                    // 找到一组解
            ans.add(new ArrayList<>(path));
            return;
        }
        if (left < 0 || i == candidates.length) {  // 剪枝
            return;
        }

        dfs(i + 1, left, candidates, ans, path);   // 不选 candidates[i]

        path.add(candidates[i]);                    // 选 candidates[i]
        dfs(i, left - candidates[i], candidates, ans, path);  // i 不推进，可重复选
        path.remove(path.size() - 1);               // 回溯
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(S)** — S 为所有可行组合的长度之和（严格上界 O(n × 2ⁿ) 级别，但剪枝后远小于） |
| 🧠 空间复杂度 | **O(target / min(candidates))** — 递归深度取决于能选多少个最小数 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 回溯（选/不选 + 可重复选取） |
| 算法类型 | 回溯 |
| 核心技巧 | **选了之后 `i` 不推进（可无限重复），不选时 `i` 推进（避免重复组合）；`left` 控制剪枝** |
| 与子集的区别 | 子集每个元素只能选一次（选了 i 推进）；组合总和可无限选（选了 i 不推进） |
| 剪枝条件 | `left < 0`（超了）或 `i == n`（元素用完了） |
| 关联题目 | [78. 子集](day24-回溯hot100-part1.md)——选/不选框架；[40. 组合总和 II](https://leetcode.cn/problems/combination-sum-ii/)——每个数字只能用一次 |

### 一句话记住

> **「选了 i 不推进、不选 i 推进——left 剪枝，可无限重复选。」**

---

*练习日期：2026-08-04*
