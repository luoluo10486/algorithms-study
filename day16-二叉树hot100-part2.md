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

## 解法：迭代 BFS（队列）

### 思路

层序遍历就是 **BFS（广度优先搜索）**——按层级从上到下，每层从左到右遍历。

核心操作：**用队列逐层处理，每轮开始时先记录当前队列大小 n，然后一次处理 n 个节点，此时队列中剩下的就是下一层的所有节点。**

```
1. 根节点入队
2. 每轮循环开始前，记录当前队列大小 n（= 当前层节点数）
3. 循环 n 次出队：
   - 收集节点值到 vals
   - 左右子节点入队（为下一层做准备）
4. vals 加入 ans
5. 继续下一轮，直到队列为空
```

利用 `int n = q.size()` 在循环开始时固定当前层的节点数，后续入队的下一层节点不会干扰本次循环。

### 思考方式图解

```
      3
     / \
    9  20
      /  \
     15   7

q=[3]           → n=1, poll 3 → vals=[3], 入队 9,20        ans=[[3]]
q=[9,20]        → n=2, poll 9 → vals=[9], 入队 null
                        poll 20 → vals=[9,20], 入队 15,7   ans=[[3],[9,20]]
q=[15,7]        → n=2, poll 15 → vals=[15], 入队 null
                        poll 7  → vals=[15,7], 入队 null   ans=[[3],[9,20],[15,7]]
q=[] → 结束

结果：[[3],[9,20],[15,7]] ✅
```

### 代码实现

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        if (root == null) return List.of();

        List<List<Integer>> ans = new ArrayList<>();
        Queue<TreeNode> q = new ArrayDeque<>();
        q.add(root);

        while (!q.isEmpty()) {
            int n = q.size();                     // 当前层的节点数
            List<Integer> vals = new ArrayList<>(n);

            while (n-- > 0) {                     // 逐一出队当前层节点
                TreeNode node = q.poll();
                vals.add(node.val);
                if (node.left != null) q.add(node.left);
                if (node.right != null) q.add(node.right);
            }

            ans.add(vals);
        }

        return ans;
    }
}
```

> **其他写法**：也可以使用双列表（cur/next）实现，无需队列，原理相同。

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 每个节点访问一次 |
| 🧠 空间复杂度 | **O(n)** — 队列最多存储一层的全部节点，最坏情况满二叉树的最后一层有 n/2 个节点 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 迭代 BFS（队列） |
| 算法类型 | 二叉树、BFS |
| 核心技巧 | **每轮先用 `int n = q.size()` 固定当前层的节点数，n 个节点处理完就是下一层** |
| 另一种写法 | 双列表法（cur/next）——不使用队列，用两个 ArrayList 交替，原理相同 |
| 关联题目 | [107. 二叉树的层序遍历 II](https://leetcode.cn/problems/binary-tree-level-order-traversal-ii/)——从底部开始层序 |
| 易错点 | `root == null` 时返回空列表而不是 `null`；`n-- > 0` 是先比较再自减 |

### 一句话记住

> **「队列逐层处理，size 固定边界——n 个节点就是一层。」**

---

## 题目：将有序数组转换为二叉搜索树（Convert Sorted Array to Binary Search Tree）

**LeetCode 108 | 二叉树 Hot100 | 难度：🟢 简单**

### 题目描述

给你一个整数数组 `nums`，其中元素已经按**升序**排列，请你将其转换为一棵**高度平衡**二叉搜索树。

高度平衡二叉树是指一个二叉树每个节点的左右两个子树的高度差的绝对值不超过 1。

### 示例

```
输入：nums = [-10,-3,0,5,9]
输出：[0,-3,9,-10,null,5]  （或其它有效答案）

        0
       / \
     -3   9
     /   /
   -10  5

输入：nums = [1,3]
输出：[3,1]  或  [1,3]
```

---

## 解法：递归分治（始终选中间元素为根）

### 思路

二叉搜索树（BST）的性质：**左子树 < 根 < 右子树**。

有序数组本身就是 BST 的**中序遍历结果**。要构建一棵高度平衡的 BST，策略就是**每次取中间元素作为根节点，左半部分构建左子树，右半部分构建右子树**。

```
递归公式：
  dfs(nums, left, right):
    如果 left == right → 空区间，返回 null
    取中间下标 m = (left + right) / 2
    创建根节点: TreeNode(nums[m])
    递归构建左子树: dfs(nums, left, m)
    递归构建右子树: dfs(nums, m + 1, right)
    返回根节点
```

**为什么取中间能保证平衡？** 因为左右子树的元素数量最多相差 1，构建出来的树就是高度平衡的。

### 思考方式图解

```
nums = [-10, -3, 0, 5, 9]

dfs([-10,-3,0,5,9], 0, 5):
  m = (0+5)/2 = 2 → root = 0
  left = dfs(nums, 0, 2)  → [-10, -3]
    m = (0+2)/2 = 1 → root = -3
    left = dfs(nums, 0, 1)  → [-10]
      m = (0+1)/2 = 0 → root = -10
      left  = dfs(nums, 0, 0) = null
      right = dfs(nums, 1, 1) = null
      return -10
    right = dfs(nums, 2, 2) = null
    return -3
  right = dfs(nums, 3, 5) → [5, 9]
    m = (3+5)/2 = 4 → root = 5
    left = dfs(nums, 3, 4) = null
    right = dfs(nums, 5, 5)  → [9]
      m = 5 → root = 9, 左右都 null
      return 9
    return 5
  return 0

结果：
        0
       / \
     -3   5
     /     \
   -10      9
```

### 代码实现

```java
class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        return dfs(nums, 0, nums.length);  // 左闭右开 [left, right)
    }

    // 构建 nums[left..right-1] 区间对应的 BST
    private TreeNode dfs(int[] nums, int left, int right) {
        if (left == right) return null;    // 空区间

        int m = (left + right) / 2;        // 中间元素作为根（左闭右开时 m 偏左）
        return new TreeNode(
            nums[m],
            dfs(nums, left, m),            // 左子树：左半区间
            dfs(nums, m + 1, right)        // 右子树：右半区间
        );
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 每个元素恰好构建一个节点 |
| 🧠 空间复杂度 | **O(log n)** — 递归栈深度 = 平衡树高，log n |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 递归分治（二分构建） |
| 算法类型 | 二叉树、BST、分治 |
| 核心技巧 | **始终取中间元素为根，左半部分递归建左子树，右半部分递归建右子树——保证平衡** |
| 区间定义 | 左闭右开 `[left, right)`，当 `left == right` 时区间为空 |
| 中间下标 | `m = (left + right) / 2`，在左闭右开区间下 m 偏左（左子树的元素数 ≥ 右子树） |
| 易错点 | 递归右子树时是 `dfs(nums, m + 1, right)` 不是 `dfs(nums, m, right)`——否则会死循环 |

### 一句话记住

> **「有序数组建 BST，取中为根递归分两半——平衡自然来。」**

---

## 题目：验证二叉搜索树（Validate Binary Search Tree）

**LeetCode 98 | 二叉树 Hot100 | 难度：🟡 中等**

### 题目描述

给你一个二叉树的根节点 `root`，判断其是否是一个有效的二叉搜索树（BST）。

**有效 BST 定义如下：**

- 节点的左子树只包含**小于**当前节点的数
- 节点的右子树只包含**大于**当前节点的数
- 所有左子树和右子树自身必须也是二叉搜索树

### 示例

```
输入：root = [2,1,3]
      2
     / \
    1   3
输出：true

输入：root = [5,1,4,null,null,3,6]
      5
     / \
    1   4
      / \
     3   6
输出：false
解释：根节点的值是 5，但右子节点的值是 4，不满足 BST 定义
```

---

## 解法：递归 + 上下界约束

### 思路

BST 的核心性质：**左子树的所有节点 < 根节点 < 右子树的所有节点**。

这个性质可以转化为**上下界约束**——在递归遍历时，给每个节点传递一个允许的取值范围 `(left, right)`：

```
对于根节点，取值范围是 (-∞, +∞)
左子节点：取值范围是 (-∞, 父节点.val)
右子节点：取值范围是 (父节点.val, +∞)
```

**为什么用 `long` 而不是 `int`？**因为节点的值可能是 `Integer.MIN_VALUE` 或 `Integer.MAX_VALUE`，如果上下界用 `int`，边界上的比较会出问题。用 `long` + `MIN_VALUE/MAX_VALUE` 保证上下界永远不会被节点的值突破。

### 思考方式图解

```
      5
     / \
    1   4
        / \
       3   6

验证过程：

node=5:  范围 (-∞, +∞)    → 5 在范围内 ✅
  node=1: 范围 (-∞, 5)    → 1 在范围内 ✅
    node=null → true
  node=4: 范围 (5, +∞)    → 4 不在范围内 ❌ → false

结果：false ✅
```

```
      2
     / \
    1   3

验证过程：

node=2:  范围 (-∞, +∞)    → 2 在范围内 ✅
  node=1: 范围 (-∞, 2)    → 1 在范围内 ✅
    node=null → true
  node=3: 范围 (2, +∞)    → 3 在范围内 ✅
    node=null → true

结果：true ✅
```

### 代码实现

```java
class Solution {
    public boolean isValidBST(TreeNode root) {
        return isValidBST(root, Long.MIN_VALUE, Long.MAX_VALUE);
    }

    // 检查 node 是否在 (left, right) 范围内，且左右子树也满足 BST
    private boolean isValidBST(TreeNode node, long left, long right) {
        if (node == null) return true;

        long x = node.val;
        // 当前节点必须在 (left, right) 范围内
        // 左子树必须在 (left, x) 范围内
        // 右子树必须在 (x, right) 范围内
        return x > left && x < right
            && isValidBST(node.left, left, x)
            && isValidBST(node.right, x, right);
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
| 算法名称 | 递归 + 上下界约束 |
| 算法类型 | 二叉树、BST |
| 核心技巧 | **给每个节点传递一个允许的取值范围 (left, right)，左子树收紧右边界，右子树收紧左边界** |
| 为什么用 long | 节点的值可能等于 `Integer.MAX_VALUE`，用 `int` 时上界 `Integer.MAX_VALUE` 会被节点的值突破 |
| 易错点 | 不能只检查 `node.left.val < node.val < node.right.val`——这只能保证局部有序，不能保证全局（如上例 5→4→3） |

### 一句话记住

> **「上下界约束传递——左子树收紧右界，右子树收紧左界，层层递归验证。」**

---

## 题目：二叉搜索树中第 K 小的元素（Kth Smallest Element in a BST）

**LeetCode 230 | 二叉树 Hot100 | 难度：🟡 中等**

### 题目描述

给定一个二叉搜索树的根节点 `root`，和一个整数 `k`，请你设计一个算法查找其中第 `k` 小的元素（从 1 开始计数）。

### 示例

```
输入：root = [3,1,4,null,2], k = 1
      3
     / \
    1   4
     \
      2
输出：1

输入：root = [5,3,6,2,4,null,null,1], k = 3
          5
         / \
        3   6
       / \
      2   4
     /
    1
输出：3
```

---

## 解法：中序遍历（BST 的中序遍历是升序序列）

### 思路

BST 最核心的性质：**中序遍历得到的是升序序列**。那么第 k 小的元素就是中序遍历过程中访问的第 k 个节点。

利用这个性质，可以在遍历过程中用一个计数器 `k` 进行递减：

```
中序遍历（左→根→右）：
  每访问一个节点，k--
  当 k == 0 时，当前节点就是第 k 小的元素
```

**剪枝优化**：当 `k <= 0` 时不再继续递归，提前终止。

### 思考方式图解

```
      3
     / \
    1   4
     \
      2

中序遍历顺序：1 → 2 → 3 → 4

k=3 时的遍历过程：

dfs(3): → dfs(left=1)
  dfs(1): → dfs(left=null)
            k=3 → --k=2 → 还没到
            → dfs(right=2)
              dfs(2): → dfs(left=null)
                        k=2 → --k=1 → 还没到
                        → dfs(right=null)
              dfs(2) 结束
  dfs(1) 结束
  k=1 → --k=0 → ans=3 ✅   ← 找到了
  dfs(right=4): k=0 → return（剪枝）
```

### 代码实现

```java
class Solution {
    private int ans;
    private int k;

    public int kthSmallest(TreeNode root, int k) {
        this.k = k;
        dfs(root);
        return ans;
    }

    // 中序遍历，访问一个节点就 k--，当 k=0 时记录答案
    private void dfs(TreeNode node) {
        if (node == null || k <= 0) return;  // k<=0 时剪枝

        dfs(node.left);
        if (--k == 0) {                       // 访问当前节点
            ans = node.val;
        }
        dfs(node.right);
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(k)** — 遍历到第 k 个节点就停止（最坏 O(n)） |
| 🧠 空间复杂度 | **O(h)** — 递归栈深度，最坏 O(n) |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 中序遍历 + 计数器 |
| 算法类型 | BST、DFS |
| 核心技巧 | **BST 中序 = 升序，第 k 步访问的节点就是第 k 小——用 `k--` 计数，`k==0` 时记录答案** |
| 剪枝条件 | `k <= 0` 提前终止递归，避免无用遍历 |
| 关联题目 | [98. 验证二叉搜索树](day16-二叉树hot100-part2.md)——BST 的基本性质 |

### 一句话记住

> **「BST 中序是升序，第 k 步就是第 k 小——`k--` 到 0 时记录。」**

---

## 题目：二叉树的右视图（Binary Tree Right Side View）

**LeetCode 199 | 二叉树 Hot100 | 难度：🟡 中等**

### 题目描述

给定一个二叉树的根节点 `root`，想象自己站在它的右侧，按照从顶部到底部的顺序，返回从右侧所能看到的节点值。

### 示例

```
输入：root = [1,2,3,null,5,null,4]
      1
     / \
    2   3
     \   \
      5   4
输出：[1,3,4]

输入：root = [1,null,3]
输出：[1,3]

输入：root = []
输出：[]
```

---

## 解法：DFS（根→右→左，每层第一个遇到的节点）

### 思路

右视图就是从右边看每层最右边的节点。但更巧妙的做法是：**按「根→右→左」的顺序遍历，每层第一个访问到的节点就是该层最右边的节点**。

```
关键观察：
  如果先访问右子树再访问左子树，那么每层第一个被访问的节点一定是最右侧的节点
```

实现时用一个 `depth` 跟踪当前深度，当 `depth == ans.size()` 时，说明当前层还没有节点被记录过——那么当前节点就是该层第一个被访问的节点，也就是右视图中的节点。

### 思考方式图解

```
      1
     / \
    2   3
     \   \
      5   4

遍历顺序：根 → 右 → 左

dfs(1, depth=0): ans.size=0 → depth==ans.size → ans=[1]
  → dfs(3, depth=1): ans.size=1 → depth==ans.size → ans=[1,3]
    → dfs(4, depth=2): ans.size=2 → depth==ans.size → ans=[1,3,4]
      → dfs(null) → return
      → dfs(null) → return
    → dfs(null) → return
  → dfs(2, depth=1): ans.size=3 → depth(1) != ans.size(3) → 不记录
    → dfs(null) → return
    → dfs(5, depth=2): ans.size=3 → depth(2) != ans.size(3) → 不记录
      → dfs(null) → return
      → dfs(null) → return

结果：[1,3,4] ✅
```

### 代码实现

```java
class Solution {
    public List<Integer> rightSideView(TreeNode root) {
        List<Integer> ans = new ArrayList<>();
        dfs(root, 0, ans);
        return ans;
    }

    // 按 根→右→左 的顺序遍历，每层第一个访问的节点就是右视图节点
    private void dfs(TreeNode root, int depth, List<Integer> ans) {
        if (root == null) return;

        // 当前层还没有记录过 → 当前节点就是该层最右侧的节点
        if (depth == ans.size()) {
            ans.add(root.val);
        }

        dfs(root.right, depth + 1, ans);   // 先遍历右子树
        dfs(root.left, depth + 1, ans);    // 再遍历左子树
    }
}
```

---

### 💡 BFS 写法：层序遍历取每层最后一个

如你所说，它本质上就是**层序遍历（第 102 题）**，只是多了一步——取当前层的最后一个节点加入答案。

用 `cur/next` 双列表进行层序遍历，每层遍历完后，`cur` 中的最后一个节点就是该层最右侧的节点：

```java
class Solution {
    public List<Integer> rightSideView(TreeNode root) {
        if (root == null) return List.of();

        List<Integer> ans = new ArrayList<>();
        List<TreeNode> cur = List.of(root);

        while (!cur.isEmpty()) {
            // 当前层最后一个节点就是右视图节点
            ans.add(cur.getLast().val);

            List<TreeNode> nxt = new ArrayList<>();
            for (TreeNode node : cur) {
                if (node.left != null) nxt.add(node.left);
                if (node.right != null) nxt.add(node.right);
            }
            cur = nxt;
        }

        return ans;
    }
}
```

**两种写法对比：**

| | DFS 根右左 | BFS 层序取最后 |
|---|-----------|--------------|
| 核心技巧 | 每层第一个遇到的节点就是右侧节点 | 每层最后一个节点就是右侧节点 |
| 遍历顺序 | 根 → 右 → 左 | 左 → 右（层序） |
| 空间 | O(h) 递归栈 | O(n) 队列 |
| 代码量 | 更简洁 | 更直观 |

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 每个节点访问一次 |
| 🧠 空间复杂度 | **O(h)** — 递归栈深度，最坏 O(n) |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | DFS + 根右左遍历 |
| 算法类型 | 二叉树、DFS |
| 核心技巧 | **按「根→右→左」顺序遍历，每层第一个遇到的节点就是右视图节点** |
| BFS 写法 | 层序遍历取每层最后一个节点（`cur.getLast()`）——本质是 102 题的基础上加一步 |
| 两种写法的关系 | DFS 版（根右左）和 BFS 版（层序取最后）是同一问题的两种视角 |
| 关联题目 | [102. 二叉树的层序遍历](day16-二叉树hot100-part2.md)——BFS 层序基础 |
| 易错点 | 不是直接取最右叶子，而是每层**第一个**被访问到的节点——先右后左保证了它就是最右侧的节点 |

### 一句话记住

> **「根右左遍历，每层第一个就是右视图。」**

---

## 题目：二叉树展开为链表（Flatten Binary Tree to Linked List）

**LeetCode 114 | 二叉树 Hot100 | 难度：🟡 中等**

### 题目描述

给你二叉树的根节点 `root`，请你将它展开为一个单链表：

- 展开后的单链表应该同样使用 `TreeNode`，其中 `right` 子指针指向链表中下一个节点，而左子指针始终为 `null`
- 展开后的单链表应该与二叉树**先序遍历**顺序相同

### 示例

```
输入：root = [1,2,5,3,4,null,6]
      1
     / \
    2   5
   / \   \
  3   4   6

输出：[1,null,2,null,3,null,4,null,5,null,6]
链式结构：
  1 → 2 → 3 → 4 → 5 → 6

输入：root = []
输出：[]

输入：root = [0]
输出：[0]
```

---

## 解法：反向先序遍历（右→左→根）

### 思路

题目要求展开后的顺序与**先序遍历**相同，即：**根 → 左 → 右**。

常规思路是先序遍历保存到列表再重建链表，但更优雅的做法是**反向思考**——从最后一个节点开始向前构建链表：

```
反向先序遍历（右→左→根）：
  1. flatten(root.right)  ← 先处理右子树
  2. flatten(root.left)   ← 再处理左子树
  3. 当前节点：
     - 左指针置 null
     - 右指针指向上一步处理好的链表头 head
     - 更新 head = 当前节点
```

**为什么反向先序遍历能奏效？**

先序遍历的顺序是 `根 → 左 → 右`，反转过来就是 `右 → 左 → 根`。按这个顺序处理，每次把当前节点的右指针指向已经构建好的链表头部，就自然得到了先序顺序的链表。

> 可以理解为：**从链表的尾部开始往前构建，后处理的节点指向先处理好的链表头。**

### 思考方式图解

```
      1
     / \
    2   5
   / \   \
  3   4   6

反向先序遍历（右→左→根）过程：

处理 6:  head=null → root.right=null, root.left=null, head=6
处理 5:  head=6    → root.right=6, root.left=null, head=5
处理 4:  head=5    → root.right=5, root.left=null, head=4
处理 3:  head=4    → root.right=4, root.left=null, head=3
处理 2:  head=3    → root.right=3, root.left=null, head=2
处理 1:  head=2    → root.right=2, root.left=null, head=1

最终链表：1 → 2 → 3 → 4 → 5 → 6 ✅
```

### 代码实现

```java
class Solution {
    private TreeNode head;  // 已构建好的链表头（从尾部开始）

    public void flatten(TreeNode root) {
        if (root == null) return;

        // 反向先序遍历：右 → 左 → 根
        flatten(root.right);
        flatten(root.left);

        // 当前节点的左指针置 null，右指针指向已构建的链表头
        root.left = null;
        root.right = head;
        head = root;         // 更新 head
    }
}
```

---

### 💡 迭代写法：左子树的最右节点指向右子树（O(1) 空间）

上述递归解法需要 O(h) 的递归栈空间。下面这个迭代解法在**原地展开，空间 O(1)**：

核心思想：

```
对于当前节点 cur：
  1. 如果 cur 有左子树：
     - 找到左子树的最右节点（前驱节点 predecessor）
     - 将最右节点的右指针指向 cur 的右子树
     - 将 cur 的左子树移到右边，左指针置 null
  2. cur = cur.right，继续处理下一个节点
```

本质上是**把左子树的最右节点接到右子树上，然后把左子树整体移到右边**——这样左子树的节点就插在了根节点和原右子树之间，符合先序顺序。

```java
class Solution {
    public void flatten(TreeNode root) {
        TreeNode cur = root;
        while (cur != null) {
            if (cur.left != null) {
                // 左子树存在 → 找到左子树的最右节点
                TreeNode next = cur.left;         // 左子树的根
                TreeNode a = next;
                while (a.right != null) {
                    a = a.right;                   // 走到左子树的最右节点
                }
                // 最右节点的右指针指向 cur 的右子树
                a.right = cur.right;
                // 把左子树移到右边
                cur.left = null;
                cur.right = next;
            }
            cur = cur.right;
        }
    }
}
```

两种写法对比：

| | 递归（反向先序） | 迭代（左子树最右→右子树） |
|---|--------------|----------------------|
| 空间 | O(h) 递归栈 | **O(1)** |
| 思路 | 从尾部向前构建链表 | 原地调整指针，将左子树插入根和右子树之间 |
| 代码量 | 简洁 | 略多但容易理解 |

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 每个节点访问一次 |
| 🧠 空间复杂度 | **O(h)** — 递归栈深度，最坏 O(n) |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 反向先序遍历（全局指针法） |
| 算法类型 | 二叉树、DFS |
| 核心技巧 | **按「右→左→根」反向先序遍历，从尾部向头部构建链表——当前节点的右指针指向已构建的链表头** |
| 为什么是反的 | 先序 = 根→左→右，从最后一个节点开始构建最自然，所以反向先序 = 右→左→根 |
| 迭代写法（O(1) 空间） | 左子树最右节点 → 右子树，然后将左子树整体移到右边 |
| 关联题目 | [199. 二叉树的右视图](day16-二叉树hot100-part2.md)——也是右→左遍历的思路 |
| 易错点 | 必须先 `flatten(root.right)` 再 `flatten(root.left)`——顺序反了会导致链表顺序错误；处理完要记得设 `root.left = null` |

### 一句话记住

> **「反向先序遍历，从尾部往前构建——右指针指向已处理好的链表头。」**

---

*练习日期：2026-07-25*
