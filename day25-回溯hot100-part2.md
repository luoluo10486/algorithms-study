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

---

### 💡 另一种写法：for 循环 + 排序剪枝

用 **for 循环**代替「选/不选」分支，遍历从 `i` 开始的每个候选数字作为下一个元素：

```java
class Solution {
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        Arrays.sort(candidates);   // 排序是为了剪枝：candidates[j] > left 时直接 break
        List<List<Integer>> ans = new ArrayList<>();
        List<Integer> path = new ArrayList<>();

        dfs(0, target, candidates, ans, path);
        return ans;
    }

    // i 表示 path 中下一个位置只能从 candidates[i..] 中选（保证组合不重复）
    private void dfs(int i, int left, int[] candidates,
                     List<List<Integer>> ans, List<Integer> path) {
        if (left == 0) {                    // 找到一组解
            ans.add(new ArrayList<>(path));
            return;
        }

        // 遍历候选：j 从 i 开始（可重复选当前元素），且 candidates[j] <= left（剪枝）
        for (int j = i; j < candidates.length && candidates[j] <= left; j++) {
            path.add(candidates[j]);                          // 选 candidates[j]
            dfs(j, left - candidates[j], candidates, ans, path);  // j 不推进（可重复选）
            path.remove(path.size() - 1);                     // 回溯
        }
    }
}
```

**两种写法对比：**

| | 选/不选 二叉 | for 循环 + 排序剪枝 |
|---|------------|------------------|
| 分支结构 | 每个元素两个分支 | 遍历 i..n-1 每个元素一个分支 |
| 剪枝方式 | `left < 0` 或 `i == n` | **`candidates[j] <= left`（排序后提前 break）** |
| 需要排序 | 不需要 | 需要（剪枝依赖有序性） |
| 代码风格 | 递归式思维 | 迭代式思维（更常见） |

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
| 两种写法 | 选/不选二叉（无需排序） / for 循环 + 排序剪枝（`candidates[j] <= left` 提前 break） |
| 与子集的区别 | 子集每个元素只能选一次（选了 i 推进）；组合总和可无限选（选了 i 不推进） |
| 剪枝条件 | `left < 0`（超了）或 `i == n`（元素用完了） |
| 关联题目 | [78. 子集](day24-回溯hot100-part1.md)——选/不选框架；[40. 组合总和 II](https://leetcode.cn/problems/combination-sum-ii/)——每个数字只能用一次 |

### 一句话记住

> **「选了 i 不推进、不选 i 推进——left 剪枝，可无限重复选。」**

---

## 题目：括号生成（Generate Parentheses）

**LeetCode 22 | 回溯 Hot100 | 难度：🟡 中等**

### 题目描述

数字 `n` 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且**有效的**括号组合。

### 示例

```
输入：n = 3
输出：["((()))","(()())","(())()","()(())","()()()"]

输入：n = 1
输出：["()"]
```

---

## 解法：回溯（左右括号计数约束）

### 思路

生成 n 对有效括号，本质上是在 2n 个位置上填入 `(` 或 `)`，但必须满足两个约束：

```
约束 1：左括号最多放 n 个 → left < n 时才能放 '('
约束 2：右括号数量不能超过左括号 → right < left 时才能放 ')'
```

**为什么 `right < left` 才能放 `)`？** 如果右括号比左括号多，比如 `())`，右括号就没有对应的左括号匹配——非法。

```
dfs(left, right)：left/right 分别记录已放的左右括号数

  如果 right == n → 2n 个位置都填完了 → 加入答案

  如果 left < n：放 '('，dfs(left+1, right)
  如果 right < left：放 ')'，dfs(left, right+1)
```

**为什么用 `char[] path`？** 和 17 题一样，`path[left+right]` 按下标直接填字符，天然支持回溯，不需要撤销操作。

### 思考方式图解

```
n = 2

dfs(0,0):
  放'(' → dfs(1,0)
    放'(' → dfs(2,0)
      放')'(right<left) → dfs(2,1)
        放')' → dfs(2,2): right==n → 加入 "(())" ✅
    放')' → dfs(1,1)
      放'(' → dfs(2,1)
        放')' → dfs(2,2): 加入 "()()" ✅
      放')'？ right(1) < left(1) 不成立 → 不能放

结果：["(())", "()()"] ✅
```

### 代码实现

```java
class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> ans = new ArrayList<>();
        char[] path = new char[2 * n];

        dfs(0, 0, n, ans, path);
        return ans;
    }

    // left/right 分别表示已放置的左右括号数量
    private void dfs(int left, int right, int n, List<String> ans, char[] path) {
        if (right == n) {                  // 右括号放满 = 全部位置填完
            ans.add(new String(path));
            return;
        }

        if (left < n) {                    // 还能放左括号
            path[left + right] = '(';
            dfs(left + 1, right, n, ans, path);
        }

        if (right < left) {                // 右括号数 < 左括号数时才能放右括号
            path[left + right] = ')';
            dfs(left, right + 1, n, ans, path);
        }
    }
}
```

---

### 💡 另一种写法：记录左括号下标（灵茶山艾府）

核心思路反过来了——**不再逐个位置填括号，而是只记录 n 个左括号各自的下标，剩下的位置全是右括号**。

```
平衡条件：任意前缀中左括号数 ≥ 右括号数
等价于：第 k 个左括号的下标 ≥ 2k - 2（第 k 个左括号前至少要有 k-1 个右括号和它配对）

递归状态：
  i = 当前已填的括号总数
  balance = 已填括号中 左括号数 - 右括号数（≥ 0 恒成立）
  path 记录所有左括号的下标

每层递归：
  先填 right 个右括号（right 从 0 到 balance），再填 1 个左括号
  → 左括号的下标 = i + right
```

```java
class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> ans = new ArrayList<>();
        List<Integer> path = new ArrayList<>();   // 记录左括号的下标
        dfs(0, 0, n, path, ans);
        return ans;
    }

    // i = 已填括号总数；balance = 左括号数 - 右括号数
    private void dfs(int i, int balance, int n, List<Integer> path, List<String> ans) {
        if (path.size() == n) {                  // n 个左括号都放好了
            char[] s = new char[n * 2];
            Arrays.fill(s, ')');                 // 默认全是右括号
            for (int j : path) {
                s[j] = '(';                      // 把左括号下标处替换为 '('
            }
            ans.add(new String(s));
            return;
        }
        // 枚举填 right=0,1,...,balance 个右括号，再填 1 个左括号
        for (int right = 0; right <= balance; right++) {
            // 左括号下标 = i + right（前面 right 个右括号 + 当前位置）
            path.add(i + right);
            dfs(i + right + 1, balance - right + 1, n, path, ans);  // 填了 right+1 个括号
            path.removeLast();  // path.remove(path.size() - 1);
        }
    }
}
```

**两种写法对比：**

| | 逐个位置填括号 | 记录左括号下标（灵茶山） |
|---|-------------|----------------------|
| path 存什么 | 每个位置的字符 | **左括号的下标** |
| 每层放几个括号 | 1 个 | **right+1 个（批量）** |
| 生成答案 | 直接 new String(path) | 默认全 `)` 再替换为 `(` |
| 思维难度 | 直观 | 更抽象但更高效（分支更少） |

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(4ⁿ / √n)** — 卡特兰数 C(2n, n)/(n+1) 量级，每种答案拷贝 O(n) |
| 🧠 空间复杂度 | **O(n)** — path 数组 + 递归栈 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 回溯（括号计数约束） |
| 算法类型 | 回溯、字符串 |
| 核心技巧 | **`left < n` 才能放 '('；`right < left` 才能放 ')'——两个约束保证括号合法** |
| 两种写法 | 逐个位置填括号 / 记录左括号下标（批量填右括号，balance 保持平衡） |
| 为什么 right < left | 右括号数超过左括号时会出现无法匹配的 `)`，如 `())` |
| 用 char[] path | 下标 `left+right` 定位当前填充位置，直接覆盖，无需撤销 |
| 关联题目 | [17. 电话号码的字母组合](day25-回溯hot100-part2.md)——同样的 char[] path 覆盖思想 |

### 一句话记住

> **「左括号限总量，右括号不超左——约束填括号，right==n 即完成。」**

---

*练习日期：2026-08-04*
