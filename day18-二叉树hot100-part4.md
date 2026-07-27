# Day 18 — 二叉树 Hot100（Part 4）

## 题目：路径总和 III（Path Sum III）

**LeetCode 437 | 二叉树 Hot100 | 难度：🟡 中等**

### 题目描述

给定一个二叉树的根节点 `root`，和一个整数 `targetSum`，求该二叉树里节点值之和等于 `targetSum` 的**路径**的数目。

**路径**不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。

### 示例

```
输入：root = [10,5,-3,3,2,null,11,3,-2,null,1], targetSum = 8
输出：3
解释：有三条和等于 8 的路径，如图所示。

       10
      /  \
     5   -3
    / \    \
   3   2   11
  / \   \
 3  -2   1

路径：5 → 3       (和为 8)
     5 → 2 → 1    (和为 8)
     -3 → 11      (和为 8)

输入：root = [1], targetSum = 1
输出：1
```

---

## 解法：前缀和 + 回溯（DFS）

### 思路

暴力解法是枚举所有路径起点和终点，O(n²)。用**前缀和**可以降到 **O(n)**。

**核心思想**：把从根到当前节点的路径上的节点值之和记作 `s`。如果存在一个更早的节点，从根到它的路径和为 `s - targetSum`，那么从那个节点到当前节点的路径和就是 `targetSum`。

```
        根
        │
   ┌────┴────┐
   │         │
   s - t     s
   │         │
 起点      终点（当前节点）
 ────────────→ 路径和 = targetSum
```

**算法步骤**（DFS 回溯）：

```
1. 用 HashMap 记录「从根到当前节点的各前缀和」出现的次数
   − 初始化 cnt[0] = 1（表示「空路径」和为 0，出现过一次）
2. DFS 遍历二叉树：
   − 更新当前前缀和 s = s + node.val
   − ans += cnt[s - targetSum]  （以当前节点为终点，有多少个起点）
   − cnt[s]++（记录当前前缀和，进入子树）
   − DFS 左右子树
   − cnt[s]--（回溯，恢复现场，确保其他分支不受影响）
```

**为什么需要回溯（`cnt[s]--`）？** 因为 HashMap 记录的是**当前路径**上的前缀和。当递归从某个节点返回时，该节点对应的前缀和不应该再被其他分支统计到——所以要在返回前撤销。

### 思考方式图解

```
targetSum = 8

       10
      /  \
     5   -3
    / \    \
   3   2   11
  / \   \
 3  -2   1

DFS 遍历过程（只展示关键步骤）：

初始化: cnt = {0:1}

node=10:  s=10  cnt[s-8]=cnt[2]=0  ans=0
          cnt[10]=1, cnt={0:1, 10:1}

node=5:   s=15  cnt[15-8]=cnt[7]=0  ans=0
          cnt[15]=1, cnt={0:1, 10:1, 15:1}

node=3:   s=18  cnt[18-8]=cnt[10]=1  ans=1  ✅ (10→5→3)
          cnt[18]=1

  node=3:   s=21  cnt[21-8]=cnt[13]=0  ans=1
            cnt[21]=1 → 回溯 → cnt[21]=0

  node=-2:  s=16  cnt[16-8]=cnt[8]=0  ans=1
            cnt[16]=1 → 回溯 → cnt[16]=0

回溯 → cnt[18]=0

node=2:   s=17  cnt[17-8]=cnt[9]=0  ans=1
          cnt[17]=1

  node=1:   s=18  cnt[18-8]=cnt[10]=1  ans=2  ✅ (10→5→2→1)
            cnt[18]=1 → 回溯 → cnt[18]=0

回溯 → cnt[17]=0, cnt[15]=0

node=-3:  s=7   cnt[7-8]=cnt[-1]=0  ans=2
          cnt[7]=1

  node=11:  s=18  cnt[18-8]=cnt[10]=1  ans=3  ✅ (10→-3→11)
            cnt[18]=1 → 回溯 → cnt[18]=0

回溯 → cnt[7]=0, cnt[10]=0

结果：3 ✅
```

### 代码实现

```java
class Solution {
    private int ans;

    public int pathSum(TreeNode root, int targetSum) {
        // key: 从根到当前节点的前缀和
        // value: 该前缀和出现的次数
        Map<Long, Integer> cnt = new HashMap<>();
        cnt.put(0L, 1);  // 空路径和为 0，出现一次
        dfs(root, 0, targetSum, cnt);
        return ans;
    }

    // s 表示从根到 node 的父节点的前缀和（node 的值尚未计入）
    private void dfs(TreeNode node, long s, int targetSum, Map<Long, Integer> cnt) {
        if (node == null) return;

        s += node.val;  // 当前前缀和

        // 以当前节点为终点，有多少个起点使得路径和为 targetSum
        ans += cnt.getOrDefault(s - targetSum, 0);

        // 记录当前前缀和，进入子树
        cnt.put(s, cnt.getOrDefault(s, 0) + 1);  // cnt[s]++
        dfs(node.left, s, targetSum, cnt);
        dfs(node.right, s, targetSum, cnt);
        // 回溯，恢复现场——减到 0 时移除 key，避免无效条目残留
        int c = cnt.getOrDefault(s, 0) - 1;
        if (c == 0) {
            cnt.remove(s);
        } else {
            cnt.put(s, c);
        }
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 每个节点访问一次 |
| 🧠 空间复杂度 | **O(n)** — HashMap + 递归栈，最坏 O(n) |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 前缀和 + DFS 回溯 |
| 算法类型 | 二叉树、前缀和、回溯 |
| 核心技巧 | **前缀和 `s`，用 HashMap 统计各前缀和的出现次数；`cnt[s - target]` 就是以当前节点为终点的路径数** |
| 为什么用 long | `s` 可能很大（节点值范围 ±10⁹），`int` 会溢出 |
| 初始化 cnt[0]=1 | 空路径的前缀和为 0，出现一次——这使得以根节点为起点的路径也能被正确统计 |
| 回溯 | `cnt[s]--` 恢复现场，确保离开当前分支后前缀和不影响其他分支 |
| 关联题目 | [560. 和为 K 的子数组](https://leetcode.cn/problems/subarray-sum-equals-k/)——完全相同的思路，只是从数组换到了二叉树 |

### 一句话记住

> **「前缀和 + HashMap 计数，回溯恢复现场——从根一路记下来，cnt[s-target] 就是路径数。」**

---

## 题目：二叉树的最近公共祖先（Lowest Common Ancestor of a Binary Tree）

**LeetCode 236 | 二叉树 Hot100 | 难度：🟡 中等**

### 题目描述

给定一个二叉树，找到该树中两个指定节点的最近公共祖先。

**最近公共祖先**的定义为：对于有根树 T 的两个节点 p、q，最近公共祖先表示为一个节点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。

### 示例

```
输入：root = [3,5,1,6,2,0,8,null,null,7,4], p=5, q=1
输出：3
        3
       / \
      5   1
     / \ / \
    6  2 0  8
      / \
     7   4

输入：root = [3,5,1,6,2,0,8,null,null,7,4], p=5, q=4
输出：5
解释：节点 5 的子树中有 4，5 就是它自己的祖先
```

---

## 解法：递归后序遍历

### 思路

核心观察：**p 和 q 的最近公共祖先，就是从下往上找，第一个左右子树中分别包含 p 和 q 的节点。**

```
递归（后序遍历）：
  1. 如果 root 是 null / p / q → 直接返回 root
  2. 递归在左子树中找 p 或 q → left
  3. 递归在右子树中找 p 或 q → right
  4. 如果 left 和 right 都不为 null → 当前 root 就是 LCA
  5. 否则返回 left 或 right 中非空的那个
```

**三种情况：**

```
情况 1：p 和 q 分别在 root 的两侧
    root ← LCA
   ↙    ↘
  p      q

情况 2：p 是 q 的祖先（或反之）
      p ← LCA
     ↙
    q

情况 3：p 和 q 都在同一侧
      root
     ↙
    x ← LCA
   ↙ ↘
  p   q
```

### 思考方式图解

```
        3
       / \
      5   1
     / \ / \
    6  2 0  8
      / \
     7   4

p=5, q=1 → LCA 应该返回 3

递归过程（后序，从叶子往上）：

  6: 左右都 null → return null
  7: 左右都 null → return null
  4: 左右都 null → return null
  2: left=null(7), right=null(4) → return null
  5: left=null(6), right=null(2) → 都不是 p/q → return 5  ✅（5 就是 p）
  0: 左右都 null → return null
  8: 左右都 null → return null
  1: left=null(0), right=null(8) → return 1  ✅（1 就是 q）
  3: left=5(非空), right=1(非空) → 左右都找到了 → return 3 ✅
```

```
p=5, q=4 → LCA 应该返回 5

递归过程：
  6: null → return null
  7: null → return null
  4: null → return null
  2: left=null, right=null → return null
  5: left=null, right=null → return 5  ✅（5 就是 p）
  1: ... → return 1
  3: left=5(非空), right=1(非空但不是 p/q) → wait...
```

**这里要注意**：对于 `p=5, q=4`，实际上 4 在 5 的子树中，递归到 5 的时候，`root == p` 就直接返回 5 了，不会往下递归去找 4。所以 5 就是 LCA。

### 代码实现

```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        // 如果 root 是 null，或者就是 p/q 本身 → 直接返回
        if (root == null || root == p || root == q) {
            return root;
        }

        // 在左右子树中找 p 和 q
        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);

        // 左右各找到一个 → 当前 root 就是最近公共祖先
        if (left != null && right != null) {
            return root;
        }

        // 否则返回非空的那一侧（说明两个目标都在同一侧）
        return left != null ? left : right;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 每个节点最多访问一次 |
| 🧠 空间复杂度 | **O(h)** — 递归栈深度，最坏 O(n) |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 递归后序遍历 |
| 算法类型 | 二叉树、DFS |
| 核心技巧 | **左右子树各找一次，都找到则当前节点是 LCA；只找到一个则返回那个（说明两个目标在同一侧）** |
| 终止条件 | root == null / root == p / root == q — 找到任何一个目标就提前返回 |
| 返回逻辑 | `left != null && right != null` → root; 否则 `left != null ? left : right` |
| 易错点 | p 在 q 的子树中时（或反之），递归到 p 就直接返回了，不会继续往下找 q——因为 p 本身就是 LCA |

### 一句话记住

> **「左右子树找 p、q，一边一个就是根，只有一边就返回那边。」**

---

## 题目：二叉树中的最大路径和（Binary Tree Maximum Path Sum）

**LeetCode 124 | 二叉树 Hot100 | 难度：🔴 困难**

### 题目描述

二叉树中的**路径**被定义为一条节点序列，序列中每对相邻节点之间都存在一条边。同一个节点在一条路径序列中**至多出现一次**。该路径**至少包含一个**节点，且不一定经过根节点。

**路径和**是路径中各节点值的总和。

给你一个二叉树的根节点 `root`，返回其**最大路径和**。

### 示例

```
输入：root = [1,2,3]
      1
     / \
    2   3
输出：6    （路径 2→1→3，和为 6）

输入：root = [-10,9,20,null,null,15,7]
     -10
     / \
    9   20
       / \
      15  7
输出：42   （路径 15→20→7，和为 42）
```

---

## 解法：DFS 后序遍历 + 全局最大值

### 思路

和「二叉树的直径（543）」的思路类似——**每个节点作为一个"拐点"，经过它的最大路径 = 左子树最大贡献 + 右子树最大贡献 + 当前节点值**。但这里求的是路径和，不是边数。

**核心思想：**

```
对于每个节点 node：
  经过 node 的最大路径和 = max(0, 左子树最大贡献) + max(0, 右子树最大贡献) + node.val
  → 更新全局最大值 ans

  node 向上返回的"最大贡献" = max(左子树最大贡献, 右子树最大贡献) + node.val
  → 但如果这个值 < 0，返回 0（因为负贡献不会帮助父节点）
```

**为什么左右子树贡献要和 0 取 max？** 如果子树的贡献是负数，加上它只会让路径和更小，不如不加（取 0 表示不选这条路）。

**为什么 dfs 返回值要和 0 取 max？** 同样道理——如果经过当前节点向上走的最大路径和 < 0，那父节点不如不走这条路。

### 思考方式图解

```
     -10
     / \
    9   20
       / \
      15  7

递归过程（后序，从叶子往上）：

  9:  l=0, r=0 → ans=max(-∞, 9)=9, return max(0+9, 0)=9
  15: l=0, r=0 → ans=max(9, 15)=15, return max(0+15, 0)=15
  7:  l=0, r=0 → ans=max(15, 7)=15, return max(0+7, 0)=7
  20: l=15, r=7 → ans=max(15, 15+7+20=42)=42 ✅
      return max(15, 7)+20 = 35, 和 0 取 max → 35
  -10: l=9, r=35 → ans=max(42, 9+35+(-10)=34)=42
        return max(9,35)+(-10)=25, 和 0 取 max → 25

结果：42 ✅
```

### 代码实现

```java
class Solution {
    private int ans = Integer.MIN_VALUE;

    public int maxPathSum(TreeNode root) {
        dfs(root);
        return ans;
    }

    // 返回 node 对父节点的最大贡献值（经过 node 向上延伸的单边最大和）
    private int dfs(TreeNode node) {
        if (node == null) return 0;

        int lVal = dfs(node.left);    // 左子树最大贡献
        int rVal = dfs(node.right);   // 右子树最大贡献

        // 经过 node 的最大路径和（把 node 当作拐点）
        ans = Math.max(ans, lVal + rVal + node.val);

        // 向上返回：取较大的一边 + node.val，如果是负数就不贡献
        return Math.max(Math.max(lVal, rVal) + node.val, 0);
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 每个节点访问一次 |
| 🧠 空间复杂度 | **O(h)** — 递归栈深度，最坏 O(n) |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | DFS 后序遍历 + 全局最大值 |
| 算法类型 | 二叉树、DFS |
| 核心技巧 | **每个节点当拐点：最大路径和 = max(0, lVal) + max(0, rVal) + node.val；向上返回 max(max(l, r)+val, 0)** |
| 与 543 直径的区别 | 543 求边数（空节点返回 -1）；124 求和（空节点返回 0，负数贡献截断为 0） |
| 为什么和 0 取 max | 负数贡献只会拉低路径和，不如不选 |
| 全局变量初始值 | `Integer.MIN_VALUE` 而不是 0——因为节点值可能全为负数 |
| 关联题目 | [543. 二叉树的直径](day16-二叉树hot100-part2.md)——同样的后序 + 全局变量模板 |

### 一句话记住

> **「后序算贡献，负数截断为 0；拐点更新全局最大和。」**

---

*练习日期：2026-07-27*
