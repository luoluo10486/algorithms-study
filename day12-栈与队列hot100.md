# Day 12 — 栈与队列 Hot100

## 题目：有效的括号（Valid Parentheses）

**LeetCode 20 | 栈 Hot100 | 难度：🟢 简单**

### 题目描述

给定一个只包括 `'('`、`')'`、`'{'`、`'}'`、`'['`、`']'` 的字符串 `s`，判断字符串是否有效。

有效字符串需满足：

1. 左括号必须用相同类型的右括号闭合
2. 左括号必须以正确的顺序闭合
3. 每个右括号都有一个对应的相同类型的左括号

### 示例

```
输入：s = "()"
输出：true

输入：s = "()[]{}"
输出：true

输入：s = "(]"
输出：false

输入：s = "([)]"
输出：false

输入：s = "{[]}"
输出：true
```

---

## 解法：栈 + 反向入栈

### 思路

括号匹配是栈最经典的应用。核心技巧：**遇到左括号，存入它期望匹配的右括号；遇到右括号，弹出栈顶比较是否一致**。

### 思考方式图解

```
s = "([{}])"

i=0: '(' → push ')',          deque = [')']
i=1: '[' → push ']',          deque = [')', ']']
i=2: '{' → push '}',          deque = [')', ']', '}']
i=3: '}' → pop → '}' ✅       deque = [')', ']']
i=4: ']' → pop → ']' ✅       deque = [')']
i=5: ')' → pop → ')' ✅       deque = []

结果：true ✅
```

### 代码实现

```java
class Solution {
    public boolean isValid(String s) {
        Deque<Character> deque = new LinkedList<>();

        for (int i = 0; i < s.length(); i++) {
            char ch = s.charAt(i);

            if (ch == '(') {
                deque.push(')');
            } else if (ch == '[') {
                deque.push(']');
            } else if (ch == '{') {
                deque.push('}');
            } else if (deque.isEmpty() || ch != deque.pop()) {
                return false;
            }
        }

        return deque.isEmpty();
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 一次遍历 |
| 🧠 空间复杂度 | **O(n)** — 栈中最多存储 n/2 个字符 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 栈 + 反向入栈 |
| 核心技巧 | **左括号存期望的右括号，右括号弹出对比 `ch != pop()`** |
| 三种失败情况 | 右括号多余（栈空）、左括号多余（遍历结束栈不空）、不匹配（`ch != pop()`） |

### 一句话记住

> **「左括号存期望，右括号弹出比——栈空或不匹配就 false。」**

---

## 题目：最小栈（Min Stack）

**LeetCode 155 | 栈 Hot100 | 难度：🟡 中等**

### 题目描述

设计一个支持 `push`、`pop`、`top` 操作，并能在**常数时间**内检索到最小元素的栈。

实现 `MinStack` 类：

- `MinStack()` — 初始化堆栈对象
- `void push(int val)` — 将元素 val 推入堆栈
- `void pop()` — 删除堆栈顶部的元素
- `int top()` — 获取堆栈顶部的元素
- `int getMin()` — 获取堆栈中的最小元素

### 示例

```
输入：
["MinStack","push","push","push","getMin","pop","top","getMin"]
[[],[-2],[0],[-3],[],[],[],[]]

输出：
[null,null,null,null,-3,null,0,-2]

解释：
MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.getMin();   // 返回 -3
minStack.pop();
minStack.top();      // 返回 0
minStack.getMin();   // 返回 -2
```

---

## 解法：辅助栈（每个元素携带当前最小值）

### 思路

如果只用普通栈，`getMin()` 需要遍历整个栈，做不到 O(1)。

核心思想：**每个元素入栈时，同时记录"入栈这一刻的最小值"**——栈中存 `[当前值, 当前最小值]` 的二元组。

```
push 时：当前最小值 = min(栈顶的最小值, 当前值)
pop 时：直接弹出（最小值信息跟着一起走）
getMin 时：直接返回栈顶二元组的最小值部分
```

**边界处理**：初始化时压入一个哨兵元素 `[0, Integer.MAX_VALUE]`，这样空栈时 `getMin()` 不用判空——栈永远不为空。

### 思考方式图解

```
操作序列：

MinStack()       → st = [(0, MAX_VALUE)]

push(-2)         → st = [(0, MAX), (-2, min(MAX,-2)=-2)]
push(0)          → st = [(0, MAX), (-2, -2), (0, min(-2,0)=-2)]
push(-3)         → st = [(0, MAX), (-2,-2), (0,-2), (-3, min(-2,-3)=-3)]

getMin()         → 返回栈顶的 min = -3 ✅

pop()            → 弹出 (-3,-3)，st = [(0, MAX), (-2,-2), (0,-2)]
top()            → 返回栈顶的 val = 0 ✅
getMin()         → 返回栈顶的 min = -2 ✅
```

### 代码实现

```java
class MinStack {
    // 栈中存二元组：[当前值, 当前最小值]
    private final Deque<int[]> st = new ArrayDeque<>();

    // 哨兵：栈底放一个无效值来避免空栈判断
    public MinStack() {
        st.push(new int[]{0, Integer.MAX_VALUE});
    }

    public void push(int value) {
        // 当前最小值 = min(栈顶最小值, 当前值)
        st.push(new int[]{value, Math.min(getMin(), value)});
    }

    public void pop() {
        st.pop();
    }

    public int top() {
        return st.peek()[0];
    }

    public int getMin() {
        return st.peek()[1];
    }
}
```

### 复杂度分析

| 操作 | 时间复杂度 |
|------|-----------|
| `push` | **O(1)** |
| `pop` | **O(1)** |
| `top` | **O(1)** |
| `getMin` | **O(1)** |
| 🧠 空间 | **O(n)** — 每个元素多存一个 int |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 辅助栈（每个元素携带当前最小值） |
| 算法类型 | 栈、设计 |
| 核心技巧 | **栈中存二元组 `[val, min]`，push 时同步更新最小值，getMin 直接返回** |
| 哨兵设计 | 构造时压入 `[0, MAX_VALUE]`，栈永不空，省去判空逻辑 |
| 关联设计题 | [232. 用栈实现队列](day11-栈与队列代码随想录.md)、[225. 用队列实现栈](day11-栈与队列代码随想录.md） |
| 易错点 | `getMin()` 在栈为空时不能调用——但哨兵设计完美解决了这个问题 |

### 一句话记住

> **「入栈带个最小值，getMin 直接栈顶取。」**

---

*练习日期：2026-07-21*
