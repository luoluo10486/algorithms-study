# Day 15 — 二叉树 Hot100

## 题目：二叉树的中序遍历（Binary Tree Inorder Traversal）

**LeetCode 94 | 二叉树 Hot100 | 难度：🟢 简单**

### 题目描述

给定一个二叉树的根节点 `root`，返回它的**中序**遍历。

### 示例

```
输入：root = [1,null,2,3]
输出：[1,3,2]

输入：root = []
输出：[]

输入：root = [1]
输出：[1]
```

---

## 解法：递归（DFS）

### 思路

中序遍历的访问顺序：**左子树 → 根节点 → 右子树**。

递归是最直观的写法，代码也最简洁。

```
中序遍历的顺序（以 root = [1,null,2,3] 为例）：

    1
     \
      2
     /
    3

遍历顺序：3 → 1 → 2
即：左(3) → 根(1) → 右子树中序(3→2)
```

### 代码实现

```java
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> ans = new ArrayList<>();
        dfs(ans, root);
        return ans;
    }

    private void dfs(List<Integer> ans, TreeNode node) {
        if (node == null) return;

        dfs(ans, node.left);   // 左
        ans.add(node.val);     // 根
        dfs(ans, node.right);  // 右
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 每个节点访问一次 |
| 🧠 空间复杂度 | **O(n)** — 递归调用栈深度 = 树高，最坏 O(n) |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 递归 DFS |
| 算法类型 | 二叉树、DFS |
| 核心技巧 | **左→根→右，递归时只需调换三行代码的位置即可得到前序/中序/后序** |
| 三种遍历顺序 | 前序：根→左→右 / 中序：左→根→右 / 后序：左→右→根 |
| 迭代写法 | 二叉树遍历也可以用栈模拟递归，后面会补充 |

### 一句话记住

> **「左根右，递归深搜——调换三行代码就是前序后序。」**

---

## 题目：二叉树的最大深度（Maximum Depth of Binary Tree）

**LeetCode 104 | 二叉树 Hot100 | 难度：🟢 简单**

### 题目描述

给定一个二叉树，找出其最大深度。

二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。

### 示例

```
输入：root = [3,9,20,null,null,15,7]
    3
   / \
  9  20
    /  \
   15   7
输出：3

输入：root = [1,null,2]
输出：2
```

---

## 解法：递归 DFS（分治）

### 思路

二叉树的最大深度 = **max(左子树最大深度, 右子树最大深度) + 1**。

这是一个典型的分治问题：

```
递归公式：
  maxDepth(node) = 0,                       if node == null
  maxDepth(node) = max(maxDepth(left), maxDepth(right)) + 1,  otherwise
```

因为不需要关注具体的遍历顺序，只需要知道左右子树各自的深度，然后取最大值加 1 即可。

### 思考方式图解

```
    3 (root)
   / \
  9  20
    /  \
   15   7

计算过程（从叶子往上）：

null: depth=0
15:   max(0,0)+1 = 1
7:    max(0,0)+1 = 1
9:    max(0,0)+1 = 1
20:   max(1,1)+1 = 2
3:    max(1,2)+1 = 3 ✅
```

### 代码实现

```java
class Solution {
    public int maxDepth(TreeNode root) {
        if (root == null) return 0;

        int ldepth = maxDepth(root.left);   // 左子树深度
        int rdepth = maxDepth(root.right);  // 右子树深度
        return Math.max(ldepth, rdepth) + 1; // 当前节点深度
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
| 算法名称 | 递归 DFS（分治） |
| 算法类型 | 二叉树、分治 |
| 核心技巧 | **当前节点深度 = max(左子树深度, 右子树深度) + 1——分治思想** |
| 终止条件 | 空节点深度为 0 |
| 层序遍历 | 也可以用 BFS 层序遍历求层数来得到最大深度 |

### 一句话记住

> **「左右子树取最大加一——递归到空返回 0。」**

---

## 题目：翻转二叉树（Invert Binary Tree）

**LeetCode 226 | 二叉树 Hot100 | 难度：🟢 简单**

### 题目描述

给你一棵二叉树的根节点 `root`，翻转这棵二叉树，并返回其根节点。

### 示例

```
输入：root = [4,2,7,1,3,6,9]
       4
     /   \
    2     7
   / \   / \
  1   3 6   9

输出：
       4
     /   \
    7     2
   / \   / \
  9   6 3   1

输入：root = [2,1,3]
输出：[2,3,1]
```

---

## 解法：递归（DFS）

### 思路

翻转二叉树的核心操作就是**交换左右子树**——每个节点都要交换它的左右子节点。

```
对于每个节点 node：
  1. 翻转左子树（递归）
  2. 翻转右子树（递归）
  3. 交换左右子节点
```

这个过程从根节点开始一直递归到叶子，最终整棵树被完全翻转。

### 思考方式图解

```
初始：
       4
     /   \
    2     7
   / \   / \
  1   3 6   9

递归到最底层：
  1: node=1 → 左右都 null，交换无变化，返回 1
  3: node=3 → 同理，返回 3
  2: left=1, right=3 → 交换后 2 的左右为 3 和 1

  6: 返回 6
  9: 返回 9
  7: left=6, right=9 → 交换后 7 的左右为 9 和 6

  4: left=翻转后的 2(3,1), right=翻转后的 7(9,6) → 交换左右

结果：
       4
     /   \
    7     2
   / \   / \
  9   6 3   1 ✅
```

### 代码实现

```java
class Solution {
    public TreeNode invertTree(TreeNode root) {
        if (root == null) return null;

        // 递归翻转左右子树
        TreeNode left = invertTree(root.left);
        TreeNode right = invertTree(root.right);

        // 交换左右子节点
        root.left = right;
        root.right = left;

        return root;
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
| 算法名称 | 递归 DFS |
| 算法类型 | 二叉树、分治 |
| 核心技巧 | **递归翻转左右子树，然后交换它们——每个节点都要交换左右孩子** |
| 关联题目 | [101. 对称二叉树](https://leetcode.cn/problems/symmetric-tree/)——翻转的对称版本 |
| 同样的问题 | 这个题目曾让 Homebrew 的作者 Max Howell 在 Google 面试中挂掉，因此成名 |

### 一句话记住

> **「递归翻转左右子树，然后交换——翻转二叉树就这么简单。」**

---

## 题目：对称二叉树（Symmetric Tree）

**LeetCode 101 | 二叉树 Hot100 | 难度：🟢 简单**

### 题目描述

给你一个二叉树的根节点 `root`，检查它是否轴对称。

### 示例

```
输入：root = [1,2,2,3,4,4,3]
      1
     / \
    2   2
   / \ / \
  3  4 4  3
输出：true

输入：root = [1,2,2,null,3,null,3]
      1
     / \
    2   2
     \   \
      3   3
输出：false
```

---

## 解法：递归比较对称节点

### 思路

判断一棵树是否对称，本质上就是**比较左子树和右子树是否是镜像对称的**。

```
对于两个节点 p（左子树）和 q（右子树），它们对称的条件：
  1. p.val == q.val
  2. p.left 与 q.right 对称
  3. p.right 与 q.left 对称
```

换句话说，**左子树的左子节点 和 右子树的右子节点 比较；左子树的右子节点 和 右子树的左子节点 比较**——两边交叉对比。

### 思考方式图解

```
      1
     / \
    2   2
   / \ / \
  3  4 4  3

比较过程：
  isSample(2(left), 2(right))
    → 2==2 ✅
    → isSample(3(left的left), 3(right的right)): 3==3 ✅
    → isSample(4(left的right), 4(right的left)): 4==4 ✅
    → true ✅
```

```
      1
     / \
    2   2
     \   \
      3   3

比较过程：
  isSample(2(left), 2(right))
    → 2==2 ✅
    → isSample(null(left的left), 3(right的right)): null != 3 ❌
    → false ❌
```

### 代码实现

```java
class Solution {
    public boolean isSymmetric(TreeNode root) {
        // 比较左子树和右子树是否镜像对称
        return isSample(root.left, root.right);
    }

    private boolean isSample(TreeNode p, TreeNode q) {
        // 如果有一个为空，则必须两个都为空才对称
        if (p == null || q == null) {
            return p == q;
        }
        // 当前值相等 + 左左与右右对称 + 左右与右左对称
        return p.val == q.val
            && isSample(p.left, q.right)
            && isSample(p.right, q.left);
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
| 算法名称 | 递归比较对称节点 |
| 算法类型 | 二叉树、递归 |
| 核心技巧 | **`p.left` 与 `q.right` 比较，`p.right` 与 `q.left` 比较——交叉对称** |
| 与翻转二叉树的关系 | 翻转二叉树（226）是把整棵树翻过来，对称二叉树（101）是检查翻过来是否一样 |
| 边界处理 | `if (p==null \|\| q==null) return p==q` — 一个为 null 时不直接返回 false，而是判断是否都为 null |

### 一句话记住

> **「左右子树交叉比较——左对右、右对左，值相等且都对称。」**

---

*练习日期：2026-07-24*
