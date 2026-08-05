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

*练习日期：2026-08-05*
