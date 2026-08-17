# Day 21 — 图论 Hot100

## 题目：岛屿数量（Number of Islands）

**LeetCode 200 | 图论 Hot100 | 难度：🟡 中等**

### 题目描述

给你一个由 `'1'`（陆地）和 `'0'`（水）组成的的二维网格，请你计算网格中岛屿的数量。

岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。

### 示例

```
输入：grid = [
  ["1","1","1","1","0"],
  ["1","1","0","1","0"],
  ["1","1","0","0","0"],
  ["0","0","0","0","0"]
]
输出：1

输入：grid = [
  ["1","1","0","0","0"],
  ["1","1","0","0","0"],
  ["0","0","1","0","0"],
  ["0","0","0","1","1"]
]
输出：3
```

---

## 解法：DFS 沉没法（Flood Fill）

### 思路

遍历每个格子，遇到陆地就**DFS 把整座岛沉掉**（标记为已访问），同时岛屿计数 +1。

```
遍历网格中的每个格子：
  如果 grid[i][j] == '1'（找到一座新的岛）：
    ans++                ← 岛屿数 +1
    dfs(grid, i, j)      ← 把这座岛全部沉掉（标记为 '2'）

dfs 函数：
  如果出界 或 当前格子不是 '1' → 返回
  标记 grid[i][j] = '2'  ← 标记为已访问
  递归访问上下左右四个相邻格子
```

**为什么标记为 '2' 而不是 '0'？** 标记为 '2' 可以在调试时区分"原来的水"和"被访问过的陆地"；标记为 '0' 也可以，逻辑等价。

### 思考方式图解

```
grid = [
  ["1","1","0"],
  ["1","0","0"],
  ["0","0","1"]
]

遍历过程：
  i=0,j=0 → grid[0][0]=='1' → ans=1, dfs 沉没此岛
    dfs(0,0): 置 '2', 递归(0,1),(1,0)
      dfs(0,1): 置 '2', 递归(0,2)=0 返回, (1,1)=0 返回
      dfs(1,0): 置 '2', 递归(1,-1)越界, (2,0)=0 返回
    → 岛 1 全部沉没

  继续遍历...
  i=2,j=2 → grid[2][2]=='1' → ans=2, dfs 沉没
    dfs(2,2): 置 '2'
    → 岛 2 全部沉没

结果：2 ✅
```

### 代码实现

```java
class Solution {
    public int numIslands(char[][] grid) {
        int ans = 0;
        for (int i = 0; i < grid.length; i++) {
            for (int j = 0; j < grid[0].length; j++) {
                if (grid[i][j] == '1') {   // 找到一座新岛
                    dfs(grid, i, j);       // 沉没整座岛
                    ans++;                 // 计数 +1
                }
            }
        }
        return ans;
    }

    // 从 (i,j) 开始 DFS，把所有相邻的 '1' 都标记为 '2'
    private void dfs(char[][] grid, int i, int j) {
        if (i < 0 || i >= grid.length || j < 0 || j >= grid[0].length || grid[i][j] != '1') {
            return;
        }
        grid[i][j] = '2';    // 标记为已访问（"沉没"）

        dfs(grid, i, j - 1); // 左
        dfs(grid, i, j + 1); // 右
        dfs(grid, i - 1, j); // 上
        dfs(grid, i + 1, j); // 下
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(mn)** — 每个格子最多访问一次 |
| 🧠 空间复杂度 | **O(mn)** — 最坏情况下递归栈深度 = 整个网格都是陆地 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | DFS 沉没法（Flood Fill） |
| 算法类型 | 图论、DFS |
| 核心技巧 | **遍历网格，遇到 '1' 就 DFS 沉没整座岛（标记为 '2'），岛屿数 +1** |
| 为什么用 '2' | 区分原始 '0'（水）和已访问的陆地；用 visited 数组也可以，但标记法省空间 |
| BFS 写法 | 也可以用队列实现 BFS 沉没，效果相同 |
| 关联题目 | [695. 岛屿的最大面积](https://leetcode.cn/problems/max-area-of-island/)——同样模板，统计面积 |

### 一句话记住

> **「遇 1 沉岛，DFS 淹没上下左右——每沉一座岛计数加一。」**

---

## 题目：腐烂的橘子（Rotting Oranges）

**LeetCode 994 | 图论 Hot100 | 难度：🟡 中等**

### 题目描述

给定一个 `m x n` 的网格，每个单元格可以有以下三个值之一：

- `0` — 空单元格
- `1` — 新鲜橘子
- `2` — 腐烂的橘子

每分钟，腐烂的橘子会将其 **4 个方向上相邻** 的新鲜橘子腐烂。返回直到网格中没有新鲜橘子为止所必须经过的**最小分钟数**。如果不可能腐烂所有橘子，返回 `-1`。

### 示例

```
输入：grid = [[2,1,1],[1,1,0],[0,1,1]]
输出：4

输入：grid = [[2,1,1],[0,1,1],[1,0,1]]
输出：-1

输入：grid = [[0,2]]
输出：0
```

---

## 解法：多源 BFS（逐层扩散）

### 思路

这是经典的**多源 BFS**——一开始可能有多个腐烂的橘子，它们同时向四周扩散，每分钟扩散一层。

```
1. 统计新鲜橘子数量 fresh，把所有初始腐烂橘子的位置加入队列 q
2. 如果 fresh == 0 → 直接返回 0
3. BFS 逐层扩散：
   ans++（分钟数 +1）
   遍历当前层的所有腐烂橘子：
     向四个方向扩散，把新鲜橘子变成腐烂
     fresh--，新腐烂的橘子加入下一层队列
4. 循环直到 fresh == 0 或队列为空
5. 如果 fresh > 0 → 返回 -1（有橘子永远烂不了）
   否则返回 ans
```

**为什么用 List 而不是 Queue？** 你的实现用 `List<int[]>` 列表交替（`temp` 和 `q`），天然按层分隔，不需要像队列那样用 `size()` 来分层。效果相同。

### 思考方式图解

```
grid = [[2,1,1],
        [1,1,0],
        [0,1,1]]

初始：fresh=6, q=[(0,0)]

第 1 分钟：ans=1
  temp=[(0,0)]
  (0,0) → (0,1)=1变2, (1,0)=1变2
  fresh=4, q=[(0,1),(1,0)]

第 2 分钟：ans=2
  temp=[(0,1),(1,0)]
  (0,1) → (0,2)=1变2
  (1,0) → (2,0)=0, (1,1)=1变2
  fresh=2, q=[(0,2),(1,1)]

第 3 分钟：ans=3
  temp=[(0,2),(1,1)]
  (0,2) → 全腐烂或无
  (1,1) → (2,1)=1变2
  fresh=1, q=[(2,1)]

第 4 分钟：ans=4
  temp=[(2,1)]
  (2,1) → (2,2)=1变2
  fresh=0, q=[]

fresh=0 → 返回 ans=4 ✅
```

### 代码实现

```java
class Solution {
    private static final int[][] DIRECTION = {{-1,0},{1,0},{0,-1},{0,1}}; // 上下左右

    public int orangesRotting(int[][] grid) {
        int m = grid.length;
        int n = grid[0].length;
        int fresh = 0;
        List<int[]> q = new ArrayList<>();

        // 统计新鲜橘子，收集初始腐烂橘子
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == 1) {
                    fresh++;
                } else if (grid[i][j] == 2) {
                    q.add(new int[]{i, j});
                }
            }
        }

        int ans = 0;
        // BFS 逐层扩散
        while (fresh > 0 && !q.isEmpty()) {
            ans++;
            List<int[]> temp = q;        // 当前层的腐烂橘子
            q = new ArrayList<>();       // 下一层的新腐烂橘子

            for (int[] pos : temp) {
                for (int[] d : DIRECTION) {
                    int i = pos[0] + d[0];
                    int j = pos[1] + d[1];
                    if (i >= 0 && i < m && j >= 0 && j < n && grid[i][j] == 1) {
                        fresh--;
                        grid[i][j] = 2;
                        q.add(new int[]{i, j});
                    }
                }
            }
        }

        return fresh > 0 ? -1 : ans;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(mn)** — 每个格子最多入队一次 |
| 🧠 空间复杂度 | **O(mn)** — 队列最多存储所有格子 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 多源 BFS（逐层扩散） |
| 算法类型 | 图论、BFS |
| 核心技巧 | **多个腐烂起点同时 BFS，用 fresh 计数判断是否全部腐烂** |
| 层数 = 分钟数 | BFS 的层数就是经过的分钟数，用列表交替天然分层 |
| 为什么用 List 不用 Queue | 用列表交替 temp/q 天然按层分隔，效果等价于队列 + size() |
| 返回 -1 的条件 | BFS 结束后 fresh > 0 → 有橘子被墙隔开，永远烂不了 |
| 关联题目 | [200. 岛屿数量](day21-图论hot100.md)——DFS 沉没法 |

### 一句话记住

> **「多源 BFS 逐层扩散，新鲜橘子归零即结束——还有剩余返回 -1。」**

---

## 题目：课程表（Course Schedule）

**LeetCode 207 | 图论 Hot100 | 难度：🟡 中等**

### 题目描述

你这个学期必须选修 `numCourses` 门课程，记为 `0` 到 `numCourses - 1`。

在选修某些课程之前需要先修读另一些课程，先修关系用 `prerequisites` 给出：`prerequisites[i] = [a, b]` 表示想要学习课程 `a`，你需要先完成课程 `b`。

判断是否**可能**完成所有课程的学习？如果可以则返回 `true`，否则返回 `false`。

### 示例

```
输入：numCourses = 2, prerequisites = [[1,0]]
输出：true
解释：想学 1 先学 0，顺序 0→1 可行。

输入：numCourses = 2, prerequisites = [[1,0],[0,1]]
输出：false
解释：1 依赖 0，0 又依赖 1，形成环，永远学不完。
```

---

## 解法：DFS 三色标记法（拓扑排序 / 判环）

### 思路

把课程和依赖关系看成一张**有向图**：边 `b → a` 表示"先学 b 才能学 a"。

能否学完所有课 = 这张有向图**有没有环**（有环 → 循环依赖 → 不可能；无环 → 可以做拓扑排序 → 可能）。

用经典的**三色标记（白/灰/黑）**做 DFS 判环：

```
colors[x] 的状态：
  0 = 白色（未访问）
  1 = 灰色（正在当前 DFS 路径上，已经入栈但还没回溯完毕）
  2 = 黑色（已经彻底访问完，安全）

DFS(x)：
  colors[x] = 1（染灰，标记"我正在这条路径上"）
  对每个邻居 y：
    如果 colors[y] == 1 → 遇到"灰色"邻居，说明从当前路径绕回了自己 → 有环！返回 true
    如果 colors[y] == 0 且 dfs(y) 也发现环 → 返回 true
  colors[x] = 2（染黑，安全结束）
  返回 false（这条路径上没有环）

主函数：对每个白色节点都 DFS 一遍（图可能不连通），只要任意一次发现环就返回 false。
```

**为什么"遇到灰色"就是环？** 灰色代表"还在当前调用栈里、没回溯完"。如果从当前路径又走到一个灰色节点，说明存在一条 `x → ... → y → ... → x` 的回路，即循环依赖。黑色节点则是"已经确认安全、不属于任何环"，再遇到它不会误判。

> **运算符优先级提醒**：代码里 `colors[y]==1 || colors[y]==0 && dfs(y,...)` 等价于 `colors[y]==1 || (colors[y]==0 && dfs(y,...))`（`&&` 优先级高于 `||`）。若 `colors[y]==1` 直接短路返回 true；否则只有白色才递归。逻辑正确，但**建议加括号写成 `colors[y]==1 || (colors[y]==0 && dfs(...))`**，可读性更好、避免隐患。

### 思考方式图解

```
numCourses=2, prerequisites=[[1,0],[0,1]]  → 有环，应返回 false

建图（b→a）：
  0 → 1
  1 → 0
          0 ⇄ 1   （互相指向，环）

DFS(0)：colors[0]=1
  → 邻居 1：colors[1]==0，DFS(1)
      DFS(1)：colors[1]=1
        → 邻居 0：colors[0]==1（灰色！）→ 返回 true（发现环）
  → DFS(0) 返回 true → 主函数返回 false ✅

numCourses=2, prerequisites=[[1,0]]  → 无环，应返回 true
建图：0 → 1
DFS(0)：colors[0]=1 → 邻居 1：colors[1]=0，DFS(1)
  DFS(1)：colors[1]=1 → 无邻居 → colors[1]=2，返回 false
colors[0]=2，返回 false（无环）
主函数：没发现任何环 → 返回 true ✅
```

### 代码实现

```java
class Solution {
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        // 建邻接表：边 b → a（b 是 a 的先修课）
        List<Integer>[] g = new ArrayList[numCourses];
        Arrays.setAll(g, i -> new ArrayList<>());
        for (int[] p : prerequisites) {
            g[p[1]].add(p[0]);     // p[1] 指向 p[0]
        }

        int[] colors = new int[numCourses];  // 0白 1灰 2黑
        for (int i = 0; i < numCourses; i++) {
            // 从每个白色节点出发 DFS；任一路径发现环就返回 false
            if (colors[i] == 0 && dfs(i, g, colors)) {
                return false;
            }
        }
        return true;
    }

    // 返回 true 表示在 x 出发的路径上发现了环
    private boolean dfs(int x, List<Integer>[] g, int[] colors) {
        colors[x] = 1;                       // 染灰：进入当前路径
        for (int y : g[x]) {
            if (colors[y] == 1 ||            // 遇到灰色邻居 → 有环
                colors[y] == 0 && dfs(y, g, colors)) {  // 或白色邻居递归发现环
                return true;
            }
        }
        colors[x] = 2;                       // 染黑：安全结束
        return false;
    }
}
```

> **其他解法对比**：
> - **BFS 拓扑排序（Kahn 算法）**：算每个节点入度，入度为 0 的入队，逐步剥层；剥完的节点数 < 总课程数说明有环。时间 O(V+E)，不用递归，更不容易爆栈。
> - **DFS 三色**（本题写法）：更直观体现"路径上的环"，但图深时递归栈可能溢出。

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(V + E)** — 每个节点、每条边都只访问一次（V=课程数，E=依赖数） |
| 🧠 空间复杂度 | **O(V + E)** — 邻接表 O(E) + 递归栈/colors 数组 O(V) |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | DFS 三色标记法（拓扑排序 / 判环） |
| 算法类型 | 图论、DFS、有向图、拓扑排序 |
| 核心技巧 | **三色 0白/1灰/2黑；DFS 遇灰色邻居即环；全黑则无环** |
| 建图方向 | `g[p[1]].add(p[0])`：`b → a` 表示先修 b 才能学 a |
| 为何"灰=环" | 灰色在调用栈中未回溯，又走到它 = 存在回路（循环依赖） |
| 返回语义 | 返回 false = 有环 = 无法修完；true = 无环 = 可拓扑排序完成 |
| 关联题目 | [210. 课程表 II](https://leetcode.cn/problems/course-schedule-ii/)（输出拓扑序）、[207 的 BFS 版](https://leetcode.cn/problems/course-schedule/)（Kahn 算法） |

### 一句话记住

> **「课程表 = 建有向图判环：DFS 三色标记，遇灰色即循环依赖，学不完。」**

---

## 题目：实现 Trie（前缀树）

**LeetCode 208 | 设计 / 字典树 / 技巧 Hot100 | 难度：🟡 中等**

### 题目描述

**Trie**（发音类似 "try"），又称**前缀树**或**字典树**，是一种有序树，用于保存关联数组，其中的键通常是字符串。

请你实现 `Trie` 类：

- `Trie()` 初始化前缀树对象。
- `void insert(String word)` 向前缀树中插入字符串 `word`。
- `boolean search(String word)` 如果字符串 `word` 在前缀树中，返回 `true`（也就是**曾经完整插入过**）；否则返回 `false`。
- `boolean startsWith(String prefix)` 如果之前插入的字符串中**有以 `prefix` 为前缀**的，返回 `true`；否则返回 `false`。

### 示例

```
输入：
["Trie", "insert", "search", "search", "startsWith", "insert", "search"]
[[], ["apple"], ["apple"], ["app"], ["app"], ["app"], ["app"]]

输出：
[null, null, true, false, true, null, true]

解释：
insert("apple") → 树里有了 apple
search("apple") → true （完整单词存在）
search("app")   → false（没完整插入过 app）
startsWith("app") → true（apple 以 app 为前缀）
insert("app")   → 现在 app 也完整存在了
search("app")   → true
```

---

## 解法：多叉树 + 三态查找

### 思路

Trie 的本质是一棵**多叉树**：每个节点挂 26 个孩子（对应 a~z），外加一个 `end` 标记表示"从根到这里的路径是否构成一个完整单词"。

```
根节点 (root)
 ├─ a ─ p ─ p ─ p ─ l ─ e (end=true)      ← insert("apple")
 │            └ (end=true)                 ← 若再 insert("app")，这里标 end=true
 └─ b ─ ...
```

三个操作都围绕"**沿字符往下走**"展开，核心差异在 `find` 的返回值设计：

```
find(prefix) 返回三态：
  0 → 路径中途断了（某个字符没有对应孩子）→ 前缀都不存在
  1 → 走到了前缀末尾，但 end==false → 是某个单词的"前缀"，但本身不是完整单词
  2 → 走到了末尾且 end==true       → 是一个完整插入过的单词

search(word)     = find(word) == 2      （必须走到末尾且是完整单词）
startsWith(pre)  = find(pre)  != 0      （只要能走完前缀就算，不管是不是完整单词）
```

### 思考方式图解

```
insert("apple"):
  cur=root
  a: root.son[a]==null? 建节点 → cur 下移
  p: 建/走 → 下移
  p: 建/走 → 下移
  p: 建/走 → 下移
  l: 建/走 → 下移
  e: 建/走 → 下移
  末尾 cur.end = true          ✅

search("app"):
  a→p→p 都走得通，停在 p(p) 节点
  cur.end == false → 返回 1 → search: 1==2? false ✅（没完整插过 app）

startsWith("app"):
  同样走到 p(p)，find 返回 1 → 1!=0 → true ✅（apple 以 app 为前缀）

search("apple"):
  a→p→p→p→l→e 走完，cur.end==true → 返回 2 → 2==2 true ✅
```

### 代码实现

```java
class Trie {
    // 内部节点：26 个孩子 + 结束标记
    private static class Node {
        Node[] son = new Node[26];
        boolean end = false;       // 从根到本节点是否构成完整单词
    }
    private final Node root = new Node();   // 根节点（不存字符）

    public Trie() {}               // 根节点在字段初始化时已建好

    // 插入：沿路径建节点，末尾标 end=true
    public void insert(String word) {
        Node cur = root;
        for (char c : word.toCharArray()) {
            c -= 'a';                          // char 转 0~25 下标
            if (cur.son[c] == null) {
                cur.son[c] = new Node();      // 缺则补建
            }
            cur = cur.son[c];
        }
        cur.end = true;
    }

    // 查找单词：必须是完整单词（end==true）才返回 true
    public boolean search(String word) {
        return find(word) == 2;
    }

    // 前缀查询：能走完前缀即可（不要求 end）
    public boolean startsWith(String prefix) {
        return find(prefix) != 0;
    }

    // 三态查找：0=路径断 / 1=走到但非单词 / 2=完整单词
    private int find(String word) {
        Node cur = root;
        for (char c : word.toCharArray()) {
            c -= 'a';
            if (cur.son[c] == null) {
                return 0;                      // 没这个分支 → 前缀都不存在
            }
            cur = cur.son[c];
        }
        return cur.end ? 2 : 1;
    }
}
```

> **代码正确性**：你的实现完全正确，且用"三态 `find`"把 `search` 和 `startsWith` 统一到一个方法里，干净利落。几个小亮点：
> - `c -= 'a'` 改的是 for-each 的**循环副本**（Java 中 for-each 遍历数组时 `c` 是每次元素的拷贝），不会污染原字符串，合法且简洁；若想更直观也可写成 `int idx = c - 'a'`。
> - `root` 用 `final` + 字段初始化，构造函数留空即可，避免忘记建根。
> - 区分 `search`/`startsWith` 的关键就在 `end` 标记：`startsWith` 只看"能不能走完"，`search` 额外要求"走完时恰好是单词结尾"。

> **延伸（高频考点）**：Trie 的威力在"多串共享前缀、按前缀聚合"。Hot100 里常配合它出题，例如 [212. 单词搜索 II](https://leetcode.cn/problems/word-search-ii/)（在矩阵里找所有字典单词，Trie + 回溯剪枝）、[677. 键值映射](https://leetcode.cn/problems/map-sum-pairs/)（带权重前缀和）。进阶还可把 `son` 改成 `Map<Character, Node>` 支持任意字符集，或用**压缩 Trie（Radix Tree）**省空间。

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | 每个操作 **O(L)**，`L` 为单词/前缀长度（逐字符走一层） |
| 🧠 空间复杂度 | **O(Σ L)** — 所有插入单词的字符总数（每个字符对应一个节点）；最坏全无公共前缀时即总字符数 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 数据结构 | Trie / 前缀树 / 字典树（每个节点 26 个孩子 + end 标记） |
| 核心操作 | **insert 沿路建节点标 end；search 要求走到末尾且 end；startsWith 走到末尾即可** |
| 三态 find | **0=路径断 / 1=走到但非单词 / 2=完整单词** —— 统一 search 与 startsWith |
| search vs startsWith | 差异全在 `end` 标记：前者要 end==true，后者只看能否走完 |
| 时间 | 每个操作 O(L)，与字典大小无关（这是 Trie 优于哈希表的场景优势） |
| 空间 | O(ΣL) 节点数，最坏等于所有字符串字符总和 |
| 关联题目 | [212. 单词搜索 II](https://leetcode.cn/problems/word-search-ii/)（Trie+回溯）、[677. 键值映射](https://leetcode.cn/problems/map-sum-pairs/)（前缀和）、[14. 最长公共前缀](https://leetcode.cn/problems/longest-common-prefix/)（也可用 Trie） |

### 一句话记住

> **「Trie = 每个节点挂 26 个孩子 + end 标记；search 要走到末尾且 end，startsWith 走到末尾即可——三态 find 一统两者。」**

---

*练习日期：2026-07-30*
