# Day 32 — 多维动态规划 Hot100（Part 2）

## 题目：最小路径和（Minimum Path Sum）

**LeetCode 64 | 多维动态规划 Hot100 | 难度：🟡 中等**

### 题目描述

给定一个包含非负整数的 `m x n` 网格 `grid`，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。

**说明**：每次只能向下或者向右移动一步。

### 示例

```
输入：grid = [[1,3,1],[1,5,1],[4,2,1]]
输出：7
解释：因为路径 1→3→1→1→1 的总和最小。

输入：grid = [[1,2,3],[4,5,6]]
输出：12
```

---

## 解法：记忆化搜索（二维 DP）

### 思路

这是「不同路径」的**孪生题**——网格结构完全一样，只是把"路径条数相加"换成了"路径权重取最小"。

到达格子 `(i, j)` 的最小路径和 = **min(来自上方, 来自左方)** 再加上自己格子的权重。

```
dfs(i, j) = 从 (0,0) 到 (i,j) 的最小路径和

递推公式：
  dfs(i, j) = min( dfs(i-1, j), dfs(i, j-1) ) + grid[i][j]
              ↑ 从上边下来            ↑ 从左边过来      ↑ 自己的权重

边界：
  i < 0 或 j < 0 → Integer.MAX_VALUE（越界，不可走，不能选）
  i == 0 且 j == 0 → grid[0][0]（起点的最小和就是它自己）
```

**为什么越界返回 `Integer.MAX_VALUE`？** 因为我们用 `Math.min` 选最小路径：
- 在**顶行**（i==0 且 j>0）：上方 `dfs(-1, j)` 越界 = MAX_VALUE，左方 `dfs(0, j-1)` 有效。取 `min(MAX, 有效值)` 自然选到有效值，不会误选越界路径。
- 在**最左列**同理。

只要每个可达格子至少有一个邻居是"有效值"（而非 MAX_VALUE），`min` 就能正确规避越界方向。这正是本题能直接用 `Integer.MAX_VALUE` 当哨兵的原因（和 62 题用 `0` 当哨兵对称：`0` 对"求和"无害，`MAX_VALUE` 对"取最小"无害）。

### 思考方式图解

```
grid（示例）：
  1  3  1
  1  5  1
  4  2  1

各格子的最小路径和（正向理解）：
   1   4   5
   2   7   6
   6   8   7  ← (2,2) 最小和 = 7

递推：每个格子 = min(上方, 左边) + 自己
  (0,1) = min(MAX, 1) + 3 = 4
  (1,0) = min(1, MAX) + 1 = 2
  (2,2) = min(6, 8) + 1 = 7 ✅
```

### 代码实现

```java
class Solution {
    public int minPathSum(int[][] grid) {
        int m = grid.length;
        int n = grid[0].length;
        int[][] memo = new int[m][n];
        for (int[] row : memo) {
            Arrays.fill(row, -1);   // 用 -1 表示"未计算"
        }
        return dfs(m - 1, n - 1, grid, memo);
    }

    private int dfs(int i, int j, int[][] grid, int[][] memo) {
        if (i < 0 || j < 0) {           // 越界，不可走
            return Integer.MAX_VALUE;
        }
        if (i == 0 && j == 0) {         // 起点
            return grid[i][j];
        }
        if (memo[i][j] != -1) {         // 已算过
            return memo[i][j];
        }
        // 从上边下来、从左边过来，取较小者，再加自己的权重
        return memo[i][j] = Math.min(dfs(i - 1, j, grid, memo),
                                     dfs(i, j - 1, grid, memo)) + grid[i][j];
    }
}
```

> **注意**：`Math.min` 的两个参数里若有一个是 `Integer.MAX_VALUE`，会"溢出"成负数吗？不会——这里 `MAX_VALUE` 只是被比较的那个值，它本身不参与 `min` 之外的算术；真正的加法 `+ grid[i][j]` 只加在 `min` 的结果上，结果一定是有效值，不会溢出。

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(mn)** — 每个格子只算一次 |
| 🧠 空间复杂度 | **O(mn)** — memo 二维数组 + 递归栈 |

> **优化**：递推 DP 同样可压缩成一维数组（滚动），空间降到 **O(n)**。递推写法：`dp[j] = grid[i][j] + min(dp[j], dp[j-1])`，注意 `j==0` 时只能取上方 `dp[0]`。

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 记忆化搜索（二维 DP） |
| 算法类型 | 动态规划、二维 DP |
| 核心技巧 | **`dfs(i,j) = min(上方, 左边) + grid[i][j]`——二维网格 DP 取最小** |
| 边界设计 | 越界返回 `Integer.MAX_VALUE`（对取最小无害）；起点返回 `grid[0][0]` |
| 与 62 题的关系 | 同样网格 DP，62 求和用 `+`，本题取最小用 `min`；哨兵 0 ↔ MAX_VALUE 对称 |
| 优化方向 | 滚动数组 O(n) / 原地修改 grid 做到 O(1) 额外空间 |
| 关联题目 | [62. 不同路径](https://leetcode.cn/problems/unique-paths/)——孪生题，求路径条数 |

### 一句话记住

> **「最小路径和 = min(上, 左) + 自己——和不同路径同构，把相加换成取最小。」**

---

## 题目：最长回文子串（Longest Palindromic Substring）

**LeetCode 5 | 字符串 / 动态规划 Hot100 | 难度：🟡 中等**

> 本题虽常作为「二维 DP」入门（设 `dp[i][j]` 表示 `s[i..j]` 是否回文），但你给的这份代码用的是**中心扩展法**——相比 DP 的 O(n²) 空间，它只用 O(1) 额外空间，是更优的实战写法。下面按中心扩展来讲。

### 题目描述

给你一个字符串 `s`，找到 `s` 中最长的回文子串。

### 示例

```
输入：s = "babad"
输出："bab"（"aba" 同样符合题意）

输入：s = "cbbd"
输出："bb"
```

---

## 解法：中心扩展法（Center Expansion）

### 思路

回文串的精髓是「中心对称」。枚举**每一个可能的对称中心**，向两边同时扩张，直到两边字符不相等为止——扩张停止时的那段就是以此中心能形成的最长回文。

关键在于：回文中心分两种情况：

```
1) 奇数长度回文：中心是"一个字符"，如 "aba" 中心是 'b'
      l = i, r = i，向两侧扩

2) 偶数长度回文：中心是"两个字符之间"，如 "abba" 中心在 bb 之间
      l = i, r = i+1，向两侧扩
```

所以代码写了两个 `for` 循环：第一个处理所有奇数长度，第二个处理所有偶数长度。

### 为什么 `while` 结束后用 `r - l - 1` 算长度？

```
以 "babad" 中心 i=1（字符 'a'）为例：
   b a b a d
     ↑i
   l=1, r=1
   第1轮：st[1]==st[1] → l=0, r=2
   第2轮：st[0]=='b' == st[2]=='b' → l=-1, r=3
   第3轮：l<0 退出

   退出时 l=-1, r=3
   真实回文区间 [l+1, r-1] = [0, 2] = "bab"
   长度 = r - l - 1 = 3 - (-1) - 1 = 3  ✅
```

**核心技巧**：`l` 和 `r` 在退出循环时已经"多跨了一步"（指向第一个不相等的字符，或越界），所以真实回文是 `[l+1, r-1]`，长度为 `r - l - 1`。`ansLeft = l+1`、`ansRight = r` 直接对齐了 `String.substring(ansLeft, ansRight)` 的**左闭右开**语义，不用再 ±1，非常干净。

### 思考方式图解

```
s = "abbc"，n = 4

奇数中心（i = 0..3）：
  i=0 "a"      → 长度1
  i=1 "b"      → "b"（两边 a≠b）→ 长度1
  i=2 "b"      → "b"（两边 b,b 相等→扩成 "bbb"? 不，s[1]=b,s[3]=c 不等）
                 实际：l=2,r=2 → l=1,r=3 (b==b) → l=0,r=4(越界) 退出
                 [l+1,r-1]=[1,3]="bb" 长度2
  i=3 "c"      → 长度1

偶数中心（i = 0..2）：
  i=0 l=0,r=1 → a≠b 立即退出 → 长度0（不更新）
  i=1 l=1,r=2 → b==b → l=0,r=3(a≠c) 退出 → [1,2]="bb" 长度2
  i=2 l=2,r=3 → b≠c 退出 → 长度0

最终最长 = "bb"（长度2）✅
```

### 代码实现

```java
class Solution {
    public String longestPalindrome(String s) {
        char[] st = s.toCharArray();
        int n = st.length;
        int ansLeft = 0;
        int ansRight = 0;
        // 奇数长度：中心是单个字符
        for (int i = 0; i < n; i++) {
            int l = i;
            int r = i;
            while (l >= 0 && r < n && st[l] == st[r]) {
                l--;
                r++;
            }
            if (r - l - 1 > ansRight - ansLeft) {  // 发现更长的回文
                ansLeft = l + 1;
                ansRight = r;
            }
        }
        // 偶数长度：中心是两个字符之间
        for (int i = 0; i < n - 1; i++) {
            int l = i;
            int r = i + 1;
            while (l >= 0 && r < n && st[l] == st[r]) {
                l--;
                r++;
            }
            if (r - l - 1 > ansRight - ansLeft) {
                ansLeft = l + 1;
                ansRight = r;
            }
        }
        return s.substring(ansLeft, ansRight);  // 左闭右开，正好取 [l+1, r-1]
    }
}
```

> **对照 DP 写法**：二维 DP 设 `dp[i][j]` 表示 `s[i..j]` 是回文，转移 `dp[i][j] = s[i]==s[j] && dp[i+1][j-1]`。同样 O(n²) 时间，但空间 O(n²)；中心扩展把空间压到 O(1)，面试更受青睐。马拉车算法（Manacher）可做到 O(n)，但代码复杂，一般不用手写。

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n²)** — 每个中心扩最多 O(n)，共 O(n) 个中心 |
| 🧠 空间复杂度 | **O(1)** — 只用几个指针变量（不算输入） |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 中心扩展法（最长回文子串） |
| 算法类型 | 字符串、回文、二维 DP 的替代解法 |
| 核心技巧 | **枚举中心 + 两侧扩张；奇数中心 `l=r=i`，偶数中心 `l=i, r=i+1`** |
| 长度计算 | 退出循环后真实回文为 `[l+1, r-1]`，长度 `r - l - 1` |
| 取子串 | `ansLeft=l+1, ansRight=r` 正好对齐 `substring` 左闭右开语义 |
| 对比 DP | DP 空间 O(n²)；中心扩展 O(1)，更优 |
| 关联题目 | [647. 回文子串](https://leetcode.cn/problems/palindromic-substrings/)——统计回文个数，同一思路 |

### 一句话记住

> **「最长回文 = 枚举每个中心向两边扩——奇数单心、偶数双心，扩完用 `r-l-1` 量长度。」**

---

## 题目：最长公共子序列（Longest Common Subsequence）

**LeetCode 1143 | 动态规划 Hot100 | 难度：🟡 中等**

> 这是**二维动态规划**的标杆题，也是理解「LCS / 编辑距离」一族问题的基石。和前面 64（最小路径和）同属"二维格子递推"，但转移逻辑更典型——**字符相等就对角 +1，不等就取上方/左方的最大值**。

### 题目描述

给定两个字符串 `text1` 和 `text2`，返回这两个字符串的**最长公共子序列**的长度。

**子序列**：由原字符串在不改变字符相对顺序的情况下删除某些字符（也可以不删）后形成的新字符串（**不要求连续**）。

### 示例

```
输入：text1 = "abcde", text2 = "ace"
输出：3
解释：最长公共子序列是 "ace"，长度为 3。

输入：text1 = "abc", text2 = "abc"
输出：3

输入：text1 = "abc", text2 = "def"
输出：0
```

---

## 解法：记忆化搜索（二维 DP）

### 思路

定义 `dfs(i, j)` = 考虑 `s` 的前 `i+1` 个字符（下标 `0..i`）和 `t` 的前 `j+1` 个字符（下标 `0..j`）时，它们的 LCS 长度。

```
dfs(i, j) = LCS(s[0..i], t[0..j]) 的长度

两种情形：
  ① s[i] == t[j]：这两个字符可以"配对"进公共子序列
     → dfs(i, j) = dfs(i-1, j-1) + 1        （对角左上 + 1）

  ② s[i] != t[j]：至少有一个字符不能进序列，二选一抛弃
     → dfs(i, j) = max( dfs(i-1, j),        （抛弃 s[i]，t 不变）
                        dfs(i, j-1) )        （抛弃 t[j]，s 不变）

边界：
  i < 0 或 j < 0 → 0（有一方已经空了，公共子序列只能是空）
```

**为什么"不等"时要取 max 而不是只走一条？** 因为不知道抛弃 `s[i]` 还是抛弃 `t[j]` 能得到更长的结果，两条路都得试，取大的。这正是二维 DP "状态来自两个方向" 的体现（和 64 题取 `min(上, 左)` 结构一模一样，只是这里是 `max`）。

### 思考方式图解

```
s = "a b c d e"   (i 行)
t = "a c e"       (j 列)

构造 dp[i][j]（i 为 s 下标，j 为 t 下标）：

      t:  ∅   a   c   e
   s:        -1  -1  -1  (虚拟空列，值0)
   ∅  -1:    0   0   0   0
   a   0 :   0   1   1   1     (s[0]=a 配对 t[0]=a → 1；之后 max 维持1)
   b   1 :   0   1   1   1
   c   2 :   0   1   2   2     (s[2]=c 配对 t[1]=c → dp[1][0]+1=2)
   d   3 :   0   1   2   2
   e   4 :   0   1   2   3  ← (4,2): s[4]=e 配对 t[2]=e → dp[3][1]+1=2+1=3 ✅

最终答案 dp[n-1][m-1] = 3（"ace"）
```

### 代码实现

```java
class Solution {
    public int longestCommonSubsequence(String text1, String text2) {
        char[] s = text1.toCharArray();
        char[] t = text2.toCharArray();
        int n = s.length;
        int m = t.length;
        int[][] memo = new int[n][m];
        for (int[] row : memo) {
            Arrays.fill(row, -1);   // -1 表示"未计算"
        }
        return dfs(n - 1, m - 1, s, t, memo);
    }

    private int dfs(int i, int j, char[] s, char[] t, int[][] memo) {
        if (i < 0 || j < 0) {            // 有一方空了
            return 0;
        }
        if (memo[i][j] != -1) {          // 已算过
            return memo[i][j];
        }
        if (s[i] == t[j]) {              // 字符相等 → 对角 +1
            return memo[i][j] = dfs(i - 1, j - 1, s, t, memo) + 1;
        }
        // 不相等 → 抛弃 s[i] 或 t[j]，取较大者
        return memo[i][j] = Math.max(dfs(i - 1, j, s, t, memo),
                                     dfs(i, j - 1, s, t, memo));
    }
}
```

> **递推 DP 写法**（空间 O(mn)，可滚动压成 O(min(m,n))）：
> `dp[i][j] = s[i]==t[j] ? dp[i-1][j-1]+1 : max(dp[i-1][j], dp[i][j-1])`，初始化第 0 行/第 0 列为 0。

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(nm)** — 每个 `(i,j)` 状态只算一次 |
| 🧠 空间复杂度 | **O(nm)** — memo 二维数组 + 递归栈（可压成 O(min(n,m))） |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 记忆化搜索（二维 DP） |
| 算法类型 | 动态规划、二维 DP、字符串 |
| 核心技巧 | **`s[i]==t[j] ? 对角+1 : max(上, 左)`——二维 DP 配对问题的标准转移** |
| 边界 | 任一为空返回 0 |
| 与 64 题的关系 | 同样二维格子递推；64 是 `min(上,左)+权重`，本题是 `max(上, 左)`（不等时）/ `对角+1`（相等时） |
| 关键区别 | 「子序列」不要求连续，所以不等时仍保留 `max(上, 左)` 而非直接归零 |
| 关联题目 | [72. 编辑距离](https://leetcode.cn/problems/edit-distance/)（增删改三操作）、[583. 两个字符串的删除操作](https://leetcode.cn/problems/delete-operation-for-two-strings/)（同族） |

### 一句话记住

> **「LCS = 相等对角 +1，不等取 max(上, 左)——二维 DP 配对问题的标准模板。」**

---

## 题目：编辑距离（Edit Distance）

**LeetCode 72 | 动态规划 Hot100 | 难度：🔴 困难**

> 这是二维 DP 的"封神题"——把 LCS（1143）的"配对我不管、不相等就 max"升级成"不相等可以删/插/改，取三者最小"。理解了它，整族字符串 DP 就通了。

### 题目描述

给你两个字符串 `word1` 和 `word2`，请你计算出将 `word1` 转换成 `word2` 所使用的最少操作数。

可对 `word1` 进行三种操作（每个操作算 1 步）：
- **插入** 一个字符
- **删除** 一个字符
- **替换** 一个字符

### 示例

```
输入：word1 = "horse", word2 = "ros"
输出：3
解释：horse → rorse（替换 h→r）→ rose（删除 r）→ ros（替换 e→s），共 3 步。

输入：word1 = "intention", word2 = "execution"
输出：5
```

---

## 解法：记忆化搜索（二维 DP）

### 思路

定义 `dfs(i, j)` = 把 `s` 的前 `i+1` 个字符（`0..i`）变成 `t` 的前 `j+1` 个字符（`0..j`）所需的最少操作数。

```
dfs(i, j) = 将 s[0..i] 转成 t[0..j] 的最小操作数

情形：
  ① s[i] == t[j]：最后一个字符已经相同，不用操作
     → dfs(i, j) = dfs(i-1, j-1)              （直接对齐，0 步）

  ② s[i] != t[j]：从三种操作里选最省的一步（+1 步）
     → min(
           dfs(i-1, j),    删除 s[i]：s 少一个，t 不变
           dfs(i, j-1),    插入 t[j]：等价于删 t[j]，s 不变
           dfs(i-1, j-1)   替换 s[i]→t[j]：两个都往前推一步
       ) + 1

边界（有一方已经处理完）：
  i < 0 → 剩 j+1 个 t 字符没凑出来，全插入 → 返回 j+1
  j < 0 → 剩 i+1 个 s 字符没消掉，全删除 → 返回 i+1
```

**和 LCS 的对比（同一张二维表，两种灵魂）：**

| | LCS (1143) | 编辑距离 (72) |
|---|---|---|
| 相等时 | `对角 + 1` | `对角 + 0`（白送） |
| 不相等时 | `max(上, 左)`（保留两个方向） | `min(上, 左, 对角) + 1`（三种操作取最优） |
| 空串边界 | 返回 `0` | 返回 `i+1` / `j+1`（剩余全删/全插） |

### 思考方式图解

```
s = "horse", t = "ros"

dp[i][j]（i 行 s，j 列 t）递推示意（部分）：

        ∅   r   o   s
   ∅    0   1   2   3
   h    1   1   2   3      (h≠r 但删 h 后对齐 r→1)
   o    2   2   1   2      (o==o 对角 2→直接 1)
   r    3   2   2   2
   s    4   3   3   2  ← (4,2): s==s 对角 (3,1)=3 → 取 2? 实际走 min 得 2 ✅
   e    5   4   4   3  ← 最终 dfs(4,2)=3

答案 = 3（horse→ros）
```

> 注意 `s[i]==t[j]` 时直接 `dfs(i-1,j-1)` 不加 1——因为这一对字符"白送"，不消耗操作数。这是 Edit Distance 与 LCS 在相等分支上的唯一区别。

### 代码实现

```java
class Solution {
    public int minDistance(String word1, String word2) {
        char[] s = word1.toCharArray();
        char[] t = word2.toCharArray();
        int n = s.length;
        int m = t.length;
        int[][] memo = new int[n][m];
        for (int[] row : memo) {
            Arrays.fill(row, -1);   // -1 表示"未计算"
        }
        return dfs(n - 1, m - 1, s, t, memo);
    }

    private int dfs(int i, int j, char[] s, char[] t, int[][] memo) {
        if (i < 0) {                    // s 已空，剩下 t 全插入
            return j + 1;
        }
        if (j < 0) {                    // t 已空，剩下 s 全删除
            return i + 1;
        }
        if (memo[i][j] != -1) {         // 已算过
            return memo[i][j];
        }
        if (s[i] == t[j]) {             // 字符相同，白送，不耗操作
            return dfs(i - 1, j - 1, s, t, memo);
        }
        // 不等：删 / 插 / 改 三种操作取最小，各 +1 步
        return memo[i][j] = Math.min(Math.min(
                dfs(i - 1, j, s, t, memo),       // 删除 s[i]
                dfs(i, j - 1, s, t, memo)),      // 插入 t[j]
                dfs(i - 1, j - 1, s, t, memo))   // 替换 s[i]→t[j]
                + 1;
    }
}
```

> **边界安全说明**：`memo` 大小为 `[n][m]`，只在 `i>=0 && j>=0` 时才访问 `memo[i][j]`；而递归进入 `i<0` 或 `j<0` 的分支会先命中 `if (i<0)` / `if (j<0)` 提前返回，**不会越界访问 memo**。这正是把"空串边界"写成 early return 的原因。

> **递推 DP 写法**（空间 O(mn)，可滚动压成 O(min(m,n))）：
> `dp[i][j] = s[i]==t[j] ? dp[i-1][j-1] : min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + 1`，第 0 行/列初始化为 `j` / `i`。

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(nm)** — 每个 `(i,j)` 状态只算一次 |
| 🧠 空间复杂度 | **O(nm)** — memo 二维数组 + 递归栈（可压成 O(min(n,m))） |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 记忆化搜索（二维 DP） |
| 算法类型 | 动态规划、二维 DP、字符串、困难 |
| 核心技巧 | **相等对角+0；不等取 `min(删, 插, 改)+1`；空串边界返回 `i+1`/`j+1`** |
| 边界 | `i<0 → j+1`（全插入）；`j<0 → i+1`（全删除） |
| 与 1143 的关系 | 同一张二维表；LCS 不相等取 `max`，本题取 `min` 且三种操作各 +1；相等时 LCS 对角+1、本题对角+0 |
| 关键理解 | `s[i]==t[j]` 时直接 `dfs(i-1,j-1)` 不加 1——字符"白送" |
| 关联题目 | [583. 两个字符串的删除操作](https://leetcode.cn/problems/delete-operation-for-two-strings/)（只允许删）、[712. 两个字符串的最小 ASCII 删除和](https://leetcode.cn/problems/minimum-ascii-delete-sum-for-two-strings/)（加权版） |

### 一句话记住

> **「编辑距离 = 相等对角白送，不等取 min(删,插,改)+1；空串边界全删/全插——LCS 的升级版。」**

---

*练习日期：2026-08-15*
