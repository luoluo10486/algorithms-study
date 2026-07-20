# Day 11 — 栈与队列（代码随想录）

## 题目：用栈实现队列（Implement Queue using Stacks）

**LeetCode 232 | 代码随想录 | 难度：🟢 简单**

### 题目描述

请你仅使用两个栈实现先入先出队列。队列应当支持一般队列支持的所有操作（`push`、`pop`、`peek`、`empty`）：

实现 `MyQueue` 类：

- `void push(int x)` — 将元素 x 推到队列的末尾
- `int pop()` — 从队列的开头移除并返回元素
- `int peek()` — 返回队列开头的元素
- `boolean empty()` — 如果队列为空，返回 `true`；否则返回 `false`

### 示例

```
输入：
["MyQueue", "push", "push", "peek", "pop", "empty"]
[[], [1], [2], [], [], []]
输出：
[null, null, null, 1, 1, false]

解释：
MyQueue myQueue = new MyQueue();
myQueue.push(1);       // queue is: [1]
myQueue.push(2);       // queue is: [1, 2] (left is front)
myQueue.peek();        // return 1
myQueue.pop();         // return 1, queue is [2]
myQueue.empty();       // return false
```

---

## 解法：双栈模拟（输入栈 × 输出栈）

### 思路

栈是**后进先出**（LIFO），队列是**先进先出**（FIFO）。用两个栈来模拟队列，核心就是**把一个栈的元素倒到另一个栈中，顺序就反转过来了**。

```
输入栈 stackIn：只管接收 push 进来的元素
输出栈 stackOut：只管 pop 和 peek 时弹出元素

关键操作 dumpstackIn：
  当 stackOut 为空时，把 stackIn 中所有元素弹出并压入 stackOut
  这样 stackOut 的栈顶就是最早进来的元素（队列的 front）
```

**为什么需要两个栈？**

```
push 1, 2, 3 到 stackIn:       stackIn = [1, 2, 3] (3 是栈顶)
把 stackIn 倒入 stackOut:      stackOut = [3, 2, 1] (1 成了栈顶)
pop stackOut → 弹出 1 ✅      先进来的先出去了
```

### 思考方式图解

```
操作序列：push(1) → push(2) → push(3) → pop() → pop() → push(4) → pop()

步骤：
1. push(1):     stackIn=[1]                stackOut=[]
2. push(2):     stackIn=[1,2]              stackOut=[]
3. push(3):     stackIn=[1,2,3]            stackOut=[]

4. pop():
   → dumpstackIn: stackIn 倒入 stackOut → stackIn=[], stackOut=[3,2,1]
   → pop stackOut → 返回 1                stackOut=[3,2]

5. pop():
   → stackOut 不为空，直接 pop → 返回 2    stackOut=[3]

6. push(4):     stackIn=[4]                stackOut=[3]

7. pop():
   → stackOut 不为空，直接 pop → 返回 3    stackOut=[]
   （注意！不会触发 dumpstackIn，因为 stackOut 非空）

8. 如果再来 pop：
   → stackOut 为空，dumpstackIn → stackIn=[4] 倒入 stackOut=[4]
   → pop → 返回 4
```

**惰性倒换**的设计很精妙——stackOut 不为空时就不倒，避免了每次 pop 都做 O(n) 操作。

### 代码实现

```java
class MyQueue {

    Stack<Integer> stackIn;
    Stack<Integer> stackOut;

    public MyQueue() {
        stackIn = new Stack<>();
        stackOut = new Stack<>();
    }

    // 核心：将 stackIn 中的元素倒入 stackOut（仅在 stackOut 为空时执行）
    private void dumpstackIn() {
        if (!stackOut.isEmpty()) return;        // stackOut 还有货，不倒
        while (!stackIn.isEmpty()) {
            stackOut.push(stackIn.pop());       // 倒！顺序反转
        }
    }

    public void push(int x) {
        stackIn.push(x);
    }

    public int pop() {
        dumpstackIn();
        return stackOut.pop();
    }

    public int peek() {
        dumpstackIn();
        return stackOut.peek();
    }

    public boolean empty() {
        return stackIn.isEmpty() && stackOut.isEmpty();
    }
}
```

### 复杂度分析

| 操作 | 时间复杂度 | 说明 |
|------|-----------|------|
| `push` | **O(1)** | 直接入 stackIn |
| `pop` | **均摊 O(1)** | 大多数情况下 stackOut 已有元素；最坏情况需要倒入（O(n)）但每个元素只倒一次 |
| `peek` | **均摊 O(1)** | 同 pop |
| `empty` | **O(1)** | 检查两个栈是否都为空 |
| 🧠 空间 | **O(n)** | 两个栈总容量 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 核心思想 | **两个栈倒来倒去——输入栈负责收，输出栈负责出，一倒顺序就反转** |
| `dumpstackIn` 的触发条件 | **只在 stackOut 为空时才倒**——惰性倒换，每个元素最多倒一次 |
| 设计模式 | **延迟执行**（lazy）：不到需要 pop/peek 的时候不倒，不倒多余的 |
| 关联题目 | [225. 用队列实现栈](https://leetcode.cn/problems/implement-stack-using-queues/)——反过来，也用两个队列模拟栈 |
| 易错点 | `peek` 和 `pop` 之前都要调用 `dumpstackIn`，不能只在 pop 里调；`empty` 要检查两个栈 |

### 一句话记住

> **「入栈只管收，出栈空了再倒——一倒顺序就反了。」**

---

## 题目：用队列实现栈（Implement Stack using Queues）

**LeetCode 225 | 代码随想录 | 难度：🟢 简单**

### 题目描述

请你仅使用两个队列实现一个后入先出（LIFO）的栈，并支持普通栈的全部四种操作（`push`、`pop`、`top`、`empty`）。

实现 `MyStack` 类：

- `void push(int x)` — 将元素 x 压入栈顶
- `int pop()` — 移除并返回栈顶元素
- `int top()` — 返回栈顶元素
- `boolean empty()` — 如果栈是空的，返回 `true`；否则返回 `false`

### 示例

```
输入：
["MyStack", "push", "push", "top", "pop", "empty"]
[[], [1], [2], [], [], []]
输出：
[null, null, null, 2, 2, false]

解释：
MyStack myStack = new MyStack();
myStack.push(1);
myStack.push(2);
myStack.top();    // return 2
myStack.pop();    // return 2
myStack.empty();  // return false
```

---

## 解法：单队列法（push 时旋转）

### 思路

栈是**后进先出**（LIFO），队列是**先进先出**（FIFO）。用队列模拟栈，核心是**在 push 的时候就把顺序调整好**——新元素入队后，把之前的所有元素重新入队（移到新元素后面），这样队头就始终是栈顶。

```
与「用栈实现队列」对比：

                    用栈实现队列          用队列实现栈
    ──────────────────────────────────────────────────
    数据结构        双栈                  单队列
    调整时机        pop/peek 时惰性倒换    push 时立即旋转
    核心操作        dumpstackIn           旋转：新元素入队后，把前面的都移到后面
```

**为什么可以用一个队列？**因为在 push 时做旋转后，队首始终是栈顶。pop 和 top 就是取队首，empty 就是队空，不需要第二个队列。

### 思考方式图解

```
push(1):
  queue = [1]

push(2):
  queue.offer(2)       →  queue = [1, 2]
  size=2, 旋转 1 次:
    queue.offer(poll()) → queue = [2, 1]

push(3):
  queue.offer(3)       →  queue = [2, 1, 3]
  size=3, 旋转 2 次:
    queue.offer(poll()) → queue = [1, 3, 2]
    queue.offer(poll()) → queue = [3, 2, 1]

top():  返回 queue.peek() = 3 ✅
pop():  返回 queue.poll() = 3 ✅
        queue = [2, 1]
```

### 代码实现

```java
class MyStack {

    Queue<Integer> queue;

    public MyStack() {
        queue = new LinkedList<>();
    }

    // 核心：push 时旋转——新元素入队后，把前面的元素都移到后面
    public void push(int x) {
        queue.offer(x);
        int size = queue.size();
        while (size-- > 1) {              // 把前 size-1 个元素移到新元素后面
            queue.offer(queue.poll());
        }
    }

    public int pop() {
        return queue.poll();
    }

    public int top() {
        return queue.peek();
    }

    public boolean empty() {
        return queue.isEmpty();
    }
}
```

### 复杂度分析

| 操作 | 时间复杂度 | 说明 |
|------|-----------|------|
| `push` | **O(n)** | 需要将前 n-1 个元素旋转到新元素后面 |
| `pop` | **O(1)** | 直接从队首弹出 |
| `top` | **O(1)** | 直接看队首 |
| `empty` | **O(1)** | 检查队列是否为空 |
| 🧠 空间 | **O(n)** | 一个队列 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 核心思想 | **push 时旋转——新元素入队后，把前面的全部移到后面，让新元素始终在队首** |
| 为什么一个队列就够了 | 因为 push 时就已经完成了调整，pop/top 直接取队首即可 |
| 与 232 的对比 | 232 用双栈 + 惰性倒换（pop 时调整）；225 用单队列 + 主动旋转（push 时调整） |
| 关联题目 | [232. 用栈实现队列](day11-栈与队列代码随想录.md)——反过来，用栈模拟队列 |
| 易错点 | `push` 里的 `while (size-- > 1)` 是先判断再自减，循环体里 size 已经减了 1 |

### 一句话记住

> **「新元素入队后，把前面的都挪到后面——栈顶就在队首了。」**

---

## 题目：有效的括号（Valid Parentheses）

**LeetCode 20 | 代码随想录 | 难度：🟢 简单**

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

## 解法：栈 + 匹配时反向入栈

### 思路

括号匹配是栈的经典应用——**遇到左括号就把对应的右括号入栈，遇到右括号就弹出栈顶检查是否匹配**。

核心思想：**与其存左括号，不如存它期望匹配的右括号**。

```
遇到 '(' → 栈 push ')' —— 期望将来遇到 ')'
遇到 '[' → 栈 push ']' —— 期望将来遇到 ']'
遇到 '{' → 栈 push '}' —— 期望将来遇到 '}'
遇到 ')' / ']' / '}' → 弹出栈顶 ch
   如果栈为空 → 多了个右括号，false
   如果 ch != 当前字符 → 括号不匹配，false
```

这种存"期望值"的技巧让匹配判断变成了一个简单的 `ch != deque.pop()`。

### 思考方式图解

```
s = "([{}])"

i=0: '(' → push ')',          deque = [')']
i=1: '[' → push ']',          deque = [')', ']']
i=2: '{' → push '}',          deque = [')', ']', '}']
i=3: '}' → pop → '}' == '}' ✅ deque = [')', ']']
i=4: ']' → pop → ']' == ']' ✅ deque = [')']
i=5: ')' → pop → ')' == ')' ✅ deque = []

遍历结束，栈为空 → true ✅
```

```
s = "([)]"

i=0: '(' → push ')',          deque = [')']
i=1: '[' → push ']',          deque = [')', ']']
i=2: ')' → pop → ']' != ')' ❌ → false
```

```
s = "())"

i=0: '(' → push ')',          deque = [')']
i=1: '(' → push ')',          deque = [')', ')']
i=2: ')' → pop → ')' == ')' ✅ deque = [')']
遍历结束，栈不为空 → false ❌（多了一个左括号）
```

### 代码实现

```java
class Solution {
    public boolean isValid(String s) {
        Deque<Character> deque = new LinkedList<>();

        for (int i = 0; i < s.length(); i++) {
            char ch = s.charAt(i);

            if (ch == '(') {
                deque.push(')');           // 存期望的右括号
            } else if (ch == '[') {
                deque.push(']');
            } else if (ch == '{') {
                deque.push('}');
            } else if (deque.isEmpty() || ch != deque.pop()) {
                // 栈为空（多了右括号）或 不匹配
                return false;
            }
        }

        return deque.isEmpty();             // 栈不为空说明多了左括号
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 一次遍历 |
| 🧠 空间复杂度 | **O(n)** — 最坏情况下栈中全是左括号 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 栈 + 反向入栈（存储期望值） |
| 算法类型 | 栈、字符串 |
| 核心技巧 | **遇到左括号就把对应的右括号入栈，后续只需用 `ch != pop()` 判断是否匹配** |
| 三种失败情况 | 1. 右括号多余（栈空） 2. 左括号多余（遍历结束栈不空） 3. 括号不匹配（`ch != pop()`） |
| 适用场景 | 括号匹配、HTML 标签解析、代码语法检查 |
| 易错点 | 用 `Deque` 而不是 `Stack`（`Stack` 是同步类，性能差）；右括号的 `else if` 里要同时检查栈空和匹配两个条件 |

### 一句话记住

> **「左括号存期望，右括号弹出比——栈空或不匹配就 false。」**

---

## 题目：删除字符串中的所有相邻重复项（Remove All Adjacent Duplicates In String）

**LeetCode 1047 | 代码随想录 | 难度：🟢 简单**

### 题目描述

给出由小写字母组成的字符串 `s`，**重复项删除操作**会选择两个相邻且相同的字母，并删除它们。

在 `s` 上反复执行重复项删除操作，直到无法继续删除。

返回完成所有重复项删除操作后的最终字符串。

### 示例

```
输入：s = "abbaca"
输出："ca"
解释：
"abbaca" → "aaca"（删除 bb）
"aaca" → "ca"（删除 aa）
最终结果："ca"

输入：s = "azxxzy"
输出："ay"
```

---

## 解法：栈模拟删除

### 思路

这道题就是**栈的经典匹配场景**——遍历字符串，如果当前字符与栈顶相同就消掉（出栈），否则入栈。

```
栈的三种消消乐对比：

             括号匹配（20）     相邻重复删除（1047）
    ──────────────────────────────────────────
    入栈时机    左括号          当前字符 ≠ 栈顶
    出栈时机    右括号（匹配）   当前字符 == 栈顶
    栈内元素    期望的右括号     待消除的字符
    判断依据    ch != pop()     ch == peek()
```

### 思考方式图解

```
s = "abbaca"

i=0: 'a' → 栈空，入栈         deque = [a]
i=1: 'b' → peek=a ≠ 'b'，入栈   deque = [a, b]
i=2: 'b' → peek=b == 'b'，出栈   deque = [a]
i=3: 'a' → peek=a == 'a'，出栈   deque = []
i=4: 'c' → 栈空，入栈         deque = [c]
i=5: 'a' → peek=c ≠ 'a'，入栈   deque = [c, a]

栈内元素从底到顶：c → a
结果："ca" ✅
```

### 代码实现

```java
class Solution {
    public String removeDuplicates(String s) {
        Deque<Character> deque = new LinkedList<>();

        for (int i = 0; i < s.length(); i++) {
            char ch = s.charAt(i);

            // 栈为空 或 栈顶与当前字符不同 → 入栈
            if (deque.isEmpty() || deque.peek() != ch) {
                deque.push(ch);
            } else {
                // 栈顶与当前字符相同 → 消除（出栈）
                deque.pop();
            }
        }

        // 从栈底到栈顶拼接成字符串
        String str = "";
        while (!deque.isEmpty()) {
            str = deque.pop() + str;    // pop 从栈顶弹出，所以插在前面
        }
        return str;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 一次遍历 |
| 🧠 空间复杂度 | **O(n)** — 最坏情况下没有重复元素 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 栈模拟消除 |
| 算法类型 | 栈、字符串 |
| 核心技巧 | **遍历字符串，与栈顶相同就消（出栈），不同则入（压栈）——类似消消乐** |
| 与 20 的关系 | 括号匹配的简化版——不需要匹配不同的括号类型，只需检查是否相同 |
| 拼接字符串 | `str = deque.pop() + str` 是因为栈是后进先出，需要反转顺序 |
| 优化方向 | 可以直接用 `StringBuilder` 当栈用（`sb.deleteCharAt(last)`），省去最后的拼接操作 |
| 易错点 | `peek()` 只查看栈顶不弹出，`pop()` 查看并弹出；循环结束后栈内元素是反的（要反着拼） |

### 一句话记住

> **「栈顶相同就消掉，不同就压栈——消消乐一题学会栈。」**

---

## 题目：逆波兰表达式求值（Evaluate Reverse Polish Notation）

**LeetCode 150 | 代码随想录 | 难度：🟡 中等**

### 题目描述

给你一个字符串数组 `tokens`，表示一个根据**逆波兰表示法**（Reverse Polish Notation, RPN）表达的算术表达式。

请你计算该表达式。返回一个表示表达式值的整数。

**注意**：
- 有效的算符为 `'+'`、`'-'`、`'*'`、`'/'`
- 每个操作数（运算对象）都是一个整数
- 两个整数之间的除法总是向**零截断**
- 表达式不含除零运算

### 示例

```
输入：tokens = ["2","1","+","3","*"]
输出：9
解释：(2 + 1) * 3 = 9

输入：tokens = ["4","13","5","/","+"]
输出：6
解释：4 + (13 / 5) = 6

输入：tokens = ["10","6","9","3","+","-11","*","/","*","17","+","5","+"]
输出：22
```

---

## 解法：栈模拟计算

### 思路

逆波兰表达式的最大特点就是**没有括号，运算符跟在操作数后面**——这天然适合用栈来计算。

```
遇到数字 → 入栈
遇到运算符 → 弹出两个操作数，计算，结果入栈
```

**注意运算顺序**：先弹出的是 `a`（右操作数），后弹出的是 `b`（左操作数）。

```
对于减法：  b - a  （不是 a - b）
对于除法：  b / a  （不是 a / b）
```

### 思考方式图解

```
tokens = ["2","1","+","3","*"]

遍历：
  "2" → 数字，入栈           deque = [2]
  "1" → 数字，入栈           deque = [2, 1]
  "+" → 弹出 a=1, b=2，计算 2+1=3，入栈  deque = [3]
  "3" → 数字，入栈           deque = [3, 3]
  "*" → 弹出 a=3, b=3，计算 3*3=9，入栈  deque = [9]

结果：9 ✅
```

```
tokens = ["4","13","5","/","+"]

遍历：
  "4"  → 数字，入栈           deque = [4]
  "13" → 数字，入栈           deque = [4, 13]
  "5"  → 数字，入栈           deque = [4, 13, 5]
  "/"  → 弹出 a=5, b=13，计算 13/5=2，入栈  deque = [4, 2]
  "+"  → 弹出 a=2, b=4，计算 4+2=6，入栈    deque = [6]

结果：6 ✅
```

### 代码实现

```java
class Solution {
    public int evalRPN(String[] tokens) {
        Deque<Integer> deque = new LinkedList<>();

        for (String s : tokens) {
            if (s.equals("+")) {
                int a = deque.pop();    // 右操作数
                int b = deque.pop();    // 左操作数
                deque.push(b + a);
            } else if (s.equals("-")) {
                int a = deque.pop();
                int b = deque.pop();
                deque.push(b - a);      // 注意：b - a，不是 a - b
            } else if (s.equals("*")) {
                int a = deque.pop();
                int b = deque.pop();
                deque.push(b * a);
            } else if (s.equals("/")) {
                int a = deque.pop();
                int b = deque.pop();
                deque.push(b / a);      // 注意：b / a，向零截断
            } else {
                deque.push(Integer.valueOf(s));  // 数字入栈
            }
        }

        return deque.pop();
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 一次遍历 |
| 🧠 空间复杂度 | **O(n)** — 栈中最多存放 n/2 个操作数 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 栈模拟计算器 |
| 算法类型 | 栈、数学 |
| 核心技巧 | **数字入栈，运算符弹出两个计算后结果入栈——逆波兰表达式天然适合栈** |
| 运算顺序 | 先弹出的是右操作数 `a`，后弹出的是左操作数 `b`，**减法和除法注意顺序：`b-a`、`b/a`** |
| 关联题目 | [224. 基本计算器](https://leetcode.cn/problems/basic-calculator/)——带括号的中缀表达式计算 |
| 易错点 | 除法向零截断是指 `-3/2 = -1`（不是向下取整的 `-2`）；Java 中 `int` 除法的向零截断符合要求 |

### 一句话记住

> **「数字入栈，符号弹出两个算——注意减法除法的顺序。」**

---

## 题目：滑动窗口最大值（Sliding Window Maximum）

**LeetCode 239 | 代码随想录 | 难度：🔴 困难**

### 题目描述

给你一个整数数组 `nums`，有一个大小为 `k` 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 `k` 个数字。滑动窗口每次只向右移动一位。

返回**每个滑动窗口中的最大值**。

### 示例

```
输入：nums = [1,3,-1,-3,5,3,6,7], k = 3
输出：[3,3,5,5,6,7]

输入：nums = [1], k = 1
输出：[1]
```

---

## 解法：单调队列（Deque）

### 思路

暴力解法是每个窗口扫一遍 O(n·k)，不可接受。

核心思想：**用一个双端队列维护当前窗口内"可能成为最大值的元素下标"，保持队列中的元素按值递减（单调递减）**。

```
单调递减队列的三条规则：

1. 入队（添加新元素）：
   while 队尾元素 ≤ 当前元素 → 队尾出队（它不可能是最大值了）
   当前元素入队（作为新的候选）

2. 出队（移除过期元素）：
   if 队首元素下标 < 窗口左边界 → 队首出队（不在窗口中了）

3. 记录结果：
   if 窗口已形成（i ≥ k-1）→ 队首元素是当前窗口最大值
```

**为什么用 Deque 存下标而不是值？** 因为存下标才能判断元素是否还在窗口内。

### 思考方式图解

```
nums = [1,3,-1,-3,5,3,6,7], k = 3

i=0: nums[0]=1, deque=[] → 入队 0              deque=[0]
i=1: nums[1]=3, 队尾 0→1 ≤ 3 → 移除0，入队1    deque=[1]
i=2: nums[2]=-1, 队尾 1→3 > -1 → 直接入队2     deque=[1,2]
     left=0, 队首 1≥0 → 有效
     left≥0 → ans[0]=nums[1]=3 ✅

i=3: nums[3]=-3, 队尾 2→-1 > -3 → 入队3       deque=[1,2,3]
     left=1, 队首 1≥1 → 有效
     left≥0 → ans[1]=nums[1]=3 ✅

i=4: nums[4]=5, 队尾 3→-3 ≤ 5 → 移除3
               队尾 2→-1 ≤ 5 → 移除2
               队尾 1→3  ≤ 5 → 移除1
     入队4                                    deque=[4]
     left=2, 队首 4≥2 → 有效
     left≥0 → ans[2]=nums[4]=5 ✅

i=5: nums[5]=3, 队尾 4→5 > 3 → 直接入队5       deque=[4,5]
     left=3, 队首 4≥3 → 有效
     left≥0 → ans[3]=nums[4]=5 ✅

i=6: nums[6]=6, 队尾 5→3 ≤ 6 → 移除5
               队尾 4→5 ≤ 6 → 移除4
     入队6                                    deque=[6]
     left=4, 队首 6≥4 → 有效
     left≥0 → ans[4]=nums[6]=6 ✅

i=7: nums[7]=7, 队尾 6→6 ≤ 7 → 移除6
     入队7                                    deque=[7]
     left=5, 队首 7≥5 → 有效
     left≥0 → ans[5]=nums[7]=7 ✅

结果：[3,3,5,5,6,7] ✅
```

### 代码实现

```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        int n = nums.length;
        int[] ans = new int[n - k + 1];
        Deque<Integer> deque = new LinkedList<>();   // 存下标，单调递减

        for (int i = 0; i < n; i++) {
            // 1. 入队：移除所有 ≤ 当前元素的队尾元素（它们不可能是最大值了）
            while (!deque.isEmpty() && nums[deque.getLast()] <= nums[i]) {
                deque.removeLast();
            }
            deque.addLast(i);                         // 当前元素入队（作为新的候选）

            // 2. 出队：移除已移出窗口的队首元素
            int left = i - k + 1;                     // 当前窗口左边界
            if (deque.getFirst() < left) {             // 队首元素下标 < 左边界 → 过期
                deque.removeFirst();
            }

            // 3. 记录结果：窗口形成后，队首就是当前窗口最大值
            if (left >= 0) {
                ans[left] = nums[deque.getFirst()];
            }
        }

        return ans;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 每个元素入队一次、出队最多一次 |
| 🧠 空间复杂度 | **O(k)** — 队列最多同时存放 k 个元素 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 单调队列（双端队列） |
| 算法类型 | 队列、滑动窗口 |
| 核心技巧 | **单调递减队列：新元素入队时清掉所有 ≤ 它的队尾，淘汰过期元素时看队首是否 < 左边界** |
| 为什么存下标 | 只有存下标才能判断元素是否过期（是否还在窗口内）；如果只存值，不知道位置 |
| 与 1047/20 的栈对比 | 之前是 LIFO 栈（后进先出）；这里是队列（先进先出），但保持了"单调性"约束 |
| 关联题目 | [59. 滑动窗口最大值 II](https://leetcode.cn/problems/hua-dong-chuang-kou-de-zui-da-zhi-lcof/)——同题 |

### 一句话记住

> **「单调递减队列，新来大的清队尾，过期小的清队首。」**

---

*练习日期：2026-07-20*
