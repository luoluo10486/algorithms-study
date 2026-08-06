# Day 26 — 回溯 Hot100（Part 3）

## 题目：单词搜索（Word Search）

**LeetCode 79 | 回溯 Hot100 | 难度：🟡 中等**

### 题目描述

给定一个 `m x n` 二维字符网格 `board` 和一个字符串单词 `word`。如果 `word` 存在于网格中，返回 `true`；否则，返回 `false`。

单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中"相邻"单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

### 示例

```
输入：board = [["A","B","C","E"],
              ["S","F","C","S"],
              ["A","D","E","E"]], word = "ABCCED"
输出：true

输入：board = [["A","B","C","E"],
              ["S","F","C","S"],
              ["A","D","E","E"]], word = "SEE"
输出：true

输入：board = [["A","B","C","E"],
              ["S","F","C","S"],
              ["A","D","E","E"]], word = "ABCB"
输出：false
```

---

## 解法：DFS 回溯（网格搜索）

### 思路

从每个格子出发，尝试用 DFS 匹配整个单词。核心要点：

1. **当前格子字符不匹配 → 直接 false**
2. **匹配到最后一个字符 → 找到完整单词，返回 true**
3. **标记当前格子为已访问**（`board[i][j] = 0`），防止路径上重复使用
4. **向四个方向 DFS 搜索下一个字符**
5. **回溯恢复**（`board[i][j] = word[k]`）

```
dfs(i, j, k)：在 (i,j) 处匹配 word 的第 k 个字符

  如果 board[i][j] != word[k] → false
  如果 k == word.length - 1 → 匹配完成 → true
  标记 board[i][j] = 0（已访问）
  向上下左右四个方向递归 dfs(x, y, k+1)
  回溯恢复 board[i][j] = word[k]
```

**为什么用 `board[i][j] = 0` 标记？** 网格中字符都是字母（非 0），用 0 标记已访问，无需额外的 visited 数组，省空间且判断 `board[i][j] != word[k]` 时天然拦截已访问格子。

### 思考方式图解

```
board = [["A","B","C","E"],
         ["S","F","C","S"],
         ["A","D","E","E"]], word = "ABCCED"

起点 (0,0)='A' ✅
  → (0,1)='B' ✅
    → (1,1)='F' ❌ (word[2]='C')
    → (0,2)='C' ✅
      → (0,3)='E' ❌ (word[3]='C')
      → (1,2)='C' ✅
        → (1,1)='F' ❌ (word[4]='E')
        → (1,3)='S' ❌
        → (2,2)='E' ✅
          → (2,1)='D' ✅ (word[6]='D')
            → 找到完整单词！返回 true ✅
```

### 代码实现

```java
class Solution {
    private static final int[][] dirs = {{0,-1},{0,1},{-1,0},{1,0}}; // 上下左右

    public boolean exist(char[][] board, String word) {
        char[] w = word.toCharArray();

        // 从每个格子出发尝试匹配
        for (int i = 0; i < board.length; i++) {
            for (int j = 0; j < board[0].length; j++) {
                if (dfs(i, j, 0, board, w)) {
                    return true;
                }
            }
        }
        return false;
    }

    // 在 (i,j) 匹配 word 的第 k 个字符
    private boolean dfs(int i, int j, int k, char[][] board, char[] word) {
        if (board[i][j] != word[k]) {       // 当前字符不匹配
            return false;
        }
        if (k == word.length - 1) {         // 最后一个字符匹配成功
            return true;
        }

        board[i][j] = 0;                    // 标记已访问（防止重复使用）

        for (int[] d : dirs) {
            int x = i + d[0];
            int y = j + d[1];
            if (0 <= x && x < board.length
                && 0 <= y && y < board[x].length
                && dfs(x, y, k + 1, board, word)) {
                return true;
            }
        }

        board[i][j] = word[k];              // 回溯：恢复现场
        return false;
    }
}
```

---

### 💡 灵茶山艾府的两个优化

基础版的 DFS 在主循环前没有做任何预处理。灵茶山艾府加了**两个优化**，能把大量无效情况提前排除，大幅提升通过率：

#### 优化一：字符计数预检查

如果 `word` 中某个字符的出现次数比 `board` 中该字符的总数还多，那必然搜不到——**直接返回 false**，省去整个 DFS：

```java
// 统计 board 中每个字符的出现次数
int[] cnt = new int[128];
for (char[] row : board) {
    for (char c : row) {
        cnt[c]++;
    }
}

// 如果 word 中某字符比 board 里还多 → 肯定搜不到
char[] w = word.toCharArray();
int[] wordCnt = new int[128];
for (char c : w) {
    if (++wordCnt[c] > cnt[c]) {
        return false;   // 提前退出
    }
}
```

#### 优化二：从更稀有的字符开始搜（必要时反转 word）

DFS 的搜索量取决于**起点字符的稀有程度**——起点越稀有，起点越少，搜索分支越少。

```
如果 word 的最后一个字符在 board 中出现次数 < 第一个字符：
  → 反转 word，从原本的末尾字符开始搜
```

比如 `word = "abcd"`，如果 `board` 中 `d` 比 `a` 少，反转成 `"dcba"` 后从 `d` 开始搜——起点少了，整个搜索树都变小：

```java
// 如果末尾字符比首字符更稀有，反转 word（从更稀有的字符开始搜）
if (cnt[w[w.length - 1]] < cnt[w[0]]) {
    w = new StringBuilder(word).reverse().toString().toCharArray();
}
```

**为什么有效？** DFS 的每个起点都要展开一棵搜索树，起点数量直接乘在总复杂度里（O(起点数 × 4^L)）。从稀有字符开始，起点数大幅下降。

**优化后的完整代码：**

```java
class Solution {
    private static final int[][] DIRS = {{0, -1}, {0, 1}, {-1, 0}, {1, 0}};

    public boolean exist(char[][] board, String word) {
        // 用数组代替哈希表统计字符出现次数
        int[] cnt = new int[128];
        for (char[] row : board) {
            for (char c : row) {
                cnt[c]++;
            }
        }

        // 优化一：word 中某字符比 board 里还多 → 直接返回 false
        char[] w = word.toCharArray();
        int[] wordCnt = new int[128];
        for (char c : w) {
            if (++wordCnt[c] > cnt[c]) {
                return false;
            }
        }

        // 优化二：末尾字符更稀有 → 反转 word 从它开始搜
        if (cnt[w[w.length - 1]] < cnt[w[0]]) {
            w = new StringBuilder(word).reverse().toString().toCharArray();
        }

        for (int i = 0; i < board.length; i++) {
            for (int j = 0; j < board[i].length; j++) {
                if (dfs(i, j, 0, board, w)) {
                    return true;
                }
            }
        }
        return false;
    }

    private boolean dfs(int i, int j, int k, char[][] board, char[] word) {
        if (board[i][j] != word[k]) return false;   // 匹配失败
        if (k == word.length - 1) return true;      // 匹配成功

        board[i][j] = 0;  // 标记访问过
        for (int[] d : DIRS) {
            int x = i + d[0];
            int y = j + d[1];
            if (0 <= x && x < board.length
                && 0 <= y && y < board[x].length
                && dfs(x, y, k + 1, board, word)) {
                return true;
            }
        }
        board[i][j] = word[k];  // 恢复现场
        return false;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(mn × 4^L)** — 最坏不变，但两个优化在平均情况下大幅剪枝 |
| 🧠 空间复杂度 | **O(L)** — 递归栈深度 = 单词长度 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | DFS 回溯（网格搜索）+ 预处理优化 |
| 算法类型 | 回溯、DFS |
| 核心技巧 | **字符不匹配即返回；匹配到末尾即成功；`board[i][j]=0` 标记已访问，回溯时恢复** |
| 优化一 | 字符计数预检查——word 中某字符比 board 多 → 直接 false，跳过整个 DFS |
| 优化二 | 从更稀有的字符开始搜——必要时反转 word，起点数大幅下降 |
| 为什么用 0 标记 | 网格只有字母（非 0），0 天然拦截已访问格，省 visited 数组 |
| 关联题目 | [200. 岛屿数量](day21-图论hot100.md)——同样的方向数组 + 网格遍历模板 |

### 一句话记住

> **「字符不匹配就退回，匹配到底就成功——先查计数再反转，从稀有字符开始搜。」**

---

## 题目：分割回文串（Palindrome Partitioning）

**LeetCode 131 | 回溯 Hot100 | 难度：🟡 中等**

### 题目描述

给你一个字符串 `s`，请你将 `s` 分割成一些子串，使每个子串都是**回文串**。返回 `s` 所有可能的分割方案。

### 示例

```
输入：s = "aab"
输出：[["a","a","b"],["aa","b"]]

输入：s = "a"
输出：[["a"]]
```

---

## 解法：回溯（枚举切割点）

### 思路

把 `s` 想象成从前往后扫描，在每个字符后面决定"切一刀"还是"不切"。核心是用**双指针 `(i, start)`** 维护当前正在拼的子串区间 `s[start..i]`。

```
dfs(i, start)：i 是当前扫描到的位置，start 是当前正在拼的子串起点

  如果 i == s.length() → 处理完最后一个字符 → 加入答案

  不切割（i 继续往后走）：dfs(i+1, start)

  切割（要求 s[start..i] 是回文）：
    path.add(s.substring(start, i+1))   ← 切下当前回文子串
    dfs(i+1, i+1)                        ← 下一个子串从 i+1 开始
    path.removeLast()                    ← 回溯
```

**为什么不切割要 `i < s.length()-1`？** 最后一个字符后面必须切割（否则最后一刀落在空串上），所以 i 走到最后一个字符时强制切割。

### 思考方式图解

```
s = "aab"

dfs(0,0): 处理到 i=0
  不切 → dfs(1,0)
    不切 → dfs(2,0)
      切(必须): s[0..2]="aab" 不是回文 → 跳过
    切: s[0..1]="aa" 是回文 ✅ → path=["aa"] → dfs(3,2)
      切(必须): s[2..2]="b" 是回文 → path=["aa","b"] → 加入 ✅
  切: s[0..0]="a" 是回文 → path=["a"] → dfs(1,1)
    不切 → dfs(2,1)
      切(必须): s[1..2]="ab" 不是回文 → 跳过
    切: s[1..1]="a" 是回文 → path=["a","a"] → dfs(2,2)
      切(必须): s[2..2]="b" 是回文 → path=["a","a","b"] → 加入 ✅

结果：[["aa","b"],["a","a","b"]] ✅
```

### 代码实现

```java
class Solution {
    public List<List<String>> partition(String s) {
        List<List<String>> ans = new ArrayList<>();
        List<String> path = new ArrayList<>();

        dfs(0, 0, s, ans, path);
        return ans;
    }

    // i 是当前扫描位置，start 是当前正在拼的子串起点（s[start..i]）
    private void dfs(int i, int start, String s,
                     List<List<String>> ans, List<String> path) {
        if (i == s.length()) {                  // 扫描完整个字符串
            ans.add(new ArrayList<>(path));
            return;
        }

        if (i < s.length() - 1) {               // 不切割，i 继续往后走
            dfs(i + 1, start, s, ans, path);
        }

        if (isPalindrome(s, start, i)) {        // 切割：当前区间是回文
            path.add(s.substring(start, i + 1));  // 切下当前回文子串
            dfs(i + 1, i + 1, s, ans, path);      // 下一个子串从 i+1 开始
            path.removeLast();                    // 回溯
        }
    }

    // 判断 s[left..right] 是否为回文
    private boolean isPalindrome(String s, int left, int right) {
        while (left < right) {
            if (s.charAt(left++) != s.charAt(right--)) {
                return false;
            }
        }
        return true;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n × 2ⁿ)** — 每个位置切/不切两种选择，最坏 2ⁿ 种方案，每种判断回文 O(n) |
| 🧠 空间复杂度 | **O(n)** — 递归栈深度 + path |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 回溯（枚举切割点） |
| 算法类型 | 回溯、字符串 |
| 核心技巧 | **双指针 `(i, start)` 维护当前子串区间——不切就 i 前进，回文就切割** |
| 为什么 `i < s.length()-1` | 最后一个字符后面必须切（否则最后一段为空），所以 i 到末尾时强制切割 |
| 优化方向 | 可以用 DP 预处理 `isPalindrome` 表（回文串的转移），把判断回文降到 O(1) |

### 一句话记住

> **「扫描中切不切，回文才能切——双指针维护区间。」**

---

## 题目：N 皇后（N-Queens）

**LeetCode 51 | 回溯 Hot100 | 难度：🔴 困难**

### 题目描述

按照国际象棋的规则，皇后可以攻击与之处在同一行或同一列或同一斜线上的棋子。

**n 皇后问题**研究的是如何将 `n` 个皇后放置在 `n×n` 的棋盘上，并且使皇后彼此之间不能相互攻击。

给你一个整数 `n`，返回所有不同的 **n 皇后问题** 的解决方案。

每一种解法包含一个不同的 n 皇后问题的棋子放置方案，该方案中 `'Q'` 和 `'.'` 分别代表了皇后和空位。

### 示例

```
输入：n = 4
输出：[[".Q..","...Q","Q...","..Q."],
      ["..Q.","Q...","...Q",".Q.."]]

输入：n = 1
输出：[["Q"]]
```

---

## 解法：回溯（行列 + 双斜线标记）

### 思路

N 皇后是**逐行放皇后**的回溯问题。每行只能放一个皇后，所以用 `queens[r] = c` 记录第 r 行皇后所在的列。

判断冲突需要三个标记数组：

```
col[c]      ← 第 c 列是否已有皇后
diag1[r+c]  ← 主对角线（↘方向）：同一斜线上的 r+c 相等
diag2[r-c+n-1] ← 副对角线（↙方向）：同一斜线上的 r-c 相等
```

**为什么两条斜线用 `r+c` 和 `r-c`？**

```
主对角线 ↘：坐标 (r, c) 满足 r+c = 常数（如 (0,0),(1,1),(2,2) 的 r+c 都是 0）
副对角线 ↙：坐标 (r, c) 满足 r-c = 常数（如 (0,2),(1,1),(2,0) 的 r-c 都是 -2）

r-c 可能为负数 → 加偏移 n-1 使其落到数组下标范围 [0, 2n-2]
```

### 思考方式图解

```
n = 4 的搜索过程（简化）：

第 0 行：尝试 c=0 → (0,0) 放 Q
  第 1 行：c=0 冲突(列), c=1 冲突(副斜线), c=2 → (1,2) 放 Q
    第 2 行：c=0 冲突(列), c=1 冲突(斜线), c=2 冲突(列), c=3 冲突(斜线)
    → 无解，回溯
  第 1 行：c=3 → (1,3) 放 Q
    第 2 行：c=0 → (2,0)? 检查 (0,0) 主斜线 r+c=2 vs (2,0) r+c=2 → 冲突
            c=1 → (2,1) 放 Q ✅
      第 3 行：c=2 → (3,2) 放 Q ✅ → 找到一组解
      第 3 行：c=3 → 冲突
    → 得到解 1：[[.Q..],[...Q],[Q...],[..Q.]]
  ...
```

### 代码实现

```java
class Solution {
    public List<List<String>> solveNQueens(int n) {
        List<List<String>> ans = new ArrayList<>();
        int[] queens = new int[n];               // 皇后放在 (r, queens[r])
        boolean[] col = new boolean[n];          // 列是否被占用
        boolean[] diag1 = new boolean[n * 2 - 1]; // 主对角线 ↘（r+c）
        boolean[] diag2 = new boolean[n * 2 - 1]; // 副对角线 ↙（r-c+n-1）

        dfs(0, queens, col, diag1, diag2, ans);
        return ans;
    }

    // r 表示正在放第 r 行的皇后
    private void dfs(int r, int[] queens, boolean[] col,
                     boolean[] diag1, boolean[] diag2, List<List<String>> ans) {
        int n = col.length;
        if (r == n) {                            // 所有行都放好了
            List<String> board = new ArrayList<>(n);
            for (int c : queens) {               // c = queens[r]，该行皇后的列
                char[] row = new char[n];
                Arrays.fill(row, '.');
                row[c] = 'Q';
                board.add(new String(row));
            }
            ans.add(board);
            return;
        }

        // 尝试在第 r 行的每一列放皇后
        for (int c = 0; c < n; c++) {
            int rc = r - c + n - 1;              // 副对角线下标（加偏移防负数）
            if (!col[c] && !diag1[r + c] && !diag2[rc]) {  // 列和两条斜线都不冲突
                queens[r] = c;                   // 放皇后（直接覆盖，无需恢复）
                col[c] = diag1[r + c] = diag2[rc] = true;  // 占用列和斜线
                dfs(r + 1, queens, col, diag1, diag2, ans); // 下一行
                col[c] = diag1[r + c] = diag2[rc] = false;  // 恢复现场
            }
        }
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n!)** — 第一行 n 种选择，第二行最多 n-1 种……但有剪枝 |
| 🧠 空间复杂度 | **O(n)** — 三个标记数组 + queens + 递归栈 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 回溯（逐行放皇后 + 斜线标记） |
| 算法类型 | 回溯 |
| 核心技巧 | **`col[c]` 查列冲突、`diag1[r+c]` 查主斜线、`diag2[r-c+n-1]` 查副斜线** |
| 斜线下标推导 | 主斜线 `r+c` 恒定；副斜线 `r-c` 恒定（加偏移 n-1 防负下标） |
| queens[r]=c 无需恢复 | 每行的皇后会被下一轮直接覆盖，所以不用回溯 queens 本身 |
| 需要恢复的 | 只有三个 boolean 标记数组 |
| 关联题目 | [79. 单词搜索](day26-回溯hot100-part3.md)——同样的回溯框架 |

### 一句话记住

> **「逐行放皇后，列 + 双斜线三标记——r+c 主斜线，r-c 副斜线。」**

---

*练习日期：2026-08-06*
