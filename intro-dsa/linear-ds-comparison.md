# Problem: Array vs Linked List vs Stack vs Queue

## Category
Intro to DSA / Theory

## Difficulty
Easy

---

## Problem Statement

Explain the differences between four basic data structures — **Array**, **Linked List**, **Stack**, and **Queue** — and where you'd use each in real life.

This is a foundational interview question. You aren't writing an algorithm; you're showing that you understand how data is **stored** and **accessed**.

---

## Clarifying Questions to Ask the Interviewer

1. **Static or dynamic array?** A static array has a fixed size at creation. A dynamic array (Java `ArrayList`, Python `list`) grows as you add more.
2. **Singly or doubly linked list?** Singly linked nodes only point to the next node; doubly linked also point to the previous one.

---

## Approach (Plain-English Explanations)

### 1. Array

An array is a chunk of **contiguous memory** that holds values of the same type.

```
Memory:
+----+----+----+----+----+
| 10 | 20 | 30 | 40 | 50 |
+----+----+----+----+----+
  0    1    2    3    4     ← indices
```

Because the values are next to each other in memory, the computer can compute **where** index `i` lives directly: `start_address + i × element_size`. That's why **array access is O(1)** — you can grab any element instantly by index.

**Strengths:**
- O(1) access by index.
- Excellent **cache performance** — the CPU loads nearby memory into fast cache, so iterating is fast.
- Low memory overhead — no extra pointers.

**Weaknesses:**
- Inserting or deleting in the **middle** is O(n) because everything after has to shift.
- Static arrays have a fixed size; dynamic arrays grow by reallocating (occasional slow operations, but **O(1) amortized** — see "amortized" below).

**Real-life analogy:** Seats in a movie theater row. You can jump to seat 47 in one step (you know the layout), but inserting a new seat in the middle would mean shoving everyone in seats 47+ down by one.

### 2. Linked List

A linked list stores values in **nodes** scattered across memory. Each node holds its value and a **pointer** to the next node.

```
+----+    +----+    +----+    +------+
| 10 | -> | 20 | -> | 30 | -> | NULL |
+----+    +----+    +----+    +------+
 head
```

There's no built-in way to jump to "the 5th node" except by walking from the head, one node at a time.

**Strengths:**
- Inserting / deleting at the **front** is O(1).
- Inserting / deleting at the **end** is O(1) **if** you keep a tail pointer.
- The list grows and shrinks naturally; no resizing.

**Weaknesses:**
- O(n) to access a specific index.
- Each node uses extra memory (the pointer).
- Bad cache performance — nodes are scattered, so the CPU keeps loading new memory regions.

**Real-life analogy:** A treasure hunt where each clue tells you where the next clue is. You can't skip ahead — you have to follow the chain.

### 3. Stack — Last In, First Out (LIFO)

A stack is an **abstract data type** (a behavior, not a specific layout). It supports two main operations:

- `push(x)` — put a value on top.
- `pop()` — remove and return the **most recently pushed** value.

```
push(1) push(2) push(3):

  | 3 |   ← top
  | 2 |
  | 1 |
  +---+

pop() returns 3.
```

**Real-life analogy:** A stack of plates. You can only add or take from the top.

**Where you'd use it:**
- The **call stack** that runs your program — every function call pushes a frame; returning pops it.
- **Undo** in a text editor — every action is pushed; "undo" pops the most recent.
- **Parsing** parentheses, brackets, expressions.
- **DFS** (depth-first search) on a graph or tree.

A stack is usually backed by an array or a linked list — implementation detail.

### 4. Queue — First In, First Out (FIFO)

A queue is also abstract. It supports:

- `enqueue(x)` — add to the **back**.
- `dequeue()` — remove from the **front** (the value that was added first).

```
front                           back
  |                              |
  v                              v
+---+---+---+---+---+
| 1 | 2 | 3 | 4 | 5 |
+---+---+---+---+---+

dequeue() returns 1.
enqueue(6) puts 6 at the back.
```

**Real-life analogy:** A queue at a coffee shop. First person in line is first to be served.

**Where you'd use it:**
- **BFS** (breadth-first search) — explore a graph layer by layer.
- **Job scheduling** — handle tasks in arrival order.
- **Producer-consumer** systems — one part produces work, another consumes it.

Queues are usually backed by a linked list or a "circular array" (array with wrapping indices).

---

## Comparison Table

| Property | Array | Linked List | Stack | Queue |
|---|---|---|---|---|
| Access by index | O(1) | O(n) | only top | only front |
| Insert/delete at end | O(1) amortized | O(1) (with tail) | push: O(1) | enqueue: O(1) |
| Insert/delete at front | O(n) | O(1) | n/a | dequeue: O(1) |
| Cache friendliness | Excellent | Poor | Depends on backing | Depends on backing |
| Memory per item | Just the value | Value + pointer(s) | Depends | Depends |
| Order of access | Random | Sequential | LIFO | FIFO |

### What does "amortized O(1)" mean?

A dynamic array doubles its capacity when it fills up. The doubling itself is O(n) work, but it happens rarely. **Averaged over many insertions**, each insertion costs O(1). That's "amortized O(1)" — a single operation might be slow, but the average over a sequence is fast.

---

## Time Complexity / Space Complexity

N/A — this is a theory question. The complexities of operations are in the table above.

---

## Code (Java) — minimal API examples

```java
// Array (dynamic)
List<Integer> arr = new ArrayList<>();
arr.add(1);            // append — amortized O(1)
int x = arr.get(0);    // O(1) access by index

// Linked list node — you'd usually define your own
class Node {
    int val;
    Node next;
    Node(int v) { this.val = v; }
}

// Stack — prefer ArrayDeque over the legacy Stack class
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1);          // top is 1
stack.push(2);          // top is 2
int top = stack.pop();  // removes 2, returns 2

// Queue — also ArrayDeque
Deque<Integer> queue = new ArrayDeque<>();
queue.offer(1);         // back: 1
queue.offer(2);         // back: 2, front: 1
int front = queue.poll(); // removes 1, returns 1
```

> **Why `ArrayDeque` and not `Stack`?** Java's `Stack` class is from the early days, extends `Vector`, and is synchronized — slower and not recommended in modern code. `ArrayDeque` is the recommended choice for both stacks and queues.

---

## Code (Python)

```python
# Array — Python lists are dynamic arrays under the hood
arr = []
arr.append(1)           # amortized O(1)
x = arr[0]              # O(1)

# Linked list node
class Node:
    def __init__(self, val):
        self.val = val
        self.next = None

# Stack — just use a list
stack = []
stack.append(1)         # push
stack.append(2)
top = stack.pop()       # pops 2

# Queue — use deque (double-ended queue), NOT a list
# Why? list.pop(0) is O(n) because it shifts everything left.
# deque.popleft() is O(1).
from collections import deque
q = deque()
q.append(1)
q.append(2)
front = q.popleft()     # returns 1 in O(1)
```

---

## Edge Cases

- **`pop()` on empty stack** → throws an exception in most languages. Always check `isEmpty()` first or wrap in a try/except.
- **`dequeue()` on empty queue** → same idea.
- **Walking past the end of a linked list** → you hit `null` (Java) or `None` (Python). Always check before dereferencing.

---

## Follow-up Questions an Interviewer Might Ask

1. **"Why does `ArrayList.add` cost O(1) on average if it has to copy when full?"**
   When the array doubles, copying takes O(n). But it happens after `n` cheap inserts. Total cost across n inserts is `O(n)`, so each insert is amortized O(1).

2. **"Can a stack be implemented with a queue, or vice versa?"** Yes. Either implementation makes one of the operations slow (O(n)).

3. **"Why is a linked list better than an array for inserting in the middle?"**
   Inserting into a linked list at a known node is O(1) (just rewire pointers). Inserting into an array at index `i` is O(n) because everything from `i` onward must shift right.

4. **"Why can't you do binary search on a linked list efficiently?"**
   Binary search needs O(1) access to the middle. On a linked list, finding the middle takes O(n) — by then you've already done linear work.

---

## Key Lessons from This Problem

1. **Memory layout matters.** Arrays win on speed because of contiguous memory and CPU caching. Linked lists win on flexibility.
2. **"Stack" and "queue" describe behavior, not a specific layout.** They can be built on top of arrays or linked lists.
3. **Choose your data structure based on which operation you do most.** Random access? Array. Insert/remove at ends? Linked list, stack, or queue.
