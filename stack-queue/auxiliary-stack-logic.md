# Problem: The Auxiliary Stack Pattern

## Category
Stack / Design

## Difficulty
Easy (concept) / Medium (applications)

---

## Problem Statement

Explain the **auxiliary stack pattern** — a technique that pairs a main stack with a second helper stack to maintain extra information.

This question tests whether you understand the **idea behind problems like Min Stack** rather than just having memorized one solution.

---

## Clarifying Questions to Ask the Interviewer

1. **Conceptual or coded?** Often both — explain the idea and give an example.
2. **What's the application?** Min Stack is the classic; we'll cover several.

---

## Approach (Plain-English Explanation)

### What's an "auxiliary stack"?

An auxiliary stack is a **second stack** that you maintain alongside your main one. Its job is to carry some derived information — the running min, max, sum, etc. — that you need to query in O(1).

The two stacks are kept **in sync**: pushes and pops on the main stack are mirrored (or selectively mirrored) on the aux stack.

### Why does this work so well?

Stacks have a beautiful property: each level of the stack corresponds to a **specific point in time** during the program's execution. By keeping a parallel "fact" at each level of an aux stack, you can answer "what was true when the main stack was in this state?" in O(1).

It's like layering two transparencies — the main stack and the aux stack — and looking at the top of both at once.

### Pattern recipe

For an "aggregate" you want to query in O(1):

1. **Initialize:** when a value is pushed to the main stack, also push the new aggregate to the aux stack.
2. **Query:** look at the top of the aux stack — that's the aggregate over the current main stack contents.
3. **Cleanup:** when popping the main stack, also pop the aux stack so they stay in sync.

The aggregate must be **monotonic in time** (e.g., min, max, sum, gcd) for this to work directly.

### Examples of the pattern

#### Min Stack — `getMin()` in O(1)

Aux stack tracks the running minimum. Push `min(x, current_min)` whenever you push.

#### Sum Stack — `getSum()` in O(1)

Aux stack stores running sum. Push `current_sum + x` whenever you push.

#### Max Stack — `getMax()` in O(1)

Symmetric to Min Stack.

#### GCD Stack — `getGCD()` in O(1)

Aux stack stores `gcd(current_gcd, x)` on each push.

#### Span / Next-Greater-Element — Monotonic stack

A slightly different flavor: the aux is the *only* stack, but it keeps elements in monotonic order (e.g., decreasing). When a new element arrives, pop everything smaller, then push.

#### Function call profiling

In a profiler, when a function is called, push its name onto the call stack and a timestamp onto an aux stack. On return, pop both and compute the duration. Same pattern.

### Walking through `MinStack` operations as a pattern example

```
Operation     Main stack       Aux stack (mins)
push(5)       [5]              [5]
push(2)       [5, 2]           [5, 2]    (2 ≤ 5, push)
push(7)       [5, 2, 7]        [5, 2]    (7 > 2, don't push)
push(2)       [5, 2, 7, 2]     [5, 2, 2] (2 ≤ 2, push)
getMin()                       2          (top of aux)
pop() → 2     [5, 2, 7]        [5, 2]    (popped value = aux top, also pop)
pop() → 7     [5, 2]           [5, 2]    (popped 7 ≠ 2, don't touch aux)
getMin()                       2          (still correct)
```

The aux stack always reflects the minimum over the current main stack. ✓

---

## Time Complexity

Every operation: **O(1)** (per main stack op).

## Space Complexity

**O(n)** — at most one aux entry per main stack entry.

---

## Code (Java) — generic auxiliary-stack scaffold

```java
import java.util.*;
import java.util.function.BinaryOperator;

class AggregatingStack<T> {
    private final Deque<T> data = new ArrayDeque<>();
    private final Deque<T> agg  = new ArrayDeque<>();
    private final BinaryOperator<T> combine;
    // combine takes (current_aggregate, new_value) -> new_aggregate
    // Examples:
    //   min:  (a, b) -> a.compareTo(b) <= 0 ? a : b
    //   max:  (a, b) -> a.compareTo(b) >= 0 ? a : b
    //   sum:  Integer::sum

    AggregatingStack(BinaryOperator<T> combine) {
        this.combine = combine;
    }

    public void push(T x) {
        data.push(x);
        T newAgg = agg.isEmpty() ? x : combine.apply(agg.peek(), x);
        agg.push(newAgg);
    }

    public T pop() {
        agg.pop();
        return data.pop();
    }

    public T top() { return data.peek(); }
    public T aggregate() { return agg.peek(); }
}
```

---

## Code (Python)

```python
from typing import Callable

class AggregatingStack:
    def __init__(self, combine: Callable):
        self.data = []
        self.agg = []
        self.combine = combine

    def push(self, x):
        self.data.append(x)
        if not self.agg:
            self.agg.append(x)
        else:
            self.agg.append(self.combine(self.agg[-1], x))

    def pop(self):
        self.agg.pop()
        return self.data.pop()

    def top(self):
        return self.data[-1]

    def aggregate(self):
        return self.agg[-1]

# Usage:
# s = AggregatingStack(min)   -> Min Stack
# s = AggregatingStack(max)   -> Max Stack
# s = AggregatingStack(lambda a, b: a + b)   -> Sum Stack
```

---

## Edge Cases

- **Aux must be popped in lockstep** with the main stack. Forgetting this is the most common bug.
- **The combine function must be associative** for the pattern to give the right global aggregate.
- **Empty stack queries** should be guarded — both stacks empty at the same time.

---

## Follow-up Questions an Interviewer Might Ask

1. **"When does the auxiliary stack pattern *not* work?"**
   When the aggregate depends on **future elements** (the stack only knows the past). For sliding-window problems, you usually need a different structure (deque, segment tree).

2. **"Can you eliminate the aux stack entirely?"**
   Sometimes — by encoding diff-based info in the main stack (see the "constant extra space" min-stack variant). Trades simplicity for trickier arithmetic.

3. **"What if the aggregate isn't easily incremental?"**
   Use a different data structure. For example, "median of values in stack" can't be done with a simple aux stack — it needs two heaps.

4. **"Show how monotonic stack relates to this pattern."**
   A monotonic stack is a single stack that's *trimmed on push* to maintain a sorted/monotonic invariant. It's a different pattern, but solves similar "next greater / next smaller" queries in O(n) total.

---

## Key Lessons from This Problem

1. **Pair structures** are powerful — when one structure can't do everything, two complementary ones often can.
2. **Each level of a stack is a snapshot** of the program's state at a point in time. An aux stack lets you query derived info about that snapshot in O(1).
3. **The pattern generalizes** to many "stack with O(1) aggregate" problems — recognize it once, use it everywhere.
