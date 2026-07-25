# Day 16 — 二叉树 Hot100（Part 2）

## 题目：二叉树的直径（Diameter of Binary Tree）

**LeetCode 543 | 二叉树 Hot100 | 难度：🟢 简单**

### 题目描述

给你一棵二叉树的根节点，返回该树的**直径**。

二叉树的直径是指树中任意两个节点之间最长路径的**长度**。这条路径可能经过也可能不经过根节点 `root`。

两个节点之间的路径长度由它们之间的边数表示。

### 示例

```
输入：root = [1,2,3,4,5]
      1
     / \
    2   3
   / \
  4   5
输出：3
解释：路径 [4,2,1,3] 或 [5,2,1,3] 的长度为 3

输入：root = [1,2]
输出：1
```

---

## 解法：DFS 后序遍历

### 思路

直径 = 左子树最大深度 + 右子树最大深度 = 经过的边数。

这里的关键是把「深度」的定义调整为**边数**而不是节点数：
- 空节点返回 `-1`（-1 + 1 = 0，表示空节点到父节点有 0 条边）
- 非空节点返回 `max(左子树边数, 右子树边数) + 1`

```
对于每个节点 node：
  左子树边数 = dfs(node.left) + 1
  右子树边数 = dfs(node.right) + 1
  经过 node 的直径 = 左子树边数 + 右子树边数
  更新全局最大值 ans
  返回 max(左子树边数, 右子树边数) 作为 node 向上的最大单边长度
```

**为什么空节点返回 -1？**

因为 `-1 + 1 = 0`——空节点到父节点有 0 条边，这样叶子节点的边数就是 `max(-1, -1) + 1 = 0 + 1 = 1`，表示叶子节点到父节点有 1 条边。符合物理意义。

### 思考方式图解

```
      1
     / \
    2   3
   / \
  4   5

递归过程（从叶子往上）：

  null: return -1

  4: llen=-1+1=0, rlen=-1+1=0 → ans=0, return max(0,0)=0
  5: llen=-1+1=0, rlen=-1+1=0 → ans=0, return max(0,0)=0
  2: llen=0+1=1, rlen=0+1=1 → ans=max(0,2)=2, return max(1,1)=1
  3: llen=-1+1=0, rlen=-1+1=0 → ans=2, return max(0,0)=0
  1: llen=1+1=2, rlen=0+1=1 → ans=max(2,3)=3 ✅, return max(2,1)=2

结果：3
```

### 代码实现

```java
class Solution {
    private int ans;

    public int diameterOfBinaryTree(TreeNode root) {
        dfs(root);
        return ans;
    }

    // 返回 node 的最大单边长度（边数）
    private int dfs(TreeNode node) {
        if (node == null) return -1;  // 空节点：-1 + 1 = 0 条边

        int llen = dfs(node.left) + 1;   // 左子树最大边数
        int rlen = dfs(node.right) + 1;  // 右子树最大边数

        ans = Math.max(ans, llen + rlen); // 经过 node 的直径

        return Math.max(llen, rlen);      // 返回最长的单边给父节点用
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 每个节点访问一次 |
| 🧠 空间复杂度 | **O(h)** — 递归栈深度 = 树高，最坏 O(n) |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | DFS 后序遍历（全局变量记录最大值） |
| 算法类型 | 二叉树、DFS |
| 核心技巧 | **空节点返回 -1，这样叶子到父节点的边数为 1；直径 = llen + rlen，取全局最大** |
| 为什么用 -1 | 深度按边数计算：空节点到父节点 0 条边，所以 `null + 1 = 0`，即 null 返回 -1 |
| 为什么用全局变量 | 直径可能出现在任意节点，不一定经过根节点，所以需要一个全局变量跟踪所有节点的直径 |
| 关联题目 | [104. 二叉树的最大深度](day15-二叉树hot100-part1.md)——最大深度是直径的基础 |

### 一句话记住

> **「空节点返回 -1，左右边数相加得直径——全局变量记录最大。」**

---

## 题目：二叉树的层序遍历（Binary Tree Level Order Traversal）

**LeetCode 102 | 二叉树 Hot100 | 难度：🟡 中等**

### 题目描述

给你二叉树的根节点 `root`，返回其节点值的**层序遍历**。（即逐层地，从左到右访问所有节点）。

### 示例

```
输入：root = [3,9,20,null,null,15,7]
      3
     / \
    9  20
      /  \
     15   7
输出：[[3],[9,20],[15,7]]

输入：root = [1]
输出：[[1]]

输入：root = []
输出：[]
```

---

## 解法：迭代 BFS（逐层遍历）

### 思路

层序遍历就是 **BFS（广度优先搜索）**——按层级从上到下，每层从左到右遍历。

核心操作：**维护一个当前层的列表 `cur`，遍历时收集值并构建下一层的列表 `next`**。

```
1. cur 初始化为 [root]
2. 遍历 cur 中的每个节点：
   - 收集节点值到 vals
   - 将左右子节点加入 next
3. vals 加入 ans
4. cur = next，继续下一轮
5. 直到 cur 为空
```

这种实现方式不需要显式的队列，用 List 天然区分了每层的边界。

### 思考方式图解

```
      3
     / \
    9  20
      /  \
     15   7

cur=[3]        → vals=[3], next=[9,20]     ans=[[3]]
cur=[9,20]     → vals=[9,20], next=[15,7]  ans=[[3],[9,20]]
cur=[15,7]     → vals=[15,7], next=[]      ans=[[3],[9,20],[15,7]]
cur=[] → 结束

结果：[[3],[9,20],[15,7]] ✅
```

### 代码实现

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        if (root == null) return List.of();

        List<List<Integer>> ans = new ArrayList<>();
        List<TreeNode> cur = List.of(root);        // 当前层节点列表

        while (!cur.isEmpty()) {
            List<TreeNode> next = new ArrayList<>();  // 下一层节点列表
            List<Integer> vals = new ArrayList<>(cur.size());  // 当前层值

            for (TreeNode node : cur) {
                vals.add(node.val);
                if (node.left != null) next.add(node.left);
                if (node.right != null) next.add(node.right);
            }

            cur = next;
            ans.add(vals);
        }

        return ans;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 每个节点访问一次 |
| 🧠 空间复杂度 | **O(n)** — 存储所有节点值，最坏情况满二叉树的最后一层有 n/2 个节点 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 迭代 BFS（层级列表法） |
| 算法类型 | 二叉树、BFS |
| 核心技巧 | **维护一个当前层列表 cur，遍历时构建下一层列表 next，天然区分层级边界** |
| 队列写法 | 也可以用 `Queue` 配合每层循环前获取 `size()` 来实现层序遍历 |
| 关联题目 | [107. 二叉树的层序遍历 II](https://leetcode.cn/problems/binary-tree-level-order-traversal-ii/)——从底部开始层序 |
| 易错点 | `root == null` 时返回空列表而不是 `null`；每层的 vals 要在循环开始时 new |

### 一句话记住

> **「当前层列表遍历完，下一层列表已建好——层层推进 BFS。」**

---

*练习日期：2026-07-25*
