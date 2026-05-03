# Problem: Implement a Stack Using an Array

## Category
Stack / Design

## Difficulty
Easy

---

## Problem Statement

Implement a **Stack** data structure backed by a **dynamic array**, supporting these operations in **amortized O(1)**:

- `push(x)` — put `x` on top.
- `pop()` — remove and return the top.
- `peek()` (or `top()`) — return the top without removing.
- `size()` — number of elements.
- `isEmpty()` — true if empty.

---

## Clarifying Questions to Ask the Interviewer

1. **Fixed capacity, or auto-grows?** Auto-grow (dynamic array) — that's the interesting case.
2. **Generic types or just integers?** Generic for Java, anything for Python.
3. **Thread-safe?** No — single-threaded is fine here.

---

## Approach

### Why an array works for a stack

Stacks add and remove from one end — the "top." If we treat the **end** of an array as the top, both operations are O(1) because:
- Adding to the end is just placing a value at index `top` and incrementing `top`.
- Removing from the end is decrementing `top`.

### Handling growth

A stack might keep growing forever, but a fixed-size array can't. So when the array fills up:

1. **Allocate a new array** twice the size.
2. **Copy** the elements over.
3. Continue pushing.

This resize is O(n), but it happens **rarely**. Specifically, after each resize, you can do n more cheap O(1) pushes before the next resize. Averaged out, **each push is amortized O(1)**.

### Why "doubling" specifically?

If you grew by a constant amount (say, +10), each push of size 10n would trigger a resize → total cost over n pushes is `O(n²)`. Bad.

Doubling makes the costs of resize add up to `O(n)` total over n pushes, so each one is amortized O(1).

```
After 1, 2, 4, 8, 16, ... resizes, total copy cost = 1 + 2 + 4 + 8 + 16 + ... ≈ 2n
```

That's a geometric sum — amortized O(1).

### Operations in detail

```
push(x):
    if top == capacity: grow the array (double the capacity)
    data[top] = x
    top++

pop():
    if top == 0: error
    top--
    return data[top]   (optionally clear the slot for GC)

peek():
    if top == 0: error
    return data[top - 1]
```

`top` is the index of the **next free slot**. So `data[top - 1]` is the actual top-of-stack.

### Walking through `push(1), push(2), push(3), pop(), peek()`

```
Start: data = [_, _, _, _, _, _, _, _], top = 0     (capacity 8)

push(1): data[0] = 1, top = 1
push(2): data[1] = 2, top = 2
push(3): data[2] = 3, top = 3

pop():   top = 2, return data[2] = 3
peek():  return data[1] = 2
```

---

## Time Complexity

- `push`: **amortized O(1)** (occasional O(n) resize).
- `pop`, `peek`, `size`, `isEmpty`: **O(1)**.

## Space Complexity

**O(n)** — the array holds all stack elements.

---

## Code (Java)

```java
class ArrayStack<T> {
    private Object[] data = new Object[8];      // start small, grow as needed
    private int top = 0;                        // next free slot

    public void push(T x) {
        if (top == data.length) {
            // Double the array.
            data = Arrays.copyOf(data, data.length * 2);
        }
        data[top++] = x;
    }

    @SuppressWarnings("unchecked")
    public T pop() {
        if (top == 0) {
            throw new NoSuchElementException("Stack is empty");
        }
        T val = (T) data[--top];
        data[top] = null;       // help garbage collection
        return val;
    }

    @SuppressWarnings("unchecked")
    public T peek() {
        if (top == 0) {
            throw new NoSuchElementException("Stack is empty");
        }
        return (T) data[top - 1];
    }

    public int size() { return top; }
    public boolean isEmpty() { return top == 0; }
}
```

> Why `Object[]` instead of `T[]`? Java doesn't allow `new T[8]` because of generic type erasure. `Object[]` is the standard workaround, with unchecked casts.

---

## Code (Python)

```python
class ArrayStack:
    def __init__(self):
        self._data = []                # Python list is already a dynamic array

    def push(self, x):
        self._data.append(x)            # amortized O(1)

    def pop(self):
        if not self._data:
            raise IndexError("Stack is empty")
        return self._data.pop()          # O(1) — pops from the end

    def peek(self):
        if not self._data:
            raise IndexError("Stack is empty")
        return self._data[-1]

    def __len__(self):
        return len(self._data)

    def is_empty(self):
        return not self._data
```

> Python's `list` is a dynamic array internally. `append` and `pop` (from the end) are both amortized O(1).

---

## Edge Cases

| Case | What happens |
|---|---|
| `pop` / `peek` on empty stack | Throw — never silently return `null` (it can hide bugs). |
| Many pops after pushes | Memory stays high until you let the array shrink. Optional: shrink (halve) the array when load < 25%. |
| Generic types in Java | Use `Object[]` and cast on `pop` / `peek`. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Why is push amortized O(1)?"**
   Doubling means total resize cost across n pushes is O(n) — that's `1 + 2 + 4 + ... + n ≈ 2n`. Average is O(1) per push.

2. **"What's the worst-case time of a single push?"**
   O(n) — the one that triggers a resize. But we say "amortized O(1)" because that worst case is rare and budget-able.

3. **"Should the array shrink on pops?"**
   Sometimes. To avoid memory waste, shrink (halve) when load drops below ~25%. Don't shrink at 50% — that creates pathological cases (alternating push/pop near the boundary causes constant resize).

4. **"Difference between this and `java.util.Stack`?"**
   `java.util.Stack` extends `Vector` (synchronized, slow). Use `Deque` / `ArrayDeque` for modern Java code.

---

## Key Lessons from This Problem

1. **Doubling is the key** to amortized-O(1) array growth.
2. **`top` is the next free slot**, not the topmost element. Easy to get wrong.
3. **Throw on empty pop/peek**, don't return null. It's harder to debug silent failures.
