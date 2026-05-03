# Problem: Min Stack — Get Minimum in O(1)

## Category
Stack / Design

## Difficulty
Medium

---

## Problem Statement

Design a stack that supports four operations, **all in O(1)**:

- `push(x)` — push x onto the stack.
- `pop()` — remove and return the top.
- `top()` — peek at the top.
- `getMin()` — return the minimum value currently in the stack.

This is **LeetCode #155**.

The hard part: **`getMin()` must be O(1)** — you can't scan the whole stack each time.

### Example

```
push(-2); push(0); push(-3);
getMin();    →  -3
pop();
top();       →  0
getMin();    →  -2
```

---

## Clarifying Questions to Ask the Interviewer

1. **All operations strictly O(1)?** Yes — that's the spec.
2. **Can the stack hold duplicates?** Yes (handle them carefully — see below).
3. **Negative numbers?** Yes.
4. **Behavior on empty stack?** Undefined — clarify.

---

## Approach

### Why a single stack isn't enough

A regular stack lets you `push`/`pop`/`top` in O(1), but `getMin()` would require scanning everything — O(n).

We need to **track the minimum at every level of the stack**, somehow.

### The Big Idea — Auxiliary Stack

Maintain a second stack, `minStack`. Its top **always equals the current minimum** of the main stack.

When you push a new value `x`:
- If `x ≤ currentMin` (i.e., `x` is the new minimum), push `x` onto `minStack`.
- Else, leave `minStack` alone — the current min is still valid.

When you pop:
- If the popped value equals `minStack.top()`, also pop `minStack`.
- Else leave `minStack` alone.

`getMin()` is just `minStack.top()` — O(1).

### Why `<=`, not `<`?

Consider duplicates. If you push `[2, 2, 1]`:

```
With <  (push only when strictly smaller):
  Main: [2, 2, 1]    minStack: [2, 1]

Now pop the 1:
  Main: [2, 2]    minStack: [2]      OK so far

Now pop a 2:
  Main: [2]    minStack: [2]
  But because we used <, we never re-pushed when we saw the second 2.
  Now we pop another 2 — but minStack still has its only 2.
  We'd need to NOT pop minStack now, but how do we know?
```

The simpler fix: **push to minStack on `<=`** so duplicates of the min are also tracked. Then `pop` always pops `minStack` whenever the popped value equals the min.

```
With <= :
  Main: [2, 2, 1]    minStack: [2, 2, 1]      (each "min seen" is recorded)

Pop 1: main=[2,2], minStack=[2,2]    ← we popped the 1 from minStack too
Pop 2: main=[2],   minStack=[2]      ← popped a 2 from minStack (matches)
Pop 2: main=[],    minStack=[]       ← pop the last 2
```

Clean and correct.

### Walking through the example

```
push(-2):
  Main: [-2]       minStack: [-2]    (first value, push to both)
push(0):
  Main: [-2, 0]    minStack: [-2]    (0 > -2, don't push)
push(-3):
  Main: [-2, 0, -3]    minStack: [-2, -3]   (-3 ≤ -2, push)

getMin() → minStack.top() = -3   ✓

pop() → returns -3
  Main: [-2, 0]    minStack: [-2]   (popped value = minStack top → pop both)

top() → 0   ✓

getMin() → minStack.top() = -2   ✓
```

---

## Time Complexity

All four operations: **O(1)**.

## Space Complexity

**O(n)** — `minStack` holds at most one entry per push.

---

## Code (Java)

```java
class MinStack {
    private final Deque<Integer> stack = new ArrayDeque<>();
    private final Deque<Integer> mins  = new ArrayDeque<>();

    public void push(int x) {
        stack.push(x);
        // Push to mins if it's empty OR x is the new minimum (<=, to handle duplicates).
        if (mins.isEmpty() || x <= mins.peek()) {
            mins.push(x);
        }
    }

    public void pop() {
        int x = stack.pop();
        // If we popped the current min, also pop the mins stack.
        if (x == mins.peek()) {
            mins.pop();
        }
    }

    public int top() {
        return stack.peek();
    }

    public int getMin() {
        return mins.peek();
    }
}
```

---

## Code (Python)

```python
class MinStack:
    def __init__(self):
        self.stack = []
        self.mins  = []

    def push(self, x: int) -> None:
        self.stack.append(x)
        if not self.mins or x <= self.mins[-1]:
            self.mins.append(x)

    def pop(self) -> None:
        x = self.stack.pop()
        if x == self.mins[-1]:
            self.mins.pop()

    def top(self) -> int:
        return self.stack[-1]

    def getMin(self) -> int:
        return self.mins[-1]
```

---

## Edge Cases

| Case | Why it works |
|---|---|
| Duplicate minimums | The `<=` push ensures every duplicate of the min gets onto `mins`, so each pop correctly removes one. |
| `pop` / `getMin` on empty stack | Undefined — guard if needed. |
| All same values | `mins` ends up the same size as the main stack. |
| Single element | Both stacks have it; works fine. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Can you do it with constant *extra* space (no second stack)?"**
   Yes — encode each value as its **difference** to the previous min in the main stack. Reconstruct min and value from this on the fly. Watch for integer overflow on the differences.

2. **"What about a Max Stack?"**
   Symmetric — track running max instead of min. Same code, flip comparisons.

3. **"What about a `getMin` queue?"**
   Different problem (a queue can't easily reuse the stack trick). Use a **monotonic deque**.

4. **"Why not just keep a single integer `min` variable?"**
   Because when you pop the current min, you'd lose track of the *new* minimum. The auxiliary stack remembers the history of minimums.

---

## Key Lessons from This Problem

1. **An auxiliary stack** can track derived information at each "level" of the main stack — a powerful pattern.
2. **`<=` (not `<`) for duplicates** — small detail, big consequence.
3. **Stack-based design** often hinges on this question: "what extra info would I need at each level?" Track that info in a parallel stack.
