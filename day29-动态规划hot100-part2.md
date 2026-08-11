# Day 29 — 动态规划 Hot100（Part 2）

## 题目：完全平方数（Perfect Squares）

**LeetCode 279 | 动态规划 Hot100 | 难度：🟡 中等**

### 题目描述

给你一个整数 `n`，返回**和为 `n` 的完全平方数的最少数量**。

完全平方数是一个整数，其值等于另一个整数的平方；换句话说，其值等于一个整数自乘的积。例如，`1`、`4`、`9` 和 `16` 都是完全平方数，而 `3` 和 `11` 不是。

### 示例

```
输入：n = 12
输出：3
解释：12 = 4 + 4 + 4

输入：n = 13
输出：2
解释：13 = 4 + 9
```

---

## 解法：记忆化搜索（背包问题思路）

### 思路

这本质是一个**完全背包问题**：物品是完全平方数 `1, 4, 9, ...`（`i²`），每个物品**可以无限取**，目标是用最少数量的物品凑出总和 `n`。

```
dfs(i, j)：用前 i 个完全平方数（1², 2², ..., i²）凑出总和 j 的最少数量

边界：
  i == 0 → 没有数可用：j==0 时用 0 个，否则不可能（返回 INF）

决策（每个 i² 选或不选，可选多次）：
  如果 j < i² → 放不下 i²，只能不选 → dfs(i-1, j)
  否则 → min(不选, 选)：
    不选：dfs(i-1, j)
    选：  dfs(i, j - i²) + 1   ← 注意 i 不变（可重复选），j 减少 i²
```

**为什么 i 可以重复选？** 完全平方数可以重复使用（如 12 = 4+4+4，用了 3 次 4），所以选了 `i²` 后 `i` 不推进——和组合总和（39）的可重复选取是同一个思想。

### 思考方式图解

```
n = 12, i 的范围 = 1..√12 = 3（可用 1², 2², 3²）

dfs(3, 12) = min(dfs(2, 12), dfs(3, 12-9)+1 = dfs(3,3)+1)
  dfs(2, 12) = min(dfs(1, 12), dfs(2, 12-4)+1 = dfs(2,8)+1)
    dfs(1, 12) = dfs(1, 12-1)+1 → dfs(1,11)+1 → ... = 12（全是 1²）
    dfs(2, 8)  = min(dfs(1,8), dfs(2,4)+1)
      dfs(1,8) = 8
      dfs(2,4) = min(dfs(1,4), dfs(2,0)+1) = min(4, 0+1) = 1
      → dfs(2,8) = min(8, 1+1) = 2  ✅ (4+4)
    → dfs(2,12) = min(12, 2+1) = 3
  dfs(3, 3)  = min(dfs(2,3), dfs(3, 3-9<0)→只不选) = dfs(2,3)
    dfs(2,3) = min(dfs(1,3), dfs(2, 3-4<0)) = dfs(1,3) = 3
    → dfs(3,3) = 3

结果：min(3, 3+1=4) = 3 ✅（12 = 4+4+4）
```

### 代码实现

```java
class Solution {
    // memo[i][j]：用前 i 个完全平方数凑出 j 的最少数量（-1 表示未计算）
    private static final int[][] memo = new int[101][10001];
    static {
        for (int[] row : memo) {
            Arrays.fill(row, -1);
        }
    }

    private static int dfs(int i, int j) {
        if (i == 0) {                       // 没有数可用
            return j == 0 ? 0 : Integer.MAX_VALUE;  // j==0 用 0 个，否则不可能
        }
        if (memo[i][j] != -1) {             // 已算过
            return memo[i][j];
        }

        if (j < i * i) {                    // 放不下 i²，只能不选
            return memo[i][j] = dfs(i - 1, j);
        }

        // 选/不选取最小：不选 dfs(i-1,j)；选 dfs(i, j-i²)+1（i 不变可重复选）
        return memo[i][j] = Math.min(dfs(i - 1, j), dfs(i, j - i * i) + 1);
    }

    public int numSquares(int n) {
        return dfs((int) Math.sqrt(n), n);  // i 从 √n 开始（最大可用平方数）
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(√n × n)** — i 有 √n 种、j 有 n 种，每个状态算一次 |
| 🧠 空间复杂度 | **O(√n × n)** — 静态 memo 数组（所有测试用例复用） |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 记忆化搜索（完全背包） |
| 算法类型 | 动态规划、背包 |
| 核心技巧 | **完全平方数是"可无限取"的物品，`dfs(i, j-i²)+1` 中 i 不变实现重复选** |
| 边界处理 | `i==0` 时只有 `j==0` 可行（返回 0），其余返回 INF 表示不可能 |
| 为什么用静态 memo | 静态数组跨测试用例复用，避免重复初始化——LeetCode 上可显著提速 |
| 剪枝 | `j < i*i` 时放不下 i²，直接走"不选"分支 |
| 关联题目 | [39. 组合总和](day25-回溯hot100-part2.md)——同样的可重复选取思想 |

### 一句话记住

> **「完全平方数是背包物品，可无限取——选 i 不推进，凑 j 取最小。」**

---

## 题目：零钱兑换（Coin Change）

**LeetCode 322 | 动态规划 Hot100 | 难度：🟡 中等**

### 题目描述

给你一个整数数组 `coins`，表示不同面额的硬币；以及一个整数 `amount`，表示总金额。

计算并返回可以凑成总金额所需的**最少的硬币个数**。如果没有任何一种硬币组合能组成总金额，返回 `-1`。

你可以认为每种硬币的数量是**无限的**。

### 示例

```
输入：coins = [1, 2, 5], amount = 11
输出：3
解释：11 = 5 + 5 + 1

输入：coins = [2], amount = 3
输出：-1

输入：coins = [1], amount = 0
输出：0
```

---

## 解法：记忆化搜索（完全背包）

### 思路

和 279 完全平方数是**同一个模板**——完全背包：硬币是"可无限取"的物品，求凑出 `amount` 的最少数量。

```
dfs(i, c)：用前 i+1 种硬币（coins[0..i]）凑出金额 c 的最少数量

边界：
  i < 0 → 没有硬币可用：c==0 时用 0 个，否则不可能（返回 INF/2）

决策（第 i 种硬币选或不选，可重复选）：
  如果 c < coins[i] → 放不下这种硬币，只能不选 → dfs(i-1, c)
  否则 → min(不选, 选)：
    不选：dfs(i-1, c)
    选：  dfs(i, c - coins[i]) + 1   ← i 不变（可重复选）
```

### 思考方式图解

```
coins = [1, 2, 5], amount = 11

dfs(2, 11) = min(dfs(1, 11), dfs(2, 6)+1)
  dfs(1, 11) = min(dfs(0, 11), dfs(1, 9)+1)
    dfs(0, 11) = 11（全用 1 元）
    dfs(1, 9) = min(dfs(0,9), dfs(1,7)+1) ... 最终 dfs(1,11) = min(11, 5+1=6)? 
    → 实际递归过程省略，最后 dfs(1,11)=6（5+2+2+2? 不对，只用前两种 [1,2]）
  dfs(2, 6) = min(dfs(1,6), dfs(2,1)+1)
    dfs(1,6) = 3（2+2+2）
    dfs(2,1) = min(dfs(1,1), 放不下5) = 1（1 元）
    → dfs(2,6) = min(3, 1+1) = 2 ✅（5+1）
  → dfs(2,11) = min(6, 2+1) = 3 ✅（5+5+1）

结果：3 ✅
```

### 代码实现

```java
class Solution {
    public int coinChange(int[] coins, int amount) {
        int n = coins.length;
        int[][] memo = new int[n][amount + 1];
        for (int[] row : memo) {
            Arrays.fill(row, -1);
        }

        int ans = dfs(n - 1, amount, coins, memo);
        return ans < Integer.MAX_VALUE / 2 ? ans : -1;  // INF/2 表示凑不出来
    }

    // dfs(i, c)：用 coins[0..i] 凑出金额 c 的最少数量
    private int dfs(int i, int c, int[] coins, int[][] memo) {
        if (i < 0) {                            // 没有硬币可用
            return c == 0 ? 0 : Integer.MAX_VALUE / 2;  // 用 INF/2 防止 +1 溢出
        }
        if (memo[i][c] != -1) {                 // 已算过
            return memo[i][c];
        }

        if (c < coins[i]) {                     // 放不下这种硬币，只能不选
            return memo[i][c] = dfs(i - 1, c, coins, memo);
        }

        // 不选 dfs(i-1,c)；选 dfs(i, c-coins[i])+1（i 不变可重复选）
        return memo[i][c] = Math.min(dfs(i - 1, c, coins, memo),
                                     dfs(i, c - coins[i], coins, memo) + 1);
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n × amount)** — n 种硬币 × amount 个金额，每个状态算一次 |
| 🧠 空间复杂度 | **O(n × amount)** — memo 二维数组 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 记忆化搜索（完全背包） |
| 算法类型 | 动态规划、背包 |
| 核心技巧 | **硬币可无限取，`dfs(i, c-coins[i])+1` 中 i 不变；与 279 完全平方数同一模板** |
| 为什么用 INF/2 | `Integer.MAX_VALUE/2` 作为"不可能"标记——`+1` 不会溢出成负数 |
| 返回 -1 的判断 | `ans < INF/2` 才是真正凑得出来，否则返回 -1 |
| 与 279 的区别 | 279 物品是自动生成的平方数；322 物品是给定的 coins 数组，且多了"凑不出返回 -1" |
| 关联题目 | [279. 完全平方数](day29-动态规划hot100-part2.md)——完全背包同款模板 |

### 一句话记住

> **「硬币无限取，凑 amount 取最少——INF/2 防溢出，凑不出返回 -1。」**

---

## 题目：单词拆分（Word Break）

**LeetCode 139 | 动态规划 Hot100 | 难度：🟡 中等**

### 题目描述

给你一个字符串 `s` 和一个字符串列表 `wordDict` 作为字典。如果可以利用字典中出现的一个或多个单词拼接出 `s` 则返回 `true`。

**注意**：不要求字典中出现的单词全部都使用，并且字典中的单词可以重复使用。

### 示例

```
输入：s = "leetcode", wordDict = ["leet", "code"]
输出：true
解释：s 可以由 "leet" + "code" 拼接成

输入：s = "applepenapple", wordDict = ["apple", "pen"]
输出：true

输入：s = "catsandog", wordDict = ["cats","dog","sand","and","cat"]
输出：false
```

---

## 解法：记忆化搜索（后缀拆分 + 剪枝）

### 思路

核心思想：**判断 `s[0..i]` 能否被拆分，就看是否存在一个 `j < i`，使得 `s[j..i]` 是字典中的单词，且 `s[0..j]` 也能被拆分**。

```
dfs(i)：s 的前 i 个字符（s[0..i)）能否被字典中的单词拼接

  如果 i == 0 → 空串可以被拼接（返回 1）
  枚举 j 从 i-1 往前（尝试最后一段 s[j..i)）：
    如果 s[j..i) 在字典中 且 dfs(j) == 1 → 返回 1
  都试过不行 → 返回 0
```

**maxLen 剪枝**：最后一段的长度不可能超过字典中最长单词的长度。所以 `j` 从 `i-1` 往前最多枚举 `maxLen` 个位置——把 O(i) 的枚举压缩到 O(maxLen)。

### 思考方式图解

```
s = "leetcode", wordDict = ["leet", "code"], maxLen = 4

dfs(8)：尝试最后一段
  j=7: "e" 不在字典 → 继续
  j=6: "de" 不在字典 → 继续
  j=5: "ode" 不在字典 → 继续
  j=4: "code" 在字典 ✅ 且 dfs(4)?
    dfs(4)：尝试最后一段
      j=3: "t" 不在字典
      j=2: "et" 不在字典
      j=1: "eet" 不在字典
      j=0: "leet" 在字典 ✅ 且 dfs(0)=1 → 返回 1
    → dfs(4)=1
  → dfs(8)=1 ✅

结果：true
```

### 代码实现

```java
class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        int maxLen = 0;
        for (String word : wordDict) {
            maxLen = Math.max(maxLen, word.length());  // 字典中最长单词长度
        }
        Set<String> words = new HashSet<>(wordDict);   // 哈希集合 O(1) 查字典
        int n = s.length();
        int[] memo = new int[n + 1];
        Arrays.fill(memo, -1);

        return dfs(n, maxLen, s, words, memo) == 1;
    }

    // dfs(i)：s 的前 i 个字符能否被拆分（1 能 / 0 不能）
    private int dfs(int i, int maxLen, String s, Set<String> words, int[] memo) {
        if (i == 0) {              // 空串可拆分
            return 1;
        }
        if (memo[i] != -1) {       // 已算过
            return memo[i];
        }

        // 枚举最后一段 s[j..i)，长度不超过 maxLen
        for (int j = i - 1; j >= Math.max(i - maxLen, 0); j--) {
            if (words.contains(s.substring(j, i))   // 最后一段在字典中
                && dfs(j, maxLen, s, words, memo) == 1) {  // 前半段也能拆分
                return memo[i] = 1;
            }
        }
        return memo[i] = 0;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n × maxLen)** — 每个位置最多枚举 maxLen 个切分点，substring 也要 O(长度) |
| 🧠 空间复杂度 | **O(n)** — memo 数组 + 递归栈 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 记忆化搜索（后缀拆分） |
| 算法类型 | 动态规划、字符串 |
| 核心技巧 | **`dfs(i) = ∃j: s[j..i) ∈ 字典 且 dfs(j)`——枚举最后一段递归判断前半段** |
| maxLen 剪枝 | 最后一段长度 ≤ 字典最长单词，枚举范围从 O(i) 压到 O(maxLen) |
| 为什么用 Set | `words.contains` O(1) 查字典，比 List 的 O(n) 快 |
| 用 int 而非 boolean memo | -1 未算 / 0 不能 / 1 能——boolean 无法区分"未算"和"false" |
| 关联题目 | [322. 零钱兑换](day29-动态规划hot100-part2.md)——同样的一维记忆化框架 |

### 一句话记住

> **「枚举最后一段，查字典 + 递归前半——maxLen 剪枝，memo 防重算。」**

---

*练习日期：2026-08-11*
