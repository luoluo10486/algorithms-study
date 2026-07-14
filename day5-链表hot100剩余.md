# Day 5 — 链表 Hot 100 剩余 + 哈希表

## 链表部分

---

## 题目：排序链表（Sort List）

**LeetCode 148 | Hot 100 | 难度：🟡 中等**

### 题目描述

给你链表的头结点 `head`，请将其按**升序**排列并返回排序后的链表。

> 进阶：在 **O(n log n)** 时间复杂度和**常数级**空间复杂度下完成。

### 示例

```
输入：head = [4,2,1,3]
输出：[1,2,3,4]

输入：head = [-1,5,3,4,0]
输出：[-1,0,3,4,5]

输入：head = []
输出：[]
```

---

## 解法：归并排序（自顶向下分治）

### 思路

数组排序的常用做法是快速排序（因为数组支持 O(1) 随机访问），但链表不行——快排的 partition 操作在链表上效率低下。

链表排序的最佳选择是**归并排序**，天然适配链表的特性：

1. **分割**：用快慢指针找中点，把链表一切为二
2. **递归**：对左右两半分别排序
3. **合并**：用合并两个有序链表的方式把排好序的两半合起来

整个过程就是「分治」三部曲——分、治、合。

### 为什么归并排序适合链表？

| 对比 | 数组 | 链表 |
|------|------|------|
| 访问方式 | 随机访问 O(1) | 顺序访问 O(n) |
| 快排 partition | 双向扫描，高效 | 只能单向，且不稳定 |
| 归并拆分 | 需要 O(n) 辅助空间 | 快慢指针原地拆分，**O(1)** |
| 归并合并 | 需要 O(n) 辅助空间 | 只需改指针指向，**O(1)** |

链表归并的额外空间 = 递归调用栈 O(log n)，比数组归并的 O(n) 辅助数组更省。

### 思考方式图解

```
head = [4, 2, 1, 3]

分治过程：

               [4, 2, 1, 3]
              /             \
          [4, 2]           [1, 3]
         /      \         /      \
       [4]      [2]     [1]      [3]
         \      /         \      /
          [2, 4]           [1, 3]
              \             /
               [1, 2, 3, 4]
```

### 代码实现

```java
class Solution {
    public ListNode sortList(ListNode head) {
        // 递归终止：空链表或只有一个节点
        if (head == null || head.next == null) {
            return head;
        }

        // Step 1: 找中点，断开链表
        ListNode head2 = middleNode(head);

        // Step 2: 递归排序左右两半
        head = sortList(head);
        head2 = sortList(head2);

        // Step 3: 合并两个有序链表
        return mergeTwoLists(head, head2);
    }

    // 876. 链表的中间结点 — 快慢指针
    private ListNode middleNode(ListNode head) {
        ListNode pre = head;
        ListNode slow = head;
        ListNode fast = head;
        while (fast != null && fast.next != null) {
            pre = slow;          // 记录 slow 的前一个节点
            slow = slow.next;
            fast = fast.next.next;
        }
        pre.next = null;         // ★ 断开连接，分成两个独立链表
        return slow;
    }

    // 21. 合并两个有序链表 — 双指针 + dummy
    private ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        ListNode dummy = new ListNode();
        ListNode cur = dummy;
        while (list1 != null && list2 != null) {
            if (list1.val < list2.val) {
                cur.next = list1;
                list1 = list1.next;
            } else {
                cur.next = list2;
                list2 = list2.next;
            }
            cur = cur.next;
        }
        cur.next = list1 != null ? list1 : list2;
        return dummy.next;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n log n)** — 归并排序的时间复杂度 |
| 🧠 空间复杂度 | **O(log n)** — 递归调用栈深度 |

### 核心细节

`middleNode` 中最关键的一行是 `pre.next = null`——它把左半边的末尾断开，否则合并时左右链表会互相干扰，导致死循环。

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 归并排序（自顶向下分治） |
| 算法类型 | 链表、分治、排序 |
| 核心技巧 | **快慢指针找中点断开 + 递归排序 + 合并两个有序链表** |
| 适用场景 | 链表排序（O(n log n)） |
| 前置基础 | LeetCode 876 链表中点 + LeetCode 21 合并有序链表 |
| 易错点 | 找中点后记得 `pre.next = null` 断开；递归基是 `head == null || head.next == null` |

### 一句话记住

> **「中点断开再递归，合并归来已有序。」**

---

*练习日期：2026-07-13*

---

---

## 题目：合并 K 个升序链表（Merge k Sorted Lists）

**LeetCode 23 | Hot 100 | 难度：🔴 困难**

### 题目描述

给你一个链表数组，每个链表都已经按**升序**排列。请你将所有链表合并到一个升序链表中，返回合并后的链表。

### 示例

```
输入：lists = [[1,4,5],[1,3,4],[2,6]]
输出：[1,1,2,3,4,4,5,6]
解释：三个有序链表合并成一个。

  1 → 4 → 5
  1 → 3 → 4
  2 → 6
  ─────────
  1 → 1 → 2 → 3 → 4 → 4 → 5 → 6

输入：lists = []
输出：[]

输入：lists = [[]]
输出：[]
```

---

## 解法：优先队列（小顶堆）

### 思路

合并 K 个有序链表，最直观的想法是「每次从所有链表的头节点中挑最小的那个」。如果每次遍历所有头节点找最小，复杂度是 O(kn)，当 k 很大时效率低。

用**优先队列（小顶堆）**可以优化取最小值的操作：

> 把每个链表的头节点放入小顶堆，每次弹出堆顶（最小值），然后把该节点的下一个节点入堆。

这样每次获取最小值和插入新值都是 O(log k)，总复杂度 O(n log k)。

### 与合并两个有序链表的联系

| 合并数量 | 方法 | 单次取最小 |
|---------|------|-----------|
| 两个 | 双指针对比 | O(1) |
| K 个 | 优先队列 | O(log k) |

本质思想一样——每次从各链表头部取最小的那个接上去。K 个时「谁最小」的判断用堆来加速。

### 思考方式图解

```
lists = [[1,4,5], [1,3,4], [2,6]]

Step 0: 三个头节点入堆
        堆：1(0), 1(1), 2(2)     ← 括号内为链表索引

Step 1: 弹出 1(0)，将 4(0) 入堆
        堆：1(1), 2(2), 4(0)
        cur: 1

Step 2: 弹出 1(1)，将 3(1) 入堆
        堆：2(2), 3(1), 4(0)
        cur: 1 → 1

Step 3: 弹出 2(2)，将 6(2) 入堆
        堆：3(1), 4(0), 6(2)
        cur: 1 → 1 → 2

... 以此类推，直到堆空

结果：1 → 1 → 2 → 3 → 4 → 4 → 5 → 6 ✅
```

### 代码实现

```java
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        // 小顶堆：按节点值升序排列
        PriorityQueue<ListNode> pq = new PriorityQueue<>((a, b) -> a.val - b.val);

        // 所有非空链表的头节点入堆
        for (ListNode head : lists) {
            if (head != null) {
                pq.offer(head);
            }
        }

        ListNode dummy = new ListNode(0);
        ListNode cur = dummy;

        while (!pq.isEmpty()) {
            ListNode node = pq.poll();   // 取出最小节点
            if (node.next != null) {
                pq.offer(node.next);     // 该链表的下一个节点入堆
            }
            cur.next = node;
            cur = cur.next;
        }

        return dummy.next;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n log k)** — n 为总节点数，k 为链表个数，每次堆操作 O(log k) |
| 🧠 空间复杂度 | **O(k)** — 优先队列中最多同时存在 k 个节点 |

### 💡 分治解法（两两配对归并）

优先队列需要 O(k) 的堆空间。如果不想用堆，还可以用**分治**的思路——像归并排序一样，把 K 个链表两两配对合并，每轮合并后链表数减半，重复直到只剩一个。

```
原始： [list0, list1, list2, list3, list4]
第1轮： [merge(0,1), merge(2,3), list4]    ← k/2 组
第2轮： [merge(merge(0,1), merge(2,3)), list4]
第3轮： 全部合并完成
```

```java
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        int m = lists.length;
        if (m == 0) return null;

        for (int step = 1; step < m; step *= 2) {
            for (int i = 0; i < m - step; i += step * 2) {
                lists[i] = mergeTwoLists(lists[i], lists[i + step]);
            }
        }

        return lists[0];
    }

    // 21. 合并两个有序链表
    private ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        ListNode dummy = new ListNode();
        ListNode cur = dummy;
        while (list1 != null && list2 != null) {
            if (list1.val < list2.val) {
                cur.next = list1;
                list1 = list1.next;
            } else {
                cur.next = list2;
                list2 = list2.next;
            }
            cur = cur.next;
        }
        cur.next = list1 != null ? list1 : list2;
        return dummy.next;
    }
}
```

两种写法对比：

| | 优先队列 | 分治 |
|--|---------|------|
| 时间 | O(n log k) | O(n log k) |
| 空间 | **O(k)** | **O(log k)**（递归实现）或 **O(1)**（迭代实现） |
| 思路 | 堆每次取最小 | 两两合并，规模减半 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 优先队列（小顶堆） |
| 算法类型 | 链表、堆、分治 |
| 核心技巧 | **用堆维护 K 个头节点的最小值，每次弹出后把下一个节点补进去** |
| 适用场景 | 多路有序归并——K 个有序数组合并、K 个有序链表合并 |
| 前置基础 | LeetCode 21 合并两个有序链表 |
| 易错点 | 入堆时要判 `head != null`；弹出后判 `node.next != null` 决定是否再入堆 |

### 一句话记住

> **「小顶堆存 K 个头，弹出最小再补位。」**

<p align="right"><i>练习日期：2026-07-13</i></p>

---

---

## 题目：LRU 缓存（LRU Cache）

**LeetCode 146 | Hot 100 | 难度：🟡 中等**

### 题目描述

请你设计并实现一个满足 **LRU（最近最少使用）缓存** 约束的数据结构。

实现 `LRUCache` 类：

- `LRUCache(int capacity)` — 以正整数作为容量初始化缓存
- `int get(int key)` — 如果关键字存在则返回其值，否则返回 -1
- `void put(int key, int value)` — 如果关键字已存在则更新其值；否则插入。当缓存容量达到上限时，它应该在写入新数据之前**删除最久未使用的数据**。

> 要求 get 和 put 的时间复杂度均为 **O(1)**。

### 示例

```
输入：
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
[[2],       [1,1],  [2,2],  [1],   [3,3],  [2],   [4,4],  [1],   [3],    [4]]
输出：
[null,       null,   null,   1,     null,   -1,    null,   -1,    3,      4]

解释：
LRUCache lRUCache = new LRUCache(2);
lRUCache.put(1, 1);          // 缓存：{1=1}
lRUCache.put(2, 2);          // 缓存：{1=1, 2=2}
lRUCache.get(1);             // 返回 1
lRUCache.put(3, 3);          // 淘汰 key 2，缓存：{1=1, 3=3}
lRUCache.get(2);             // 返回 -1（没找到）
lRUCache.put(4, 4);          // 淘汰 key 1，缓存：{4=4, 3=3}
lRUCache.get(1);             // 返回 -1
lRUCache.get(3);             // 返回 3
lRUCache.get(4);             // 返回 4
```

---

## 解法：哈希表 + 双向链表

### 思路

O(1) 的 get 和 put 需要两个数据结构的配合：

| 需求 | 用啥 | 为什么 |
|------|------|--------|
| O(1) 查找 key → value | **哈希表** `HashMap<key, Node>` | 哈希表查 key 是 O(1) |
| O(1) 增删节点 & 维护顺序 | **双向链表** | 链表头部 = 最近使用，尾部 = 最久未使用 |

> **哈希表负责找，双向链表负责排。**

### 把书架想象成 LRU

灵茶山艾府用一个很妙的类比：

> 把书（Node）一本本叠放在桌上：
> - **get**：从书堆里找到这本书，抽出来放到最上面
> - **put(已有)**：更新内容，抽出来放到最上面
> - **put(新书)**：放到最上面；如果书太多，把最底下的那本扔掉

双向链表让「抽出来」和「放到最上面」这两个操作都是 O(1)。

### 数据结构设计

```java
private static class Node {
    int key, value;
    Node prev, next;
    Node(int k, int v) { key = k; value = v; }
}

private final int capacity;
private final Node dummy = new Node(0, 0);   // 哨兵节点，简化边界
private final Map<Integer, Node> keyToNode;  // key → 节点
```

关键：**链表中不仅存 value，还要存 key**，这样在淘汰尾部节点时才能从哈希表中删除对应的 key。

### 核心操作图解

```
dummy 是哨兵节点，构成环形双向链表：
dummy.next = 最上面的书（最近使用）
dummy.prev = 最底下的书（最久未使用）

初始状态（capacity=2）：
dummy ↔ dummy    (空链表)

put(1,1)：
dummy ↔ node(1,1) ↔ dummy
         ↑最近        ↑最久

put(2,2)：
dummy ↔ node(2,2) ↔ node(1,1) ↔ dummy
         ↑最近                 ↑最久

get(1) → 把 1 移到最上面：
dummy ↔ node(1,1) ↔ node(2,2) ↔ dummy
         ↑最近                 ↑最久

put(3,3) → 容量超了，淘汰最久 (2)：
dummy ↔ node(3,3) ↔ node(1,1) ↔ dummy
         ↑最近                 ↑最久
```

### 代码实现

```java
class LRUCache {
    // 双向链表节点
    private static class Node {
        int key, value;
        Node prev, next;
        Node(int k, int v) { key = k; value = v; }
    }

    private final int capacity;
    private final Node dummy = new Node(0, 0);  // 哨兵节点
    private final Map<Integer, Node> keyToNode = new HashMap<>();

    public LRUCache(int capacity) {
        this.capacity = capacity;
        dummy.prev = dummy;   // 初始时自己指向自己
        dummy.next = dummy;
    }

    public int get(int key) {
        Node node = getNode(key);
        return node != null ? node.value : -1;
    }

    public void put(int key, int value) {
        Node node = getNode(key);
        if (node != null) {
            node.value = value;  // 已有，直接更新
            return;
        }
        node = new Node(key, value);
        keyToNode.put(key, node);
        pushFront(node);         // 新书放最上面
        if (keyToNode.size() > capacity) {
            Node backNode = dummy.prev;   // 最久未使用
            keyToNode.remove(backNode.key);
            remove(backNode);
        }
    }

    // getNode: 找到节点并移到最上面
    private Node getNode(int key) {
        if (!keyToNode.containsKey(key)) return null;
        Node node = keyToNode.get(key);
        remove(node);      // 从当前位置抽出来
        pushFront(node);   // 放到最上面
        return node;
    }

    // 删除节点
    private void remove(Node x) {
        x.prev.next = x.next;
        x.next.prev = x.prev;
    }

    // 头插（放在 dummy 后面）
    private void pushFront(Node x) {
        x.prev = dummy;
        x.next = dummy.next;
        x.prev.next = x;
        x.next.prev = x;
    }
}
```

### 复杂度分析

| 操作 | 时间复杂度 | 说明 |
|------|-----------|------|
| `get` | **O(1)** | 哈希表查 + 链表删除/头插 |
| `put` | **O(1)** | 同上，更新或淘汰尾部 |
| 🧠 空间 | **O(capacity)** | 哈希表 + 链表节点 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 哈希表 + 双向链表 |
| 算法类型 | 数据结构设计、链表应用 |
| 核心技巧 | **哈希表 O(1) 定位，双向链表 O(1) 移动；dummy 节点省去大量判空** |
| 适用场景 | 缓存淘汰策略、需要同时满足 O(1) 查找和 O(1) 增删的场景 |
| 易错点 | 链表节点必须存 key（淘汰时要从哈希表删除）；dummy 环形链表初始化时自己指向自己；双向链表删除要改两个方向的指针 |

### 一句话记住

> **「哈希表定位，双向链表排序——LRU 的黄金搭档。」**

<p align="right"><i>练习日期：2026-07-14</i></p>
