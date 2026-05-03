# Problem: Implement a Queue Using Two Stacks

## Category
Stack / Queue / Design

## Difficulty
Easy

---

## Problem Statement

Design a **FIFO queue** using only two LIFO stacks. The queue must support `push` (enqueue), `pop` (dequeue), `peek` (front), and `empty`.

This is **LeetCode #232** — a classic puzzle.

---

## Clarifying Questions to Ask the Interviewer

1. **Push must be O(1)?** Yes (typical contract).
2. **Pop and peek can be amortized O(1)?** Yes — they may sometimes do extra work.
3. **Stacks are the only allowed primitive?** Yes — that's the whole point.

---

## Approach

### The Big Idea — Two Stacks With Different Roles

- **Stack `in`:** where new items are pushed (the "input" stack).
- **Stack `out`:** from which items are popped (the "output" stack).

`in` holds items in **reverse arrival order** (newest on top). `out` holds items in **arrival order** (oldest on top).

### Why this works

When you need to pop or peek the front of the queue:

1. If `out` is empty, **drain everything from `in` into `out`**. This reverses the order — what was newest in `in` becomes furthest from the top of `out`, and what was oldest is now on top of `out`.
2. Pop or peek the top of `out`.

This achieves FIFO behavior using two LIFO structures.

### Walking through some operations

```
push(1)
  in:  [1]            out: []
push(2)
  in:  [1, 2]         out: []      (2 is on top of in)
push(3)
  in:  [1, 2, 3]      out: []

pop()
  out is empty → drain in to out:
    pop 3 from in, push to out → in: [1, 2],    out: [3]
    pop 2 from in, push to out → in: [1],       out: [3, 2]
    pop 1 from in, push to out → in: [],        out: [3, 2, 1]
  Pop top of out → 1.   ✓ (the oldest in arrived)
  in: [],   out: [3, 2]

peek()
  out has 2 on top → 2.  ✓
  in: [],   out: [3, 2]

push(4)
  in: [4],   out: [3, 2]    (don't touch out!)

pop()
  out has 2 on top → 2.  ✓ (2 still came in before 3, 4)
  in: [4],   out: [3]
```

### Key invariant — drain only when `out` is empty

Don't keep moving items between stacks unnecessarily. Move them once when needed; keep them on `out` until used.

### Why amortized O(1)?

Each element makes at most **two trips**: pushed onto `in`, then later moved to `out`. Across `n` operations, total work is O(n) → O(1) per operation on average.

A *single* `pop` might be O(n) (when it triggers a big drain), but it only happens after many cheap O(1) `push`es. The cost averages out.

---

## Time Complexity

- `push`: **O(1)**.
- `pop`, `peek`: **amortized O(1)**, worst-case O(n) on a single call (the drain).

## Space Complexity

**O(n)** — total elements across both stacks.

---

## Code (Java)

```java
class MyQueue {
    private final Deque<Integer> in  = new ArrayDeque<>();
    private final Deque<Integer> out = new ArrayDeque<>();

    // Always O(1) — just push onto in.
    public void push(int x) {
        in.push(x);
    }

    public int pop() {
        shiftIfNeeded();
        return out.pop();
    }

    public int peek() {
        shiftIfNeeded();
        return out.peek();
    }

    public boolean empty() {
        return in.isEmpty() && out.isEmpty();
    }

    // Move everything from in to out, but ONLY when out is empty.
    // This is what reverses the order from LIFO to FIFO.
    private void shiftIfNeeded() {
        if (out.isEmpty()) {
            while (!in.isEmpty()) {
                out.push(in.pop());
            }
        }
    }
}
```

---

## Code (Python)

```python
class MyQueue:
    def __init__(self):
        self._in = []
        self._out = []

    def push(self, x: int) -> None:
        self._in.append(x)            # always O(1)

    def pop(self) -> int:
        self._shift()
        return self._out.pop()

    def peek(self) -> int:
        self._shift()
        return self._out[-1]

    def empty(self) -> bool:
        return not self._in and not self._out

    def _shift(self) -> None:
        # Drain in -> out only when out is empty.
        if not self._out:
            while self._in:
                self._out.append(self._in.pop())
```

---

## Edge Cases

| Case | What happens |
|---|---|
| `pop` / `peek` on empty queue | Undefined behavior — clarify with the interviewer (throw or return sentinel). |
| Mixed push/pop sequences | Still amortized O(1) — the analysis covers all interleavings. |
| One element queue | Push to `in`. Pop drains and returns it. Works correctly. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Why is this 'amortized' O(1) and not always O(1)?"**
   The drain step is O(n) on the operation that triggers it. But that operation can only happen after at least n cheap push operations. Per-operation, the cost averages to O(1).

2. **"Could you do it with one stack instead of two?"**
   Not without recursion or using the call stack as the second "stack." Two is the minimum.

3. **"Implement a stack using two queues."**
   Reverse problem. You can make either `push` or `pop` slow (O(n)). Standard solution: push is O(n) — always rotate the queue so the newest element is at the front.

4. **"Why don't we move items back to `in` after using them?"**
   That would multiply the work. The invariant "only move when `out` is empty" is what gives the amortized bound.

---

## Key Lessons from This Problem

1. **A pair of LIFO structures can simulate a FIFO** — by reversing the order on transfer.
2. **Amortized analysis** is essential for understanding "average cost" in algorithms with occasional expensive operations.
3. **Don't move data unnecessarily.** The "drain only when needed" invariant is what keeps this algorithm efficient.
