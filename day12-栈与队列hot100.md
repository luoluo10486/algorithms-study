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

## 题目：字符串解码（Decode String）

**LeetCode 394 | 栈 Hot100 | 难度：🟡 中等**

### 题目描述

给定一个经过编码的字符串，返回它解码后的字符串。

编码规则为：`k[encoded_string]`，表示其中方括号内部的 `encoded_string` 正好重复 `k` 次。注意 `k` 保证为正整数。

你可以认为输入字符串总是有效的；输入字符串中没有额外的空格，且输入的方括号总是符合格式要求的。

### 示例

```
输入：s = "3[a]2[bc]"
输出："aaabcbc"

输入：s = "3[a2[c]]"
输出："accaccacc"

输入：s = "2[abc]3[cd]ef"
输出："abcabccdcdcdef"
```

---

## 解法：双栈（或单栈存 Pair）

### 思路

字符串解码的核心是处理**嵌套**的 `k[...]` 结构——栈天生适合处理这种逐层展开的问题。

核心思想：**遇到 `[` 时，把当前的字符串和数字存入栈中，重置 StringBuilder 和 k 开始处理子问题；遇到 `]` 时，弹出栈顶的 Pair，将当前累积的字符串重复 k 次拼到栈顶字符串后面。**

```
关键数据结构：栈中存 Pair("当前层已拼好的字符串", 重复次数)
数字 k：用一个变量累积，处理多位数（如 "12[abc]"）
字符串 res：用 StringBuilder 累积当前层的字符
```

### 思考方式图解

```
s = "3[a2[c]]"

遍历过程：
  '3' → k = 3
  '[' → push ("", 3), 重置 res="", k=0
  'a' → res = "a"
  '2' → k = 2
  '[' → push ("a", 2), 重置 res="", k=0
  'c' → res = "c"
  ']' → pop ("a", 2) → res = "a" + "c"×2 = "acc"
  ']' → pop ("", 3)  → res = "" + "acc"×3 = "accaccacc"

结果："accaccacc" ✅
```

```
s = "2[abc]3[cd]ef"

遍历过程：
  '2' → k = 2
  '[' → push ("", 2)
  "abc" → res = "abc"
  ']' → pop ("", 2) → res = "abcabc"
  '3' → k = 3
  '[' → push ("abcabc", 3)
  "cd" → res = "cd"
  ']' → pop ("abcabc", 3) → res = "abcabc" + "cd"×3 = "abcabccdcdcd"
  "ef" → res = "abcabccdcdcdef"

结果："abcabccdcdcdef" ✅
```

### 代码实现

```java
class Solution {
    // 栈中存 Pair：当前层已拼好的字符串 和 重复次数
    private record Pair(String s, int k) {}

    public String decodeString(String s) {
        Deque<Pair> stack = new ArrayDeque<>();
        StringBuilder res = new StringBuilder();
        int k = 0;

        for (char c : s.toCharArray()) {
            if (Character.isLetter(c)) {
                res.append(c);                        // 普通字母，追加到当前层
            } else if (Character.isDigit(c)) {
                k = k * 10 + (c - '0');                // 累积多位数
            } else if (c == '[') {
                stack.push(new Pair(res.toString(), k)); // 保存现场
                res.setLength(0);                      // 开始子问题
                k = 0;
            } else { // c == ']'
                Pair p = stack.pop();                  // 恢复上一层的字符串和 k
                // 当前 res 重复 p.k 次，拼到 p.s 后面
                res = new StringBuilder(p.s).repeat(res, p.k);
            }
        }

        return res.toString();
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 每个字符处理一次，展开后的字符串拼接耗时与最终长度成正比 |
| 🧠 空间复杂度 | **O(n)** — 栈 + 展开后的字符串 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 栈模拟解码（双栈思想） |
| 算法类型 | 栈、字符串 |
| 核心技巧 | **遇到 `[` 存现场入栈，遇到 `]` 恢复现场展开重复——嵌套结构天然适合栈** |
| 关键细节 | `k * 10 + (c - '0')` 处理多位数；`res.setLength(0)` 重置 StringBuilder 比 `new` 更省性能 |
| 另一种写法 | 可以用两个栈（数字栈 + 字符串栈）等价实现，但 Pair 封装更清晰 |
| 易错点 | `]` 解码时：`p.s` 是上一层已拼好的字符串，`res` 是当前层要重复的内容，顺序别搞反 |

### 一句话记住

> **「左括号存现场，右括号展开重复——栈里放 Pair，嵌套不怕。」**

---

*练习日期：2026-07-21*
