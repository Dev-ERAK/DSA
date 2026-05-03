# Problem: Evaluate Postfix (Reverse Polish Notation) Expression

## Category
Stack

## Difficulty
Medium

---

## Problem Statement

You're given an arithmetic expression in **postfix** form (also called **Reverse Polish Notation**, or RPN). Evaluate it and return the result.

**Postfix** means operators come **after** their operands.

This is **LeetCode #150**.

### Examples

```
["2", "1", "+", "3", "*"]    →  9     ((2 + 1) * 3)
["4", "13", "5", "/", "+"]   →  6     (4 + (13 / 5))
```

### Wait — what is postfix?

Normal math (called **infix**) uses the form `2 + 3`. The operator `+` is *between* its operands.

**Postfix** writes the same expression as `2 3 +`. Operands first, then the operator. It looks weird but has a huge advantage: **no parentheses, no precedence rules.**

Examples of conversion:

```
infix: (2 + 1) * 3
postfix: 2 1 + 3 *

infix: 4 + 13 / 5
postfix: 4 13 5 / +     (because of operator precedence, /  binds tighter)
```

---

## Clarifying Questions to Ask the Interviewer

1. **How does division round?** "Truncate toward zero" (LeetCode convention) — `-7 / 2 = -3`, **not** `-4`.
2. **Will operands always be integers?** Yes (LeetCode 150).
3. **Are operators always binary (+, −, ×, ÷)?** Yes.
4. **Division by zero?** Usually undefined — clarify.

---

## Approach — Stack

### The Big Idea

Postfix is *designed* to be evaluated with a stack. Walk the tokens left to right:

- If the token is a **number**, push it onto the stack.
- If the token is an **operator**, pop the top **two** numbers (`b` then `a`), compute `a op b`, and push the result back.

When you've processed all tokens, the stack should contain exactly **one** number — the answer.

### Why postfix is easy with a stack

In postfix, when you see an operator, both its operands have already been emitted (and therefore are on the stack — the most recent two). No need to track parentheses, no need to handle precedence.

### Walking through `["2", "1", "+", "3", "*"]`

```
Token "2": push 2.    Stack: [2]
Token "1": push 1.    Stack: [2, 1]
Token "+": pop 1 and 2. Compute 2 + 1 = 3. Push 3. Stack: [3]
Token "3": push 3.    Stack: [3, 3]
Token "*": pop 3 and 3. Compute 3 * 3 = 9. Push 9. Stack: [9]

End of input. Answer = 9.
```

### Critical detail — order of popping

When you pop two values for `a - b` or `a / b`, the **first** value you pop is `b`, the **second** is `a`. Because of LIFO order, the most recent push (the right operand) comes off first.

```
For "a - b" written as a, b, '-':
  push a    → stack: [a]
  push b    → stack: [a, b]
  pop b     ← top
  pop a     ← below
  result = a - b   (NOT b - a!)
```

This matters for non-commutative operators (`-` and `/`). Plus and times you can do in any order.

### Walking through `["4", "13", "5", "/", "+"]`

```
"4":  push 4.     Stack: [4]
"13": push 13.    Stack: [4, 13]
"5":  push 5.     Stack: [4, 13, 5]
"/":  pop 5 (b), pop 13 (a). 13 / 5 = 2 (truncate). Push. Stack: [4, 2]
"+":  pop 2, pop 4. 4 + 2 = 6. Push. Stack: [6]

Answer: 6.
```

---

## Time Complexity

**O(n)** — each token causes at most one push and one pop.

## Space Complexity

**O(n)** — the stack can hold up to about half the tokens at peak.

---

## Code (Java)

```java
public int evalRPN(String[] tokens) {
    Deque<Integer> stack = new ArrayDeque<>();

    for (String t : tokens) {
        switch (t) {
            case "+":
            case "-":
            case "*":
            case "/":
                int b = stack.pop();      // right operand (popped first)
                int a = stack.pop();      // left operand
                stack.push(apply(a, b, t));
                break;
            default:
                stack.push(Integer.parseInt(t));
        }
    }
    return stack.pop();
}

private int apply(int a, int b, String op) {
    return switch (op) {
        case "+" -> a + b;
        case "-" -> a - b;
        case "*" -> a * b;
        case "/" -> a / b;        // Java int division truncates toward zero
        default -> throw new IllegalArgumentException(op);
    };
}
```

---

## Code (Python)

```python
class Solution:
    def evalRPN(self, tokens):
        stack = []
        for t in tokens:
            if t in {"+", "-", "*", "/"}:
                b = stack.pop()                # right operand
                a = stack.pop()                # left operand
                if t == "+":   stack.append(a + b)
                elif t == "-": stack.append(a - b)
                elif t == "*": stack.append(a * b)
                else:
                    # Python's // floors (rounds down). We need truncation
                    # toward zero. int(a / b) does that.
                    stack.append(int(a / b))
            else:
                stack.append(int(t))
        return stack[0]
```

> **Python gotcha:** `-7 // 2 = -4` (floor division), but the problem wants `-3` (truncate toward zero). Always use `int(a / b)` here.

---

## Edge Cases

| Case | Result |
|---|---|
| Single number | That number. |
| Negative numbers (`"-3"`) | `Integer.parseInt` / `int()` handles the sign correctly. |
| Division by zero | Throws — handle if needed. |
| Multi-digit operands (`"13"`) | Handled by `parseInt` — no special tokenizing. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Convert infix to postfix."**
   Use the **Shunting-Yard algorithm** by Dijkstra. It uses a stack and an output queue, handling operator precedence and parentheses.

2. **"Evaluate prefix expressions (operator first)?"**
   Same idea, but walk the tokens **right to left**. Push numbers; when you hit an operator, pop two values and compute.

3. **"Why do compilers and calculators use postfix internally?"**
   Postfix is unambiguous — no precedence, no parentheses. It maps directly to stack-machine operations, which is how many virtual machines (e.g., the JVM) work.

4. **"What about more operators (^, %, function calls)?"**
   Same algorithm — extend the operator handling. Function calls of arity > 2 just pop more operands.

---

## Key Lessons from This Problem

1. **Stacks are perfect for postfix expressions.** That's not a coincidence — postfix was *designed* for stack evaluation.
2. **Pop order matters** for non-commutative operators. Always remember: top of stack = right operand.
3. **Truncation vs floor division** — these are different in many languages. Be precise about which one your problem requires.
