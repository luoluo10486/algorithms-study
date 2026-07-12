# Day 4 — 链表 Hot 100 练习

## 题目：相交链表（Intersection of Two Linked Lists）

**LeetCode 160 | Hot 100 | 难度：🟢 简单**

### 题目描述

给你两个单链表的头节点 `headA` 和 `headB`，请你找出并返回两个单链表**相交的起始节点**。如果两个链表不存在相交节点，返回 `null`。

> 相交的定义：两个链表在某个节点之后完全重合（引用相同，不是值相同）。

### 示例

```
输入：intersectVal = 8, listA = [4,1,8,4,5], listB = [5,6,1,8,4,5]
输出：Intersected at '8'
解释：相交节点的值为 8。

        4 → 1 ↘
               8 → 4 → 5
    5 → 6 → 1 ↗

输入：intersectVal = 0, listA = [2,6,4], listB = [1,5]
输出：null
解释：两个链表不相交。
```

---

## 解法：双指针（浪漫相遇法 ❤️）

### 思路

核心问题：两个链表的长度不同，如何让两个指针同时到达相交点？

**突破口**：让两个指针走「相同总路程」。

> `a` 从 headA 出发，走到末尾后切换到 headB 继续走；
> `b` 从 headB 出发，走到末尾后切换到 headA 继续走。
>
> 如果相交，`a` 和 `b` 会在相交点相遇；如果不相交，它们会同时走到 null。

为什么这样可行？
- 假设 `headA` 到相交点的距离为 `lenA`，`headB` 到相交点的距离为 `lenB`，相交点后的公共部分长度为 `lenC`
- `a` 走的总路程 = `lenA + lenC + lenB`
- `b` 走的总路程 = `lenB + lenC + lenA`
- 两者相等 → 同时到达相交点

### 思考方式图解

```
headA:     4 → 1 ↘
                    8 → 4 → 5
headB: 5 → 6 → 1 ↗

a 的行走路线：4 → 1 → 8 → 4 → 5 → null → 5 → 6 → 1 → 8  ← 相遇 ❤️
b 的行走路线：5 → 6 → 1 → 8 → 4 → 5 → null → 4 → 1 → 8  ← 相遇 ❤️

            └───────── 前半段 ─────────┘ └── 后半段 ──┘
            (各自独有的部分 + 公共部分)     (对方的独有部分)
```

### 代码实现

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        ListNode a = headA;
        ListNode b = headB;

        while (a != b) {
            // a 走完 A 后切换到 B
            if (a == null) {
                a = headB;
            } else {
                a = a.next;
            }
            // b 走完 B 后切换到 A
            if (b == null) {
                b = headA;
            } else {
                b = b.next;
            }
        }

        return a;  // 相遇点 or null
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(m + n)** — m、n 分别为两链表长度，最多遍历一次 |
| 🧠 空间复杂度 | **O(1)** — 只用了两个指针 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 双指针追逐（浪漫相遇法） |
| 算法类型 | 链表、双指针 |
| 核心技巧 | **两个指针分别走完自己的链表后切换到对方链表，消除长度差** |
| 适用场景 | 链表相交判断、链表环入口查找等需要消除长度差的问题 |
| 易错点 | 切换时机是当前指针走到 `null` 时，不是当前节点的 `next` 为 null 时；不交时会同时走到 null，while 循环结束 |

### 一句话记住

> **「你走下我的路，我走下你的路，我们就会在交点相遇。」**

---

*练习日期：2026-07-12*

---

---

## 题目：反转链表（Reverse Linked List）

**LeetCode 206 | Hot 100 | 难度：🟢 简单**

### 题目描述

给你单链表的头节点 `head`，请你反转链表，并返回反转后的链表。

### 示例

```
输入：head = [1,2,3,4,5]
输出：[5,4,3,2,1]

输入：head = [1,2]
输出：[2,1]

输入：head = []
输出：[]
```

---

## 解法：迭代（原地反转）

### 思路

反转链表其实就是**把每个节点的 next 指针指向前一个节点**，但直接改 `cur.next = pre` 会丢掉下一个节点的访问入口，所以需要一个 `temp` 暂存。

核心维护三个指针：

| 指针 | 作用 |
|------|------|
| `pre` | 已经反转好的部分的头节点（初始为 null） |
| `cur` | 当前正在处理的节点 |
| `temp` | 暂存 `cur.next`，防止断链 |

每一步：`cur.next → pre`，然后三个指针整体右移一位。

### 思考方式图解

```
null ← 1 → 2 → 3 → 4 → 5
  ↑    ↑
 pre   cur

Step 1: temp = cur.next (2)
        cur.next = pre (null)        null ← 1    2 → 3 → 4 → 5
        pre = cur (1)
        cur = temp (2)                     ↑     ↑
                                          pre   cur

Step 2: temp = cur.next (3)
        cur.next = pre (1)               null ← 1 ← 2    3 → 4 → 5
        pre = cur (2)
        cur = temp (3)                              ↑     ↑
                                                   pre   cur

Step 3: ... 以此类推 ...

最终：null ← 1 ← 2 ← 3 ← 4 ← 5
                                    ↑
                                   pre
                                   (cur = null, 循环结束)
```

### 代码实现

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode pre = null;   // 已经反转好的链表头
        ListNode cur = head;   // 当前待反转的节点

        while (cur != null) {
            ListNode temp = cur.next;  // 先保存下一个节点，防止断链
            cur.next = pre;            // 当前节点指向前一个节点（反转）
            pre = cur;                 // pre 右移
            cur = temp;                // cur 右移
        }

        return pre;  // pre 就是反转后的新头
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 遍历一次链表 |
| 🧠 空间复杂度 | **O(1)** — 只用了三个指针 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 迭代反转（三指针） |
| 算法类型 | 链表、指针操作 |
| 核心技巧 | **用 temp 暂存下一个节点，再逐个反转 next 指针** |
| 适用场景 | 链表反转、回文链表判断（配合快慢指针）、K 个一组反转的基础操作 |
| 其他解法 | 递归解法（O(n) 空间，利用递归栈） |

### 一句话记住

> **「先存后路再回头，三个指针走天下。」**

<p align="right"><i>练习日期：2026-07-12</i></p>

---

---

## 题目：回文链表（Palindrome Linked List）

**LeetCode 234 | Hot 100 | 难度：🟢 简单**

### 题目描述

给你一个单链表的头节点 `head`，请你判断该链表是否为**回文链表**。

> 回文：正着读和反着读一样，如 `1 → 2 → 2 → 1`。

### 示例

```
输入：head = [1,2,2,1]
输出：true

输入：head = [1,2]
输出：false
```

### 进阶要求

你能否用 **O(n) 时间**和 **O(1) 空间**解决？

---

## 解法：快慢指针找中点 + 反转后半段

### 思路

判断回文最直接的方式是「从两端往中间比较」。但链表不像数组那样可以随机访问末尾元素，所以需要把后半段链表**反转**后再比较。

这题是**两道基础题的组合应用**：

| 步 | 操作 | 对应题目 |
|----|------|----------|
| 1 | 快慢指针找到链表中点 | LeetCode 876 |
| 2 | 反转后半段链表 | LeetCode 206 |
| 3 | 同时遍历前半段和反转后的后半段，逐个比较值 | — |

> 注意：找中点时，奇数长度链表中点偏右（如 `[1,2,3,2,1]` 的中点是第 3 个节点 `3`），反转后 `3 → 2 → 1`，与前半段 `1 → 2 → 3` 比较，中间值 3 和自己比较不影响结果。

### 思考方式图解

```
链表：1 → 2 → 2 → 1

Step 1: 快慢指针找中点
        slow ← slow.next
        fast ← fast.next.next

        1 → 2 → 2 → 1
        ↑
      slow
        fast            slow 走到 2（中点）
                        fast 走到 null

Step 2: 反转后半段
        1 → 2    2 → 1   →   1 → 2    1 → 2
              ↑                         ↑
             mid                      head2

Step 3: 同时遍历比较
        head: 1 → 2
        head2: 1 → 2     ← 逐个比较，值相同 → true
```

### 代码实现

```java
class Solution {
    public boolean isPalindrome(ListNode head) {
        // Step 1: 找中点
        ListNode mid = middleNode(head);
        // Step 2: 反转后半段
        ListNode head2 = reverseList(mid);

        // Step 3: 比较前半段和反转后的后半段
        while (head2 != null) {
            if (head.val != head2.val) {
                return false;
            }
            head = head.next;
            head2 = head2.next;
        }
        return true;
    }

    // 876. 链表的中间结点 — 快慢指针
    private ListNode middleNode(ListNode head) {
        ListNode slow = head;
        ListNode fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }
        return slow;
    }

    // 206. 反转链表 — 迭代三指针
    private ListNode reverseList(ListNode head) {
        ListNode pre = null;
        ListNode cur = head;
        while (cur != null) {
            ListNode nxt = cur.next;
            cur.next = pre;
            pre = cur;
            cur = nxt;
        }
        return pre;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 找中点 O(n/2) + 反转 O(n/2) + 比较 O(n/2) |
| 🧠 空间复杂度 | **O(1)** — 只用了几个指针变量 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 快慢指针 + 反转后半段 |
| 算法类型 | 链表、双指针、组合题 |
| 核心技巧 | **快慢指针找中点，反转后半段，两段同步比较——用空间换 O(1)** |
| 适用场景 | 链表回文判断、需要同时从两端遍历链表的场景 |
| 前置基础 | LeetCode 876（找中点）+ LeetCode 206（反转链表） |

### 一句话记住

> **「中点切开后半翻，两头同步比个遍。」**

<p align="right"><i>练习日期：2026-07-12</i></p>

---

---

## 题目：环形链表（Linked List Cycle）

**LeetCode 141 | Hot 100 | 难度：🟢 简单**

### 题目描述

给你一个链表的头节点 `head`，判断链表中是否有环。

> 环的定义：链表中某个节点的 next 指针指向了之前某节点，导致链表尾部形成了闭环。

### 示例

```
输入：head = [3,2,0,-4], 环的索引 = 1
输出：true
解释：链表尾节点 -4 的 next 指向索引 1 的节点 2，形成环。

    3 → 2 → 0 → -4
        ↑         │
        └─────────┘

输入：head = [1,2], 环的索引 = 0
输出：true
解释：尾节点 2 的 next 指向头节点 1，形成环。

    1 → 2
    ↑    │
    └────┘

输入：head = [1], 环的索引 = -1
输出：false
解释：无环。
```

---

## 解法：快慢指针（Floyd 判圈算法）

### 思路

想象两个运动员在环形跑道上跑步——快跑者最终会追上慢跑者，这就是 Floyd 判圈算法的核心。

> 用两个指针：`slow` 每次走 1 步，`fast` 每次走 2 步。
> 如果有环，`fast` 一定会「套圈」追上 `slow`；如果无环，`fast` 会先走到 null。

### 为什么 fast 每次移动 2 步，而不是 3 步或更多？

- 如果 `fast` 每次比 `slow` 多走 1 步（fast=2, slow=1），**每轮距离缩短 1 步**，`fast` 必然能追上 `slow`
- 如果 `fast` 每次走 3 步，在环较小的情况下可能直接跳过了 `slow`，需要额外处理
- 2 步是最简单可靠的方案

### 思考方式图解

```
有环链表：3 → 2 → 0 → -4
            ↑           │
            └───────────┘

快慢指针初始都在 head (3)：

Round 1: slow=2  fast=0
Round 2: slow=0  fast=2
Round 3: slow=-4 fast=0
Round 4: slow=2  fast=-4   ← 绕圈 but 还没相遇
Round 5: slow=0  fast=0    ← slow == fast ✅ 有环！
```

### 代码实现

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        ListNode fast = head;
        ListNode slow = head;

        while (fast != null && fast.next != null) {
            slow = slow.next;          // 慢指针走 1 步
            fast = fast.next.next;     // 快指针走 2 步
            if (slow == fast) {        // 相遇 = 有环
                return true;
            }
        }

        return false;  // fast 走到尽头 = 无环
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 无环时 fast 遍历一次到底；有环时 slow 进环后最多走一圈内被追上 |
| 🧠 空间复杂度 | **O(1)** — 只用了两个指针 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | Floyd 判圈算法（快慢指针） |
| 算法类型 | 链表、双指针 |
| 核心技巧 | **快指针每次 2 步、慢指针每次 1 步，有环必相遇** |
| 适用场景 | 判断链表是否有环、寻找环入口（环形链表 II） |
| 易错点 | `while` 条件必须判 `fast != null && fast.next != null`，漏了 `fast.next` 会 NPE |

### 一句话记住

> **「一快一慢，有环必追。」**

<p align="right"><i>练习日期：2026-07-12</i></p>

---

---

## 题目：环形链表 II（Linked List Cycle II）

**LeetCode 142 | Hot 100 | 难度：🟡 中等**

### 题目描述

给定一个链表的头节点 `head`，返回链表开始入环的第一个节点。如果链表无环，则返回 `null`。

> 这是「环形链表」的进阶版——不仅要判环，还要找到环的入口。

### 示例

```
输入：head = [3,2,0,-4], 环的索引 = 1
输出：返回索引 1 的节点
解释：环的入口是节点 2。

    3 → 2 → 0 → -4
        ↑         │
        └─────────┘

输入：head = [1], 环的索引 = -1
输出：null
```

---

## 解法：快慢指针 + 数学推导

### 思路

在「环形链表」判断有环的基础上增加一步：

> **相遇后，让一个指针从头出发，另一个指针从相遇点出发，每次都走 1 步——它们再次相遇的位置就是环的入口。**

这背后有一个严谨的数学推导：

```
设：
  a = 头节点 → 环入口的距离
  b = 环入口 → 快慢指针相遇点的距离
  c = 相遇点 → 环入口的距离（环剩余部分）

环周长 = b + c

slow 走的距离 = a + b
fast 走的距离 = a + b + k(b + c)  (k 为 fast 在环内绕的圈数)

fast = 2 × slow
→ a + b + k(b + c) = 2(a + b)
→ k(b + c) = a + b
→ a = k(b + c) - b
→ a = (k-1)(b + c) + c

当 k = 1 时：a = c   ← 最简情况
```

结论：**从头节点到环入口的距离 = 相遇点到环入口的距离**（当 fast 在环内走了 1 圈多的时候）。所以两个指针分别从头和相遇点同速前进，必然在环入口相遇。

### 思考方式图解

```
有环链表：3 → 2 → 0 → -4
            ↑           │
            └───────────┘

Step 1: 快慢指针找到相遇点

   3 → 2 → 0 → -4
       ↑         │
       └─────────┘
       slow=2  fast=2  ← 相遇点 (节点 0)

Step 2: index1 从相遇点出发，index2 从头出发

   3 → 2 → 0 → -4
   ↑       ↑
 index2  index1

   index2 走一步到 2
   index1 走一步到 -4

   index2 走一步到 2 (环入口)
   index1 走一步到 2 (环入口)  ← 相遇！✅
```

### 代码实现

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode fast = head;
        ListNode slow = head;

        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;

            if (slow == fast) {  // 有环
                ListNode index1 = fast;  // 相遇点
                ListNode index2 = head;  // 头节点
                // 同步前进，相遇处就是环入口
                while (index1 != index2) {
                    index1 = index1.next;
                    index2 = index2.next;
                }
                return index1;
            }
        }

        return null;  // 无环
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 判环 O(n) + 找入口 O(n) |
| 🧠 空间复杂度 | **O(1)** — 只用了几个指针变量 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | Floyd 判圈 + 数学推导找入口 |
| 算法类型 | 链表、双指针、数学 |
| 核心技巧 | **相遇后，一个从头、一个从相遇点同速走，再次相遇就是环入口** |
| 适用场景 | 链表环入口查找 |
| 前置基础 | LeetCode 141 环形链表——判环是找入口的前提 |
| 数学本质 | `a = c`（头到入口 = 相遇到入口），前提是 fast 比 slow 多走一圈 |

### 一句话记住

> **「相遇之后，从头再来，入口自现。」**

<p align="right"><i>练习日期：2026-07-12</i></p>

---

---

## 题目：合并两个有序链表（Merge Two Sorted Lists）

**LeetCode 21 | Hot 100 | 难度：🟢 简单**

### 题目描述

将两个**升序**链表合并为一个新的**升序**链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。

### 示例

```
输入：list1 = [1,2,4], list2 = [1,3,4]
输出：[1,1,2,3,4,4]

输入：list1 = [], list2 = [0]
输出：[0]

输入：list1 = [], list2 = []
输出：[]
```

---

## 解法：双指针 + 虚拟头节点

### 思路

合并两个有序链表的过程很像**拉链**——两个链表像两条拉链齿，`cur` 指针像拉链头，每次从两个链表的当前节点中挑一个较小的接上去，然后对应指针后移。

引入一个**虚拟头节点** `dummy`，让 `cur` 从 dummy 开始串联节点。这样做的好处是：不需要单独处理头节点为空的边界情况，最后直接返回 `dummy.next` 即可。

### 思考方式图解

```
list1: 1 → 2 → 4
list2: 1 → 3 → 4

dummy → ?

Step 1: 1 vs 1  (相等取 list2)
        dummy → 1 (list2)
        list2 → 3

Step 2: 1 vs 3
        dummy → 1 → 1 (list1)
        list1 → 2

Step 3: 2 vs 3
        dummy → 1 → 1 → 2 (list1)
        list1 → 4

Step 4: 4 vs 3
        dummy → 1 → 1 → 2 → 3 (list2)
        list2 → 4

Step 5: 4 vs 4  (相等取 list2)
        dummy → 1 → 1 → 2 → 3 → 4 (list2)
        list2 → null

list1 还剩 4 → 直接拼接：

        dummy → 1 → 1 → 2 → 3 → 4 → 4

返回 dummy.next → [1,1,2,3,4,4] ✅
```

### 代码实现

```java
class Solution {
    public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        ListNode dummy = new ListNode();   // 虚拟头节点
        ListNode cur = dummy;              // 当前指针

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

        // 拼接剩余部分（最多有一个链表非空）
        cur.next = list1 != null ? list1 : list2;

        return dummy.next;  // 跳过虚拟头节点
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(m + n)** — m、n 分别为两链表长度 |
| 🧠 空间复杂度 | **O(1)** — 只用了几个指针 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 双指针 + 虚拟头节点 |
| 算法类型 | 链表、双指针 |
| 核心技巧 | **dummy 节点统一处理，cur 每次接较小值** |
| 其他解法 | 递归解法（代码更短，但空间 O(m+n) 来自递归栈） |
| 前置基础 | 本题是「合并 K 个升序链表」等难题的基础操作 |

### 一句话记住

> **「拉链合并，虚拟头省心。」**

<p align="right"><i>练习日期：2026-07-12</i></p>

---

---

## 题目：两数相加（Add Two Numbers）

**LeetCode 2 | Hot 100 | 难度：🟡 中等**

### 题目描述

给你两个**非空**的链表，表示两个**非负的整数**。它们每位数字都是按照**逆序**的方式存储的，并且每个节点只能存储**一位**数字。

请你将两个数相加，并以相同形式返回一个表示和的链表。

> 逆序存储的好处：表头就是个位，做加法时可以从头开始逐位相加，刚好符合竖式计算的顺序。

### 示例

```
输入：l1 = [2,4,3], l2 = [5,6,4]
输出：[7,0,8]
解释：342 + 465 = 807

    2 → 4 → 3
  + 5 → 6 → 4
  ───────────
    7 → 0 → 8

输入：l1 = [0], l2 = [0]
输出：[0]

输入：l1 = [9,9,9,9,9,9,9], l2 = [9,9,9,9]
输出：[8,9,9,9,0,0,0,1]
解释：9999999 + 9999 = 10009998（逆序：8→9→9→9→0→0→0→1）
```

---

## 解法：递归逐位相加

### 思路

加法竖式是从最低位（个位）开始逐位相加，满十进一。链表恰好是逆序存储的，所以表头就是最低位——直接从头到尾遍历就是在做竖式加法。

递归的视角：定义 `addTwo(l1, l2, carry)` 表示「以 l1 和 l2 为当前位、carry 为进位，计算当前位的和并返回后续结果链表」：

1. 如果 `l1`、`l2` 都 null 且 `carry = 0` → 递归终止，返回 null
2. 当前位的和 = `l1.val + l2.val + carry`（null 的节点视为 0）
3. 创建当前节点：`val = sum % 10`，`next = addTwo(l1.next, l2.next, sum / 10)`

### 思考方式图解

```
l1: 2 → 4 → 3    (342)
l2: 5 → 6 → 4    (465)

逐位递归：

addTwo(l1=2, l2=5, carry=0)
  sum=2+5+0=7
  node(7, addTwo(l1=4, l2=6, carry=0))

    addTwo(l1=4, l2=6, carry=0)
      sum=4+6+0=10
      node(0, addTwo(l1=3, l2=4, carry=1))

        addTwo(l1=3, l2=4, carry=1)
          sum=3+4+1=8
          node(8, addTwo(l1=null, l2=null, carry=0))

            addTwo(null, null, 0)
              return null

结果：7 → 0 → 8 ✅
```

```
长度不等的情况：
l1: 9 → 9 → 9 → 9 → 9 → 9 → 9    (9999999)
l2: 9 → 9 → 9 → 9                  (9999)

倒数第二步：
  addTwo(l1=9(l6), l2=null, carry=1)
    sum=9+0+1=10
    node(0, addTwo(l1=null, l2=null, carry=1))

      addTwo(null, null, 1)
        sum=0+0+1=1
        node(1, null)

结果：8 → 9 → 9 → 9 → 0 → 0 → 0 → 1 ✅
```

### 代码实现

```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        return addTwo(l1, l2, 0);
    }

    private ListNode addTwo(ListNode l1, ListNode l2, int carry) {
        // 递归终止：没有节点要处理，也没有进位
        if (l1 == null && l2 == null && carry == 0) {
            return null;
        }

        int sum = carry;
        if (l1 != null) {
            sum += l1.val;
            l1 = l1.next;
        }
        if (l2 != null) {
            sum += l2.val;
            l2 = l2.next;
        }

        // 当前位 = sum % 10，进位 = sum / 10
        return new ListNode(sum % 10, addTwo(l1, l2, sum / 10));
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(max(m, n))** — 递归深度等于较长链表的长度 |
| 🧠 空间复杂度 | **O(max(m, n))** — 递归调用栈深度 |

### 💡 迭代写法（省递归栈）

递归写起来简洁，但空间 O(n) 来自调用栈。迭代版用 `dummy` 节点 + `carry` 变量沿着链表走，空间降到 **O(1)**，逻辑和递归完全对称：

```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(); // 哨兵节点
        ListNode cur = dummy;
        int carry = 0;
        while (l1 != null || l2 != null || carry != 0) {
            int sum = carry;
            if (l1 != null) {
                sum += l1.val;
                l1 = l1.next;
            }
            if (l2 != null) {
                sum += l2.val;
                l2 = l2.next;
            }
            cur = cur.next = new ListNode(sum % 10);
            carry = sum / 10;
        }
        return dummy.next;
    }
}
```

> 核心变化：`while` 条件把 `carry != 0` 也纳入判断，**自动处理最后多出的进位**——多创建一位节点。

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 递归逐位相加 |
| 算法类型 | 链表、递归、数学 |
| 核心技巧 | **利用逆序存储的特性，递归天然契合竖式加法——每层处理一位，carry 传下去** |
| 适用场景 | 链表表示的大数加法、链表表示的大数运算 |
| 迭代写法 | 已附在上方，空间 **O(1)** |
| 易错点 | 两个链表长度不同时，短链表的节点视为 null（值为 0）；最后一位相加可能产生额外进位（如 999+1） |

### 一句话记住

> **「逆序链表直接加，递归竖式满十进一。」**

<p align="right"><i>练习日期：2026-07-12</i></p>

---

---

## 题目：删除链表的倒数第 N 个结点（Remove Nth Node From End of List）

**LeetCode 19 | Hot 100 | 难度：🟡 中等**

### 题目描述

给你一个链表，删除链表的**倒数第 `n` 个**结点，并且返回链表的头结点。

> 进阶：尝试使用一趟扫描实现。

### 示例

```
输入：head = [1,2,3,4,5], n = 2
输出：[1,2,3,5]
解释：倒数第 2 个节点是 4，删除后链表为 [1,2,3,5]。

    1 → 2 → 3 → 4 → 5
                ↑
               删除这个

输入：head = [1], n = 1
输出：[]
解释：只有一个节点，删除后为空。

输入：head = [1,2], n = 1
输出：[1]
```

---

## 解法：快慢指针 + 虚拟头节点

### 思路

倒数第 n 个节点，正着看就是「距离末尾 n 步」的那个节点。

核心技巧：让 `fast` 先走 **n+1** 步，然后 `slow` 和 `fast` 同步移动。当 `fast` 走到 null 时，`slow` 刚好停在待删节点的**前一个节点**——直接跳过它即可。

> 为什么要走 n+1 步而不是 n 步？
> 因为我们要删除节点，需要先找到它的**前驱节点**（prev），而不是它本身。走 n+1 步 → fast 比 slow 多 n+1 → fast 到 null 时 slow 刚好在待删节点的前面一位。

### 为什么需要虚拟头节点？

如果删除的是头节点（head），需要修改 head。用 `dummy` 可以统一处理所有情况，最后直接返回 `dummy.next` 即可。

### 思考方式图解

```
链表：1 → 2 → 3 → 4 → 5,  n = 2

Step 0: 设置 dummy
        dummy → 1 → 2 → 3 → 4 → 5
        ↑
      fast/slow

Step 1: fast 先走 n+1 = 3 步
        dummy → 1 → 2 → 3 → 4 → 5
         ↑               ↑
        slow            fast

Step 2: fast/slow 同步移动，直到 fast == null
        dummy → 1 → 2 → 3 → 4 → 5
                        ↑               ↑
                       slow            fast=null

        slow 指向 3（待删节点 4 的前驱）

Step 3: 删除 slow.next（节点 4）
        slow.next = slow.next.next

        1 → 2 → 3 → 5
```

### 代码实现

```java
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode dummy = new ListNode(-1);
        dummy.next = head;

        ListNode fast = dummy;
        ListNode slow = dummy;

        // fast 先走 n+1 步
        for (int i = 0; i <= n; i++) {
            fast = fast.next;
        }

        // 同步移动，直到 fast 到 null
        while (fast != null) {
            fast = fast.next;
            slow = slow.next;
        }

        // 删除 slow 后面的节点
        slow.next = slow.next.next;

        return dummy.next;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 一趟扫描 |
| 🧠 空间复杂度 | **O(1)** — 只用了几个指针 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 快慢指针 + 虚拟头节点 |
| 算法类型 | 链表、双指针 |
| 核心技巧 | **fast 先走 n+1 步，保证 slow 落在待删节点的前驱位置** |
| 适用场景 | 链表倒数第 k 个节点的查找与删除 |
| 易错点 | 步数是 n+1 不是 n（要找前驱）；必须用 dummy 处理删除头节点的边界 |

### 一句话记住

> **「快针先走 n 加一，慢针停在删除前。」**

<p align="right"><i>练习日期：2026-07-12</i></p>

---

---

## 题目：两两交换链表中的节点（Swap Nodes in Pairs）

**LeetCode 24 | Hot 100 | 难度：🟡 中等**

### 题目描述

给你一个链表，两两交换其中相邻的节点，并返回交换后链表的头节点。你不能只是单纯的改变节点内部的值，而是需要**实际的节点交换**。

### 示例

```
输入：head = [1,2,3,4]
输出：[2,1,4,3]

    1 → 2 → 3 → 4
    ↓   ↓
    2 → 1 → 4 → 3

输入：head = [1]
输出：[1]
解释：只有一个节点，无需交换。

输入：head = [1,2,3]
输出：[2,1,3]
解释：奇数个节点，最后一组没得换。
```

---

## 解法：迭代 + 哑节点（四指针穿针引线）

### 思路

两两交换的核心是**每次处理一对相邻节点**，把 `node1` 和 `node2` 的位置互换。

关键：交换一组节点会影响四个节点的指针指向，因此需要四个指针来维护：

```
交换前：... → node0 → node1 → node2 → node3 → ...
                           ╲ ╱
                         交换这一对

交换后：... → node0 → node2 → node1 → node3 → ...
```

| 指针 | 角色 |
|------|------|
| `node0` | 待交换对的**前驱节点** |
| `node1` | 待交换对的**第一个节点** |
| `node2` | 待交换对的**第二个节点** |
| `node3` | 待交换对的**后继节点**（下一对的开始） |

### 思考方式图解

```
dummy → 1 → 2 → 3 → 4

node0    node1  node2  node3
 ↓       ↓      ↓      ↓
dummy →  1   →  2   →  3 → 4

三步交换：
① node0.next = node2      dummy → 2
                        1 → 2 → 3    (1 仍然指向 2)
② node2.next = node1      2 → 1
③ node1.next = node3      1 → 3

结果：dummy → 2 → 1 → 3 → 4

然后 node0 移到 node1，准备下一对：
                      node0=1  node1=3  node2=4  node3=null
                       ↓        ↓        ↓        ↓
dummy → 2 → 1 → 3 → 4

同理交换 3→4 后：
dummy → 2 → 1 → 4 → 3 ✅
```

### 代码实现

```java
class Solution {
    public ListNode swapPairs(ListNode head) {
        ListNode dummy = new ListNode(-1);
        dummy.next = head;

        ListNode node0 = dummy;   // 前驱
        ListNode node1 = head;    // 当前对第一个节点

        while (node1 != null && node1.next != null) {
            ListNode node2 = node1.next;   // 当前对第二个节点
            ListNode node3 = node2.next;   // 下一对的开始

            // 交换 node1 和 node2
            node0.next = node2;   // 前驱指向 node2
            node2.next = node1;   // node2 指向 node1
            node1.next = node3;   // node1 指向后继

            // 移动到下一组
            node0 = node1;
            node1 = node3;
        }

        return dummy.next;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 遍历一次链表 |
| 🧠 空间复杂度 | **O(1)** — 只用了几个指针 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 四指针迭代交换 |
| 算法类型 | 链表、指针操作 |
| 核心技巧 | **用 node0→node1→node2→node3 四个指针维护一组交换，关键三步重连** |
| 适用场景 | 链表节点成对交换、K 个一组反转链表的基础版本 |
| 易错点 | while 条件要判 `node1 != null && node1.next != null`（保证有完整一对）；交换后前驱要更新为 `node1`（不是 node2） |

### 一句话记住

> **「四针定乾坤，三步换一对。」**

<p align="right"><i>练习日期：2026-07-12</i></p>

---

---

## 题目：K 个一组翻转链表（Reverse Nodes in k-Group）

**LeetCode 25 | Hot 100 | 难度：🔴 困难**

### 题目描述

给你链表的头节点 `head`，每 `k` 个节点一组进行翻转，返回修改后的链表。

`k` 是一个正整数，它的值小于或等于链表的长度。如果节点总数不是 `k` 的整数倍，那么请将最后剩余的节点**保持原有顺序**。

> 进阶：设计一个只用 O(1) 额外空间的算法。

### 示例

```
输入：head = [1,2,3,4,5], k = 2
输出：[2,1,4,3,5]

输入：head = [1,2,3,4,5], k = 3
输出：[3,2,1,4,5]
解释：前 3 个一组翻转，剩余 2 个保持原样。

输入：head = [1,2,3,4,5], k = 1
输出：[1,2,3,4,5]
解释：k=1 相当于不动。
```

---

## 解法：统计长度 + 分段反转

### 思路

这道题是「反转链表」和「两两交换」的**泛化版本**。

核心思路分三步：

1. **先遍历统计链表长度** `n`，算出需要翻转多少组（`n / k`）
2. **外层循环**：每轮处理一组 k 个节点，使用标准的迭代反转（三指针）
3. **组间拼接**：反转完一组后，把上一组的尾部接到这一组的头部

### 关键：两组之间的拼接

每组反转完成后需要处理四次指针：

```
反转前：p0 → (node1 → node2 → ... → nodek) → nextGroup
反转后：p0 → (nodek → ... → node2 → node1) → nextGroup
                    ↑                       ↑
                   pre(cur=nextGroup)      cur
```

| 操作 | 说明 |
|------|------|
| `next = p0.next` | 记下当前组的第一个节点（反转后会变成最后一个） |
| `p0.next.next = cur` | 当前组的最后一个节点（node1）指向下一组的头部 |
| `p0.next = pre` | 前驱直接指向反转后的新头（nodek） |
| `p0 = next` | 前驱移到当前组的最后一个节点，准备下一组 |

### 思考方式图解

```
head = [1,2,3,4,5], k = 3

n = 5, 需要翻转 1 组（5/3=1），剩余 2 个不动。

初始：dummy → 1 → 2 → 3 → 4 → 5
       ↑
       p0

第 1 组反转（从节点 1 开始反转 3 个）：
标准反转过程：
  ① 1.next=null, pre=1, cur=2
  ② 2.next=1, pre=2, cur=3
  ③ 3.next=2, pre=3, cur=4

  反转后：1 → null   2 → 1   3 → 2 → 1
                                    ↑
                                   pre
                                    cur=4

  拼接：
  next = p0.next → 1
  p0.next.next = cur   → 1.next = 4
  p0.next = pre        → dummy.next = 3
  p0 = next            → p0 = 1

结果：dummy → 3 → 2 → 1 → 4 → 5 ✅
```

### 代码实现

```java
class Solution {
    public ListNode reverseKGroup(ListNode head, int k) {
        // 统计链表长度
        int n = 0;
        for (ListNode cur = head; cur != null; cur = cur.next) {
            n++;
        }

        ListNode dummy = new ListNode(0, head);
        ListNode p0 = dummy;   // 当前组的前驱
        ListNode pre = null;
        ListNode cur = head;

        // 每轮翻转一组，共 n/k 组
        for (; n >= k; n -= k) {
            // 标准反转 k 个节点
            for (int i = 0; i < k; i++) {
                ListNode next = cur.next;
                cur.next = pre;
                pre = cur;
                cur = next;
            }

            // 组间拼接
            ListNode next = p0.next;    // 当前组反转后的尾部
            p0.next.next = cur;         // 尾部指向下一组的头部
            p0.next = pre;              // 前驱指向当前组的新头
            p0 = next;                  // 前驱移动到下一组的前一个节点
        }

        return dummy.next;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 统计长度 O(n) + 每组反转 O(k) × (n/k) 组 = O(n) |
| 🧠 空间复杂度 | **O(1)** — 只用了几个指针变量 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 分段反转 + 组间拼接 |
| 算法类型 | 链表、指针操作 |
| 核心技巧 | **先统计长度确定组数，每组反转后通过 p0 做好前后衔接** |
| 适用场景 | 链表的成组反转操作（k = 2 时就是「两两交换」） |
| 前置基础 | LeetCode 206 反转链表 + LeetCode 24 两两交换 |
| 易错点 | 组间拼接的 `p0.next.next = cur` 这一行不能丢，否则链表会断开；剩余不足 k 个的节点保持原序 |

### 一句话记住

> **「先算几组再翻转，组间拼接靠 p0。」**

<p align="right"><i>练习日期：2026-07-12</i></p>

---

---

## 题目：随机链表的复制（Copy List with Random Pointer）

**LeetCode 138 | Hot 100 | 难度：🟡 中等**

### 题目描述

给你一个长度为 `n` 的链表，每个节点除了有一个 `next` 指针指向下一个节点，还有一个 `random` 指针指向链表中的任意节点或 `null`。

请你**深拷贝**这个链表，构造一个由全新节点组成的链表，新节点的 `next` 和 `random` 都应指向新链表中的对应节点。

### 示例

```
输入：head = [[7,null],[13,0],[11,4],[10,2],[1,0]]
输出：[[7,null],[13,0],[11,4],[10,2],[1,0]]
解释：每个节点的 random 指向的节点索引如下：
      节点0(7)  → random=null
      节点1(13) → random=节点0(7)
      节点2(11) → random=节点4(1)
      节点3(10) → random=节点2(11)
      节点4(1)  → random=节点0(7)
```

### 进阶要求

你能否用 **O(1) 额外空间**（不计输出链表）实现？

---

## 解法：在原链表中穿插新节点（O(1) 空间）

### 思路

常规做法是用哈希表映射 `原节点 → 新节点`，但需要 O(n) 空间。

不用哈希表的巧妙解法只有三步：

> **① 在每个原节点后面插入一个拷贝节点 → ② 设置拷贝节点的 random → ③ 分离交错链表**

### 三步图解

```
原始链表（r 表示 random 指针）：

        7 → 13 → 11 → 10 → 1
        ↓     ↙         ↙
       null   7         11
```

#### Step 1：穿插新节点

在每个原节点后面插入它的拷贝节点：

```
        7 → 7' → 13 → 13' → 11 → 11' → 10 → 10' → 1 → 1'
```

`cur.next = new Node(cur.val, cur.next)`——赋值给 `cur.next` 时，`cur.next` 原本指向的下一个原节点作为第二个参数传入构造函数。

#### Step 2：设置 random

再次遍历**原节点**（每隔一个节点跳一次），如果原节点的 random 不为 null，则它的拷贝节点的 random 就是 `cur.random.next`：

```
        7 → 7' → 13 → 13' → 11 → 11' → 10 → 10' → 1 → 1'
        ↓    ↓     ↙    ↙          ↙    ↙     ↙    ↙
       null null  7    7'         11   11'   11   11'
```

#### Step 3：分离

把交错链表拆成两条独立链表，同时恢复原链表的 next：

```
原链表： 7 → 13 → 11 → 10 → 1
新链表： 7' → 13' → 11' → 10' → 1'
```

### 代码实现

```java
class Solution {
    public Node copyRandomList(Node head) {
        if (head == null) return null;

        // Step 1: 在每个原节点后面插入拷贝节点
        for (Node cur = head; cur != null; cur = cur.next.next) {
            cur.next = new Node(cur.val, cur.next);
        }

        // Step 2: 设置拷贝节点的 random
        for (Node cur = head; cur != null; cur = cur.next.next) {
            if (cur.random != null) {
                cur.next.random = cur.random.next;
            }
        }

        // Step 3: 分离交错链表
        Node dummy = new Node(0);
        Node tail = dummy;
        for (Node cur = head; cur != null; cur = cur.next, tail = tail.next) {
            Node copy = cur.next;      // 拷贝节点
            tail.next = copy;          // 把拷贝节点加入新链表
            cur.next = copy.next;      // 恢复原节点的 next
        }

        return dummy.next;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 三次遍历 |
| 🧠 空间复杂度 | **O(1)**（不计输出链表）— 没有使用哈希表或其他额外数据结构 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 穿插法（交错链表 + 分离） |
| 算法类型 | 链表、深拷贝、指针操作 |
| 核心技巧 | **在原链表上插入拷贝节点，让 random 映射关系通过 `cur.random.next` 自然建立** |
| 适用场景 | 带随机指针的链表深拷贝 |
| 其他写法 | 哈希表法（更直观，但空间 O(n)） |
| 易错点 | Step 3 分离时 `cur.next = copy.next` 不能省，否则原链表被破坏；`cur.random` 可能为 null，必须判空 |

### 一句话记住

> **「插拷贝、设随机、再分离——三步走，不用哈希表。」**

<p align="right"><i>练习日期：2026-07-12</i></p>
