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

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(mn × 4^L)** — m×n 个起点，每个最多探索 4^L 条路径（L 为单词长度） |
| 🧠 空间复杂度 | **O(L)** — 递归栈深度 = 单词长度 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | DFS 回溯（网格搜索） |
| 算法类型 | 回溯、DFS |
| 核心技巧 | **字符不匹配即返回；匹配到末尾即成功；`board[i][j]=0` 标记已访问，回溯时恢复** |
| 为什么用 0 标记 | 网格只有字母（非 0），0 天然拦截已访问格，省 visited 数组 |
| 剪枝顺序 | 先判断 `board[i][j] != word[k]` 再判断是否最后一个——减少无效递归 |
| 关联题目 | [200. 岛屿数量](day21-图论hot100.md)——同样的方向数组 + 网格遍历模板 |

### 一句话记住

> **「字符不匹配就退回，匹配到底就成功——标记 0 防重复，回溯恢复现场。」**

---

*练习日期：2026-08-05*
