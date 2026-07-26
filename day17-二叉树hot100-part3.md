# Day 17 — 二叉树 Hot100（Part 3）

## 题目：从前序与中序遍历序列构造二叉树（Construct Binary Tree from Preorder and Inorder Traversal）

**LeetCode 105 | 二叉树 Hot100 | 难度：🟡 中等**

### 题目描述

给定两个整数数组 `preorder` 和 `inorder`，其中 `preorder` 是二叉树的**先序遍历**，`inorder` 是同一棵树的**中序遍历**，请构造二叉树并返回其根节点。

### 示例

```
输入：preorder = [3,9,20,15,7], inorder = [9,3,15,20,7]
输出：[3,9,20,null,null,15,7]

      3
     / \
    9  20
      /  \
     15   7

输入：preorder = [-1], inorder = [-1]
输出：[-1]
```

---

## 解法：递归分治（先序定根，中序分左右）

### 思路

利用先序和中序的性质来递归构建：

```
先序遍历：  [根] [左子树先序] [右子树先序]
中序遍历：  [左子树中序] [根] [右子树中序]
```

**核心步骤：**

1. **先序的第一个元素就是根节点** `preorder[0]`
2. 在**中序**中找到根节点的位置 `leftsize` → 左边就是左子树，右边就是右子树
3. 由 `leftsize` 知道左子树的节点个数，从而将先序数组也分成左子树和右子树两部分
4. 递归构建左右子树

### 思考方式图解

```
preorder = [3, 9, 20, 15, 7]
inorder  = [9, 3, 15, 20, 7]

Step 1: 根 = preorder[0] = 3
Step 2: inorder 中 3 的下标 = 1 → leftsize = 1
         左子树中序 = [9]，右子树中序 = [15, 20, 7]
Step 3: 左子树先序 = pre[1..1] = [9]
         右子树先序 = pre[2..4] = [20, 15, 7]

递归左子树：pre=[9], in=[9]
  → 根 = 9, leftsize = 0
  → 左=null, 右=null, 返回 9

递归右子树：pre=[20, 15, 7], in=[15, 20, 7]
  → 根 = 20, inorder 中 20 的下标 = 1 → leftsize = 1
  → 左子树先序=[15], 中序=[15]
    → 根=15, 返回 15
  → 右子树先序=[7], 中序=[7]
    → 根=7, 返回 7
  → 返回 20(15, 7)

结果：
      3
     / \
    9  20
      /  \
     15   7 ✅
```

### 代码实现

```java
class Solution {
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        int n = preorder.length;
        if (n == 0) return null;

        // 先序第一个元素就是根节点
        int leftsize = indexOf(inorder, preorder[0]);  // 左子树节点数

        // 分割数组
        int[] pre1 = Arrays.copyOfRange(preorder, 1, 1 + leftsize);   // 左子树先序
        int[] pre2 = Arrays.copyOfRange(preorder, 1 + leftsize, n);   // 右子树先序
        int[] in1  = Arrays.copyOfRange(inorder, 0, leftsize);        // 左子树中序
        int[] in2  = Arrays.copyOfRange(inorder, 1 + leftsize, n);    // 右子树中序

        // 递归构建左右子树
        TreeNode left = buildTree(pre1, in1);
        TreeNode right = buildTree(pre2, in2);

        return new TreeNode(preorder[0], left, right);
    }

    // 在数组中找到目标值的下标（题目保证有解）
    private int indexOf(int[] a, int x) {
        for (int i = 0; ; i++) {
            if (a[i] == x) return i;
        }
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n²)** — 每次递归需要 O(n) 扫描中序找根 + 数组拷贝 |
| 🧠 空间复杂度 | **O(n²)** — 每次递归都 copyOfRange 创建新数组，总空间 O(n²) |

> **优化方向**：用 HashMap 缓存中序中每个值的下标（`Map<val, index>`），使查找 O(1)；用下标指针代替数组拷贝，可以将时间和空间都优化到 **O(n)**。

---

### 💡 优化：HashMap + 指针（灵茶山艾府，O(n)）

上述实现每次递归都要 `indexOf` 扫描中序数组，还要 `copyOfRange` 创建新数组——这两步可以优化掉：

```
1. HashMap 缓存中序值到下标的映射 → 查找 O(1)
2. 用下标指针（preL, preR, inL）代替数组拷贝 → 空间 O(n)
```

核心是把对数组的操作转化为对**下标区间**的操作，无需创建任何新数组。

```java
class Solution {
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        int n = preorder.length;
        // HashMap 预分配空间，缓存中序值 → 下标
        Map<Integer, Integer> index = HashMap.newHashMap(n);
        for (int i = 0; i < n; i++) {
            index.put(inorder[i], i);
        }
        // 左闭右开区间 [preL, preR), [inL, inR)
        return dfs(0, n, 0, preorder, index);
    }

    private TreeNode dfs(int preL, int preR, int inL,
                         int[] preorder, Map<Integer, Integer> index) {
        if (preL == preR) return null;  // 空区间

        int leftSize = index.get(preorder[preL]) - inL;  // 左子树大小 = 根在中序中的位置 - 中序左边界

        TreeNode left = dfs(preL + 1, preL + 1 + leftSize, inL, preorder, index);
        TreeNode right = dfs(preL + 1 + leftSize, preR, inL + 1 + leftSize, preorder, index);

        return new TreeNode(preorder[preL], left, right);
    }
}
```

**两种写法对比：**

| | 基础版（数组拷贝） | 优化版（HashMap + 指针） |
|---|-----------------|-----------------------|
| 查找根位置 | `indexOf` O(n) 扫描 | **HashMap.get O(1)** |
| 数组分割 | `copyOfRange` 创建新数组 | **下标指针，不创建新数组** |
| 时间复杂度 | O(n²) | **O(n)** |
| 空间复杂度 | O(n²) | **O(n)** |
| 代码可读性 | 直观，数组分割一目了然 | 需要理解指针区间的含义 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 递归分治 |
| 算法类型 | 二叉树、分治 |
| 核心技巧 | **先序定根，中序分左右——左子树大小由根在中序中的位置决定** |
| 基础写法 | copyOfRange 分割数组，逻辑直观，O(n²) |
| 优化写法 | HashMap 缓存下标 + 下标指针，**O(n)** |
| 关联题目 | [106. 从中序与后序遍历序列构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)——同样的思路，只是根从后序的最后一个取 |

### 一句话记住

> **「先序第一个是根，中序找到分左右——递归构建。」**

---

*练习日期：2026-07-26*
